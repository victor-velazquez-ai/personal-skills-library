# Metamorphic relations for agentic systems

**The no-oracle problem.** You cannot write `assert output == expected` when
the wording changes every run. So teams either skip correctness testing or
fall back on fuzzy similarity scores that pass confidently-wrong answers.

**Metamorphic testing is the escape.** Instead of asserting *what* the output
is, assert a **relation between the outputs of two related inputs**. You never
need to know the right answer — only that two answers must relate a certain
way. Violations are real bugs, deterministically detected, with no judge and
no hallucination risk.

> If `ask("who was on site today")` and `ask("list the people present today")`
> return different row sets, one of them is wrong. **You don't need to know
> which** to know you have a bug.

This is the highest-yield technique available for LLM systems, and the most
under-used.

---

## How to run one

1. **Source input** `x` → run → output `f(x)`.
2. **Transform** it: `x' = T(x)` where the relation `R` must hold.
3. **Follow-up run** → `f(x')`.
4. **Assert** `R(f(x), f(x'))`.
5. **On violation:** report *both* runs. The pair is the repro.

**Compare data, not prose.** Normalise to the underlying result — row sets,
IDs, counts, the executed query — and compare *those*. Prose will differ every
run and tells you nothing. If the system won't expose its result set, that is
itself a finding (see "unscoped output" in `failure-patterns.md`) — fall back
to extracting entities/numbers from the response, and note the weaker oracle.

**Control for real non-determinism.** Establish the idempotence baseline (MR-2)
*first*. If `f(x)` already varies run-to-run, every other relation is measured
against noise. Run each side N times and compare distributions, not singletons.

**Beware the dominant defect.** If one bug fails a large share of runs outright
(everything returns empty, everything errors), **it confounds every other
relation** — the code path you meant to test never executes. You will report
"MR-9 violated" when MR-9 never ran.

> **Field note.** On this skill's first use, a bug that zeroed ~25% of queries
> made two relations (NULL-handling, scope monotonicity) unmeasurable — every run
> of those cells hit the dominant bug first. The correct move was to report them
> **NOT MEASURED**, with the reason, and re-queue them behind the fix. Do not
> convert a confounded cell into a finding. "I could not measure this, and here's
> why" is a real result; a fabricated violation is worse than no test.

**Design the fixture to discriminate.** Every relation needs data that can
*distinguish* pass from fail. A monotonicity test where the filtered set happens
to equal the base set proves nothing, however green it looks.

> **Field note.** Same run: MR-3 (adding a filter must not increase rows) was
> unmeasurable because every worker with data on the test day belonged to the one
> company being filtered on — base and filtered were both 4. The fixture, not the
> system, was the problem. **For each relation, ask: what row must exist for this
> to be able to fail?** Then seed it.

---

## The catalogue

Each relation: what it asserts · the transform · the bug class it catches.

### MR-1 — Paraphrase invariance
**Assert:** semantically identical requests return identical data.
**Transform:** reword — synonyms, terse ↔ verbose, question ↔ imperative,
reordered clauses, domain jargon ↔ plain language.
**Catches:** prompt brittleness — "I used one wrong word and it changed
everything". The most common complaint about agentic products, and almost
never tested.
**Note:** the *prose* may differ. The *data* may not.

### MR-2 — Idempotence / self-consistency
**Assert:** `f(x)` run N times returns the same data.
**Transform:** none — repeat.
**Catches:** the flake budget itself. Unpinned sampling, non-deterministic
routing, race conditions in state.
**This is the baseline for every other relation.** Report as n/N. A system that
fails MR-2 cannot pass anything else reliably, and fixing it is usually a
config change, not a redesign.

### MR-3 — Scope monotonicity
**Assert:** adding a constraint never *increases* the result set.
`|f(x AND filter)| ≤ |f(x)|`
**Transform:** add a filter (a company, a date bound, a status).
**Catches:** **auto-relaxation** — retry loops that widen constraints until
something returns, silently answering a different question than the one asked.
A brutal, cheap, high-yield relation.

### MR-4 — Decomposition consistency
**Assert:** `f(A ∪ B) == f(A) ∪ f(B)` for independent entities.
**Transform:** split a multi-entity request into single-entity requests.
**Catches:** **silently dropped entities** — ask for 3 names, get 2, no
mention. Also single-value assumptions in filter construction, and blended
embeddings that can't represent two entities at once.

### MR-5 — Surface parity
**Assert:** the same question through any entry point returns the same data.
**Transform:** chat → API → scheduled → batch.
**Catches:** divergent code paths. Background/scheduled paths often lack
context (auth, timezone, locale) the interactive path has — and **nobody
watches them**, so the divergence persists for months.

