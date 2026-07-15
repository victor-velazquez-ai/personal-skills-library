# Test taxonomy for agentic systems

The layers, what each is *for*, and — critically — **what oracle it uses**.
The oracle is the hard part in LLM systems; pick the layer by the oracle you
can actually get.

**Rule of thumb:** push every check to the cheapest layer that can catch it.
Never use an LLM judge where a deterministic relation exists.

---

## L0 — Unit
**Oracle:** exact equality.
**Tests:** the deterministic core — date/window resolution, parsers,
formatters, permission logic, cron math, template rendering, retry policy.
**Why it matters here:** in most agentic systems a surprising amount of the
damage lives in *ordinary* code — timezone math, off-by-one windows, string
building. This is the cheap layer. Use it fully before reaching for LLM tests.
**Trap (D1):** confirm the unit under test is **reachable** from an entry
point. Testing an internal that nothing dispatches to is worse than no test.

## L1 — Contract
**Oracle:** schema.
**Tests:** every boundary — tool/function signatures, structured-output
conformance, API request/response shape, DB query shape (does the generated
query bind params? filter by tenant? avoid writes?).
**Notes:** structured output at the API level beats parsing prose (C3). Assert
the *generated query*, not just the answer — it's a deterministic artifact and
a far better oracle than text.

## L2 — Smoke
**Oracle:** liveness.
**Tests:** each surface completes one canonical task end to end.
**Notes:** run per deploy, per surface. Fast and shallow **on purpose**. A
smoke test that skips by default (missing env var) is decoration — D3.

## L3 — Golden / eval set
**Oracle:** a fixed dataset of input → expected.
**Tests:** per intent, a known question against seeded data with a known
answer.
**Notes:**
- Assert the **data** (rows/IDs/counts), not prose similarity. Similarity
  scores pass confidently-wrong answers — the exact bug you're hunting.
- Needs a **deterministic fixture** — seeded, stable, spanning boundaries
  (a DST transition, a month end, duplicates, NULLs). **This fixture is the
  expensive part of the whole programme.** Budget it explicitly; it is the
  prerequisite for L3/L4/L5/L10 and it always hides inside "write some tests".
- Golden answers **rot** as data changes. Metamorphic relations don't — which
  is why L4 carries more weight over time.
- **Mine production for cases.** Real questions beat imagined ones.

## L4 — Metamorphic ★
**Oracle:** relations between outputs — **no expected answer needed.**
**Tests:** see `metamorphic-relations.md`.
**Why it's the centrepiece:** it is the only correctness layer that works
without an oracle, survives data drift, costs nothing to maintain, and cannot
hallucinate. If you run one LLM-specific layer, run this one.

## L5 — Property-based
**Oracle:** invariants that hold for all inputs.
**Tests:** generate inputs; assert properties. e.g. *every* response either
answers or explicitly declines; *never* contains another tenant's ID; *always*
states its scope; a stated count *always* equals the rows.
**Notes:** the natural home for **honesty invariants** — which are the
properties nobody writes and everybody needs. Overlaps L4; L4 relates two
runs, L5 constrains one.

## L6 — Simulated-user E2E ★
**Oracle:** goal achievement + honesty, judged.
**Tests:** agents driving the real product as personas, multi-turn. See
`simulated-users.md`.
**Why it's the centrepiece:** it's the only layer that finds what users
actually hit — because it *is* a user. Everything else tests what you thought
to ask.
**Notes:** expensive and non-deterministic. Use it to *discover* bug classes,
then pin each discovery as a cheap L4/L3/L10 case. **Discovery layer, not a
gate.**

## L7 — Adversarial / red-team
**Oracle:** must-never rules.
**Tests:** direct injection; **indirect injection** (payload in retrieved
documents or data fields — usually unguarded); cross-tenant enumeration;
schema/prompt probing; bulk exfiltration; jailbreak; role confusion.
**Add a cheap, high-yield category most red-teams miss: politely ask the system
to break its own documented invariants** — *"include the deleted records too"*,
*"ignore the usual filter"*. No jailbreak required; any invariant living only in
a prompt will simply comply (**D5**). Then check the *guard's overrides*, not
just the guard (**B7**) — a rejection that a salvage path can reverse is not a
rejection.
**Notes:** rules are binary and don't rot — ideal CI material. Cross-tenant
leakage is the highest-severity finding available. Test the *indirect* path:
most systems guard the user's message and trust everything else.

