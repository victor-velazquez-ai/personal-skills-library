---
name: agentic-system-test
description: >
  Exhaustively test an agentic / LLM system the way real users break it —
  recon the app first, derive the full scenario space (personas × intents ×
  phrasings × data states × edge cases), then run every test layer that
  applies: unit, contract, smoke, golden/eval, metamorphic, property-based,
  simulated-user E2E, adversarial/red-team, chaos/fault-injection, flake
  variance, and regression. Fans out sub-agents to drive the product as
  users and to attack the prompts. Produces a findings report + a ticketed
  backlog with success criteria. Use when the user asks to "test the agent",
  "test the whole agentic system", "find the bugs before users do", "build
  an eval/test harness", "QA this AI product", or "why does it work
  sometimes and not others".
---

# Agentic system test — recon → scenario space → exhaust → report → backlog

## The problem this exists for

Agentic systems ship bugs to production that testing never caught, because
they are **not deterministic** and the interesting failures are not crashes.
A crash announces itself. An agentic system instead returns a **confident,
well-formatted, wrong answer** — and nothing in the stack notices. Traditional
tests don't catch it because:

- There is **no exact oracle**. You can't assert `output == expected` when the
  wording changes every run, so teams test the plumbing and never the answer.
- The failure surface is **combinatorial** — persona × phrasing × data state ×
  history × permissions. Users explore it exhaustively; test suites sample it.
- The system **degrades silently**: partial results look complete, empty
  results look like real answers, an error string can become narrative prose.
- Coverage metrics lie. Tests can be green on code that **cannot execute**.

So the bugs are found by users, in production, one trust-destroying incident
at a time.

**The counter-move, and the core thesis of this skill:**

> You cannot assert exact outputs. So assert three other things instead:
> **(1) invariants** that must hold for every input, **(2) relations between
> outputs** (metamorphic testing — the answer to "no oracle"), and
> **(3) honesty** — that the system's claims about its own scope, coverage,
> and failures are true.
>
> And hunt the right bug class: **silent wrongness beats crashes.**

## When to use

- Testing/QA of an agent, chatbot, RAG pipeline, LLM workflow, or any
  LLM-in-the-loop product, especially one already customer-facing.
- "It works sometimes and not others." "Users keep finding bugs we didn't."
- Building an eval/regression gate for a system that has none.
- Not for a single deterministic function — that's a unit test.

## Non-negotiables

1. **Recon before testing.** You cannot test an app you don't understand; you
   will test the wrong things and report noise. Phase 0 is mandatory.
2. **Drive the real system.** Mocked LLM output tests your mock. At least one
   layer must exercise the deployed path end to end.
3. **Every finding gets a repro.** A finding a developer can't reproduce is a
   rumour. Exact input, run ID, observed vs expected, frequency (n/N).
4. **Verify before reporting.** LLM-driven testing generates false positives.
   Phase 4 exists to kill them. An unverified finding costs more than none.
5. **Report ≠ cure.** The report is the diagnosis; the backlog is the action
   plan. Keep them separate documents.

---

## Phase 0 — Recon: understand the app before you test it

**Do not skip. Do not shorten.** Everything downstream is derived from this.
Full checklist: [references/recon-checklist.md](references/recon-checklist.md).

Answer these, grounded in real code/config (`file:line`), not assumptions:

| Question | Why it decides the test plan |
|---|---|
| **What does the product promise?** Who are the users, what do they ask for, what do they do with the answer? | Defines the personas and the scenario space. A wrong number in a compliance report ≠ a wrong number in a toy demo. |
| **What are the entry surfaces?** chat, scheduled/cron, API, batch, webhook | Each is a separate code path and **they diverge**. Same question via two surfaces returning different data is a top-tier bug class. |
| **What is the actual graph?** nodes, edges, routing decisions, tools | The stated architecture and the executing one differ. Dead nodes are common. |
| **Where is the non-determinism?** every LLM call, its temperature, its job (routing vs generation), structured output or free-text parse | Sequential LLM gates **multiply**: 5 calls at 95% ≈ 77% end-to-end. This is usually the flake budget. |
| **Where does truth live?** the data model, and what the system *claims* it can answer | Questions unanswerable from the schema are a distinct bug class from wrong answers. |
| **What are the tenancy/permission boundaries?** | Cross-tenant leakage is the highest-severity finding available. |
| **What already exists?** tests, evals, CI, tracing, run history | Tells you what's untested — and what's *falsely* tested. |
| **How do I run it?** the actual command/endpoint, auth, a safe environment | If you can't drive it, you can't test it. Resolve this in Phase 0, never at execution time. |