### MR-6 — Temporal stability
**Assert:** a relative window resolved at different moments *within* that
window resolves identically.
**Transform:** run "today" at 09:00 and at 23:00 local; run from different
server timezones; run either side of UTC midnight.
**Catches:** the entire timezone-drift family — the single most common
correctness bug in scheduled reporting. If "today" depends on *when* or
*where* you ask, everything downstream is wrong.
**Also test:** DST transitions, month/quarter/year ends, leap day.

### MR-7 — Permission monotonicity
**Assert:** a narrower-scoped principal never receives more than a broader one.
`f(user_narrow, x) ⊆ f(user_broad, x)`
**Transform:** same question, decreasing permission scope.
**Catches:** **cross-tenant leakage** — the highest-severity finding
available. Pair with L7 adversarial: ask for another tenant's data *by name*.

### MR-8 — Order invariance
**Assert:** reordering entities doesn't change the returned set.
**Transform:** "A and B" → "B and A".
**Catches:** last-write-wins filter construction; first-match short-circuits.

### MR-9 — Complement completeness
**Assert:** `count(A) + count(NOT A) == count(all)`.
**Transform:** partition on a boolean/categorical property.
**Catches:** rows silently vanishing — NULL-propagating joins/predicates
(`a || ' ' || b` where `b` is NULL), inner joins that should be outer,
undisclosed truncation.

### MR-10 — Robustness / noise invariance
**Assert:** trivial perturbation doesn't zero the result.
**Transform:** typo, case change, extra whitespace, punctuation, honorific,
diacritics, nickname.
**Catches:** exact-match brittleness where fuzzy matching is expected. **A
typo must never silently return "no results"** — it must clarify or near-match.
Silent zero is the failure; an honest "did you mean X?" is a pass.

### MR-11 — Aggregation consistency
**Assert:** the summary agrees with the detail. `sum(f(part_i)) == f(whole)`;
a stated count equals the actual rows.
**Transform:** compare a rollup against its components.
**Catches:** **narrative/data divergence** — the prose claiming one thing while
the rows say another; stats computed on a sample and reported as exact;
undisclosed truncation ("showing 20" of 347, renumbered 1–20).

### MR-12 — Conversational context stability
**Assert:** a follow-up preserves established scope. `ask(A); ask("and last
week?")` == `ask(A + last week)`.
**Transform:** single-turn ↔ equivalent multi-turn.
**Catches:** state loss between turns — dropped filters, forgotten entities,
clobbered session state. The "it forgot what we were talking about" complaint.

### MR-13 — Null-effect invariance
**Assert:** irrelevant additions don't change the data.
**Transform:** append "please", "thanks", "asap", an emoji, a signature.
**Catches:** prompt-injection sensitivity and over-eager intent
reclassification. Cheap to run, occasionally alarming.

### MR-14 — Format independence
**Assert:** the same question in different output formats carries the same
data.
**Transform:** "show me" → "export to CSV" → "chart it".
**Catches:** format-specific paths silently truncating, dropping, or
re-aggregating.

---

## Choosing relations

Not all apply to every system. Derive from the recon map:

| If the system… | Run |
|---|---|
| answers in natural language | MR-1, MR-2, MR-13 |
| filters or queries data | MR-3, MR-4, MR-8, MR-9, MR-11 |
| handles named entities | MR-4, MR-8, MR-10 |
| has relative time ("today", "last week") | **MR-6** |
| has more than one entry surface | **MR-5** |
| is multi-tenant or permissioned | **MR-7** |
| is conversational | MR-12 |
| exports/renders multiple formats | MR-14 |
| retries or self-corrects | **MR-3** (retry loops are relaxation engines) |

**Start with MR-2 (baseline), then MR-3, MR-4, MR-6.** In practice those three
find the most damage per unit of effort in data-backed agentic products.

---

## Implementation notes

- **Generate pairs programmatically**, not by hand. An LLM is excellent at
  producing 20 paraphrases per intent; the *comparison* must stay
  deterministic code.
- **Store the pair, always** — both inputs, both outputs, both run IDs. The
  pair is the bug report.
- **Tolerances:** exact set equality for IDs/counts. Never similarity scores
  for data — that's how confidently-wrong answers pass.
- **Cheap and permanent.** Relations don't rot like golden answers do: they
  hold as the data changes, so they belong in CI from day one.
- **Violations point at a mechanism.** MR-3 failing means something widens
  filters. MR-5 failing means the surfaces diverge. Follow the relation to the
  code — it's a short walk, and it's what turns a finding into a ticket.