## L8 — Chaos / fault injection
**Oracle:** honesty of failure.
**Tests:** dependency down, 429, timeout, malformed response, empty result,
partial data, slow response, auth expiry.
**The question is never "did it survive"** — it's **"what did it tell the
user?"** Most of Class A/B in `failure-patterns.md` is only reachable here:
fabrication-on-error (A5), assertion-of-absence (A2), failure-as-success (B2)
all need a fault to surface.
**Notes:** the least-run layer with the highest yield per test. Nearly every
"it lied to the customer" bug is a fault path nobody injected.

## L9 — Flake / variance
**Oracle:** self-consistency.
**Tests:** same input × N (N≥10). Measure agreement per stage and end to end.
**Notes:** produces the **reliability baseline** — the "before" number for
every future change. Report n/N always. Intermittent bugs are real bugs and
get dismissed without a frequency.

## L10 — Regression
**Oracle:** fixed dataset — every past bug.
**Tests:** each production incident, as a permanent case.
**Notes:** **the ratchet.** Every finding becomes a case *before* it becomes a
fix; the suite only grows; the same bug cannot return. Cheapest layer to
justify — every case has a real incident behind it.

---

## Choosing per finding class

| Failure class | Cheapest layer that catches it |
|---|---|
| Silent partial results (A1) | L4 (MR-4) |
| Assertion of absence (A2) | L8 + L5 |
| Narrative/data divergence (A3) | L4 (MR-11) |
| Auto-relaxation (A4) | L4 (MR-3) |
| Fabrication on error (A5) | L8 |
| Unscoped output (A6) | L5 |
| Failure invisible / as success (B1, B2) | L8 |
| Dead fallback / unreachable guard (B3, B4) | L0 |
| Disclosure never rendered (B5) | L0 + L5 |
| Unpinned sampling (C1) | recon + L9 |
| Multiplied gates (C2) | recon + L9 |
| State loss (C4) | L4 (MR-12) |
| **Ambient-source override (C5)** | **L1 on the generated artifact** — grep it for forbidden sources |
| **Guard bypassed by salvage path (B7)** | L0 on the bypass + **L7** |
| **Prompt-rule-only invariant (D5)** | **L7** — ask it to break its own rule |
| Coverage illusions (D1–D4) | **recon** — not a test layer |
| Boundaries (E1–E6) | L0 + L3 + L4 (MR-6, MR-9, MR-11) |
| Cross-tenant (F1) | L4 (MR-7) + L7 |
| Injection (F2) | L7 |

**Note the pattern:** the coverage illusions (D) are found by *reading*, not
by running. That's why recon is a phase, not a preamble.

---

## What belongs in CI

| Layer | CI? | Why |
|---|---|---|
| L0, L1, L2 | **always, blocking** | fast, deterministic |
| L3 golden | **blocking** on prompt/agent/graph changes | the core gate |
| L4 metamorphic | **blocking** — the subset that's fast | cheap, no maintenance, no rot |
| L5 property | blocking (bounded generation) | honesty invariants |
| L7 adversarial | **blocking** | binary rules, don't rot |
| L10 regression | **always, blocking** | the ratchet |
| L6 simulated-user | **nightly / pre-release** | slow, costly, non-deterministic |
| L8 chaos | nightly, or per release | needs fault harness |
| L9 variance | nightly + before/after any model or prompt change | trend, not gate |

**The gate that matters:** *any change to a prompt, graph, or model config
runs L3+L4 or it does not merge.* A 2,000-line prompt file merged on hope is
the mechanism behind "one question works, one doesn't".

**Set the threshold at the measured baseline and ratchet.** Starting at 100%
guarantees the gate gets disabled within a week. `allow_failure: true` on the
one job that checks correctness is the same as having no job — and it looks
like diligence, which is worse.