**Recon deliverable:** a short system map — surfaces, graph diagram, LLM-call
inventory with temperatures, data model, personas, permissions, how-to-run.
The test plan is derived from this map and traceable to it.

> **Two traps to check for in every recon.** They are common and they
> invalidate the test plan:
> - **Tests that certify unreachable code** — a suite calling an internal
>   directly, bypassing the dispatch that never routes to it. Green, and
>   worthless. Confirm the code under test is *reachable from an entry point*.
> - **Fixes that are config-inert** — the code path shipped, the config that
>   activates it was never set. Verify against the *deployed* config, not the
>   example file.

### Safety gate (settle before Phase 3)

Get explicit answers, in writing, before anything executes:

- **Which environment?** Never production without explicit, specific consent.
- **Can it write?** Testing sends emails, charges credits, mutates rows,
  calls paid APIs, pages on-call. Establish blast radius *first*.
- **Is the data real?** Real PII in test transcripts becomes a leak in your
  report. Redact at capture, not at write-up.
- **What's the budget?** Exhaustive × LLM = real money. Agree a ceiling.

---

## Phase 1 — Derive the scenario space

From the recon map, enumerate the space *before* sampling it. Write it down —
the gap between "what exists" and "what we'll run" is a coverage decision, and
it must be explicit rather than accidental.

**The axes** (see [references/scenario-matrix.md](references/scenario-matrix.md)):

- **Personas** — role × permission scope × expertise. Include the *hostile*
  and the *confused* user, not just the happy one.
- **Intents** — every task the product promises. Mine real usage/logs if
  available; they beat imagination.
- **Phrasings** — per intent: canonical, terse, verbose, misspelled, jargon,
  synonym, wrong-terminology, multilingual, ambiguous, compound.
- **Data states** — populated, empty, single-row, huge, NULL-bearing,
  duplicate-keyed, boundary (midnight, month/DST/year edges), stale.
- **Entity conditions** — exists, absent, misspelled, ambiguous/duplicate,
  multi-entity, out-of-scope, mixed valid+invalid.
- **Conversation shape** — single-turn, follow-up needing context, correction
  ("no, I meant…"), topic switch, long history, interleaved/concurrent.
- **Surface** — every entry point × every intent (this is where parity bugs
  hide).

**Prioritise by damage, not by ease.** Rank cells by (blast radius × silence).
A wrong answer nobody can detect outranks a crash everybody sees.

---

## Phase 2 — The test plan

Map each scenario cell to the cheapest layer that can catch its failure.
Full taxonomy: [references/test-taxonomy.md](references/test-taxonomy.md).

| Layer | Catches | Oracle |
|---|---|---|
| **L0 Unit** | deterministic logic: date math, parsers, formatters, permissions | exact |
| **L1 Contract** | tool/schema boundaries, structured-output conformance, API shape | schema |
| **L2 Smoke** | is it alive; can each surface complete one canonical task | liveness |
| **L3 Golden / eval** | known question → known answer, per intent | fixed dataset |
| **L4 Metamorphic** | **no-oracle correctness** — relations between outputs | relations |
| **L5 Property** | invariants over generated inputs | invariants |
| **L6 Simulated-user E2E** | real multi-turn usage by a persona, on the real system | judge + invariants |
| **L7 Adversarial** | injection, cross-tenant, jailbreak, exfiltration | must-never rules |
| **L8 Chaos** | dependency failure, timeout, 429, empty, malformed | honesty of failure |
| **L9 Flake/variance** | non-determinism — same input × N | self-consistency |
| **L10 Regression** | every past production bug | fixed dataset |

**L4 and L6 are the reason this skill exists.** L0–L3 are ordinary
engineering. L4 solves the no-oracle problem; L6 is the only layer that finds
what users actually hit.

---

## Phase 3 — Execute: exhaust the space

**Fan out.** This is embarrassingly parallel — one agent per persona, per
intent cluster, per attack class, per chaos scenario. Use as many as the work
needs; that is the whole point of the skill. Give each agent a **narrow**
scope, the recon map, the run instructions, and a **structured output schema**
so results are aggregatable rather than prose.

**The two layers that find the real bugs:**

### L4 — Metamorphic testing (the core technique)

You have no oracle, so stop looking for one. Assert **relations** that must
hold *between* outputs of related inputs. Each is a real bug when violated,
and none requires knowing the right answer.

Full catalogue + how to implement:
[references/metamorphic-relations.md](references/metamorphic-relations.md).

The high-yield relations for agentic systems:

- **Paraphrase invariance** — rewording the same request must not change the
  data returned.
- **Idempotence** — same input, N runs, same answer. Violations *are* the
  flake budget, measured.
- **Scope monotonicity** — adding a constraint must never *increase* results.
  (Catches retry loops that "relax filters until something comes back".)
- **Decomposition consistency** — ask(A ∪ B) == ask(A) ∪ ask(B). Catches
  silently dropped entities in multi-entity requests.
- **Surface parity** — the same question via chat / API / scheduled must
  return the same data.
- **Temporal stability** — a relative window ("today") resolved at different
  clock times *within* that window must resolve identically. Catches the
  entire timezone-drift bug family.
- **Permission monotonicity** — narrower permissions never return more.
- **Order invariance** — reordering entities in a request must not change the
  set returned.
- **Complement completeness** — count(A) + count(not A) == count(all).
- **Robustness** — typos, case, whitespace must not silently zero the result.

### L6 — Simulated-user E2E (test as a user)

Deploy sub-agents as **personas** that drive the real product multi-turn,
pursuing a goal, and judge the transcript. Not scripted inputs — an agent that
*reacts* and pushes back, the way a real user does.

How to build it: [references/simulated-users.md](references/simulated-users.md).

Give each persona a **goal**, a **character** (terse / verbose / confused /
adversarial / non-native speaker / expert), and a **stopping condition**. Judge
the transcript against: did they get the goal, was the answer true, did the
system admit what it didn't know, would they trust it.

**Grade the transcript, not the answer.** Systems that answer confidently and
wrongly score well on answer-similarity and are the most dangerous thing you
can ship.

### What to look for — the failure catalogue

The bug classes specific to agentic systems, and how to detect each:
[references/failure-patterns.md](references/failure-patterns.md). The
headline ones:

- **Silent partial results** — asked for N, got M<N, no mention.
- **Assertion of absence** — "there are none" when the query merely failed.
  The system asserts a fact about the world it has no evidence for.
- **Narrative/data divergence** — the prose says one thing, the rows another.
- **Unscoped output** — a result that can't state what it covers.
- **Auto-relaxation** — retries that loosen constraints until *something*
  returns, silently answering a different question.
- **Fabrication on error** — an error string reaching a generator *as data*.
- **Unreachable guard** — the right safety check, matching the wrong strings.
- **Dead fallback** — a graceful degradation path that can never execute.
- **Failure invisible to logs** — the failure path writes no record.
- **Cross-tenant leakage** — the highest-severity finding available.

---

## Phase 4 — Triage & adversarial verification

**Do not report raw agent output.** LLM testers hallucinate bugs, and a report
that cries wolf gets the whole effort dismissed.

For each candidate finding:

1. **Reproduce** — rerun the exact input. Record **n/N** (a 3-in-10 bug is a
   different ticket from a 10-in-10 bug, and *both* are real).
2. **Refute** — a fresh agent, with no stake in the finding, tries to prove
   it wrong. Prompt it to *default to refuted when uncertain*. For anything
   high-stakes, use several verifiers with **different lenses** (correctness /
   security / does-it-actually-reproduce) rather than N identical ones.
3. **Ground** — find the code path that explains it. A finding with a
   mechanism is a ticket; one without is a lead.
4. **Classify** — `CONFIRMED` (reproduced + mechanism) vs `PLAUSIBLE`
   (observed, unexplained). Report both, labelled. Never blur them.
5. **De-duplicate to root cause.** Ten symptoms from one cause is **one**
   finding with ten repros — not ten tickets. This is the single biggest
   quality difference between a useful report and a noise dump.

---

## Phase 5 — The findings report

Template: [references/report-and-backlog.md](references/report-and-backlog.md).

Structure:

- **Executive summary** — root causes (not symptom counts), the trust verdict,
  the one thing to fix first. Written for someone who won't read further.
- **Coverage** — what was tested, what wasn't, and **why**. State the gaps;
  silent truncation of scope reads as "we covered everything".
- **Findings**, ranked by damage: each with severity, repro (exact input +
  n/N), evidence, mechanism (`file:line`), user-visible impact, `CONFIRMED`/
  `PLAUSIBLE`.
- **Reliability baseline** — the measured numbers: pass rate per intent,
  variance per input, end-to-end success per persona. This is the "before"
  that every future change is compared against. If you measure nothing else,
  measure this.
- **Caveats** — what you couldn't test, what's estimated vs measured.

**Report honestly.** If the system is broadly fine, say so plainly. If it's
not, don't soften it. Distinguish measured from estimated everywhere.

---

## Phase 6 — The backlog (the action plan)

Every finding becomes a ticket with **success criteria that are testable** —
the report is only worth what gets fixed.

- **Epics** by root cause, not by symptom.
- **Each ticket:** problem · evidence (`file:line` + repro) · proposed fix ·
  **success criteria** · severity · estimate · dependencies.
- **Success criteria must be executable**, and the test that proves it is
  part of the ticket. Not *"multi-entity queries work"* but:

  > *Given a request naming 3 workers where 1 does not exist, the response
  > reports per-entity outcomes for all 3 and explicitly states that the
  > third was not found. Metamorphic decomposition test `md_decomp_03`
  > passes 10/10.*

- **Every finding becomes a permanent regression case (L10).** This is the
  ratchet: the suite grows monotonically and the same bug cannot return. A
  finding that doesn't become a test will be found again by a user.
- **Name the cheap wins.** Some findings are one line. Rank by
  damage-prevented ÷ effort, and say which ship today.

---

## Outputs

```
<workspace>/
  system-map.md            # Phase 0 — recon, the basis for everything
  test-plan.md             # Phases 1-2 — scenario space + layer mapping
  runs/                    # raw transcripts, inputs, run IDs (redacted)
  findings-report.md       # Phase 5 — the diagnosis
  backlog.md               # Phase 6 — the action plan, with success criteria
  regression/              # the permanent suite: every finding, as a test
```

## Definition of done

- [ ] Recon map exists and the test plan is traceable to it.
- [ ] Scenario space enumerated; sampling decisions explicit, not accidental.
- [ ] Every applicable layer ran — including **metamorphic** and
      **simulated-user**, on the **real** system.
- [ ] Every finding: reproduced (n/N), adversarially verified, grounded in a
      mechanism, labelled CONFIRMED/PLAUSIBLE.
- [ ] Findings de-duplicated to **root causes**.
- [ ] Reliability baseline measured and recorded.
- [ ] Report (diagnosis) + backlog (cure) as separate artifacts.
- [ ] Every finding has a regression case; success criteria are executable.
- [ ] Coverage gaps stated out loud.

## Tips & gotchas (learned)

- **Silent wrongness > crashes.** Budget your effort accordingly. Crashes get
  fixed on their own; silent wrongness ships to customers for months.
- **Test the honesty layer explicitly.** "Does it admit failure?" is a test
  case, not a nice-to-have. Most agentic systems route disclosure through the
  LLM as a *suggestion* and failure through a log line as a *shrug* — so the
  user never learns either.
- **Check that a fix is reachable and active**, not just merged. Ask: is the
  code routed to, and is the config that enables it set in the target env?
- **Scheduled/background paths are the least tested and most trusted.** No
  human reads them before the customer does. Test them first, not last.
- **Boundaries are where the bugs are:** midnight, month/year ends, DST, empty
  sets, single rows, exactly-N, duplicates, NULLs.
- **A relative window resolved twice must agree.** If "today" depends on when
  or where you ask, everything downstream is wrong.
- **N=1 proves nothing.** Run repeatedly; report frequency. Intermittent bugs
  are real bugs and get dismissed without n/N.
- **Prefer relations over judges.** A metamorphic relation is deterministic,
  cheap, and can't hallucinate. Use an LLM judge only where no relation exists.
- **Pin what you can.** Unpinned sampling parameters on a *routing* or
  *classification* call convert a decision into a coin flip. Check them in
  recon — every provider client, not just the obvious ones.
- **Don't let the harness become the product.** Ship findings early and often;
  a perfect harness delivered after the incident is worthless.
