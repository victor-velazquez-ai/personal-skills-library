# The report and the backlog

Two artifacts, in order, never merged:

> **The report is the diagnosis. The backlog is the cure.**

A report with fixes buried in the narrative doesn't get executed. A backlog
without a report doesn't get funded. Keep them separate.

---

## Part 1 — The findings report

### Structure

```markdown
# <System> — test findings

**Tested:** <what, which env, which build/commit>
**Method:** <layers run, personas, scenario count, runs executed>
**Date / Author**

## Executive summary
<Root causes — not symptom counts. The trust verdict. The one thing to fix
first. Written for someone who reads nothing else.>

## Coverage
<What was tested. What wasn't, and why. State the gaps.>

## Reliability baseline
<The measured numbers. The "before" for every future change.>

## Findings
<Ranked by damage. Each with the anatomy below.>

## Caveats
<Measured vs estimated. What you couldn't reach. Open questions.>
```

### The executive summary

Three things, in this order:

1. **Root causes, not symptoms.** "Four incidents, one cause" beats "17
   findings". If you found 17 symptoms of 3 causes, the summary says **3**.
2. **The trust verdict.** Can users rely on this today, and for what? The
   honest answer to "should we keep this in front of customers".
3. **The one thing to fix first**, with its cost. If it's cheap, say so —
   cheap fixes get done today; expensive ones get scheduled and forgotten.

Write it for someone who will read *only this*. No jargon, no file paths.

### Anatomy of a finding

```markdown
### F<n> — <symptom in the user's words>

**Severity:** P0 · **Class:** A1 silent-partial · **Status:** CONFIRMED
**Frequency:** 7/10 runs

**What the user sees:** <the experience, not the code>
**What's actually true:** <ground truth>
**Why it matters:** <the decision they'd make wrongly; the blast radius>

**Reproduce:**
1. As persona <X> with scope <Y>
2. Send: `<exact input>`
3. Observe: <exact observation>
Run IDs: <ids> · Env: <env> · Build: <sha>

**Mechanism:** <the code path — file:line — that explains it>
**Detected by:** <MR-4 decomposition / chaos: SES 500 / persona run>
**Proposed fix:** <shape of the fix, not the patch>
```

**Rules:**

- **Severity by damage, not by noise.** A silent wrong number outranks a
  crash. The crash gets fixed on its own; the wrong number ships for months.
  Ask: *would anyone notice if this stayed broken?* The scarier the answer to
  "no", the higher the severity.
- **Frequency always.** `7/10` is a different ticket from `10/10`, and both
  are real. Without n/N, intermittent findings get waved away as flukes.
- **`CONFIRMED` vs `PLAUSIBLE`, never blurred.** CONFIRMED = reproduced *and*
  mechanism found. PLAUSIBLE = observed, unexplained. Report both — labelled.
  One overstated finding costs the credibility of the other sixteen.
- **De-duplicate to root cause.** Ten symptoms of one bug is **one** finding
  with ten repros. This is the single biggest difference between a report
  that gets acted on and one that gets skimmed.
- **The user's words in the title.** "The report showed a whole week when I
  asked for today" — not "date window drift in the retry path". The mechanism
  goes in the mechanism field.

### The reliability baseline

The most reusable thing the report produces. Without it, nobody can tell
whether the next change helped.

```markdown
| Metric | Measured | Method |
|---|---|---|
| End-to-end success (all personas) | 62% | 40 persona-runs |
| Self-consistency (MR-2) | 7.4/10 | 20 inputs × 10 runs |
| Paraphrase invariance (MR-1) | 71% | 30 pairs |
| Surface parity (MR-5) | 4/12 diverge | 12 intents × 2 surfaces |
| Honest-failure rate (L8) | 2/9 faults disclosed | 9 injections |
```

**Label measured vs estimated everywhere.** An estimate presented as a
measurement is the same class of bug you're reporting.

### Coverage — say what you didn't do

```markdown
| Area | Covered | Gap |
|---|---|---|
| Intents | 12/12 | — |
| Personas | 5/8 | no read-only, no non-native, no wrong-tenant |
| Surfaces | 2/4 | batch and webhook untested — no safe env |
| Boundaries | partial | **DST untested — fixture has no transition** |
```

**Silent truncation of scope reads as "we covered everything."** If you
sampled, say what you dropped. The gap list is often the most valuable
paragraph in the document — it's next sprint's plan.

---

## Part 2 — The backlog

Every finding becomes a ticket whose **success criteria are executable**.

### Ticket shape

```markdown
### T<n> — <fix, imperative> *(P0 · 5 pts · <component>)*
> As a <user>, <the outcome they need>.

**Fixes:** F<n>, F<m>            # root cause → several symptoms
**Problem:** <one sentence>
**Evidence:** <file:line> · repro: `<input>` · 7/10 runs
**Proposed fix:** <shape>

**Success criteria:**
- [ ] <executable assertion>
- [ ] <executable assertion>
- [ ] Regression case `<id>` added and passing 10/10
- [ ] Metamorphic relation `<MR-n>` passes for this intent

**Dependencies:** <>
```

### Success criteria — the part that matters

**Not testable:**
> - [ ] Multi-entity queries work correctly

**Testable:**
> - [ ] Given a request naming 3 entities where 1 doesn't exist, the response
>       reports a per-entity outcome for all 3 and explicitly states the third
>       was not found.
> - [ ] "Found, zero results" and "not found" render as different messages.
> - [ ] `md_decomp_03` (MR-4) passes 10/10.
> - [ ] Regression case `reg_042` (the original repro) passes 10/10.

**The test that proves it is part of the ticket.** If a criterion can't be
executed, it will be marked done on vibes.

**For non-deterministic behaviour, criteria need a frequency.** "Passes" means
nothing when the system is stochastic — write `10/10`, or `≥9/10`, and say
which.

### Epics by root cause

Group by **cause**, not symptom. If one timezone fault produced four symptoms,
that's **one epic** with the fix and the four regression cases — not four
tickets that each half-fix it.

Typical epics:

| Epic | Contains |
|---|---|
| **Stop the bleeding** | one-line/config fixes that ship today |
| **Correctness** | the root-cause fixes |
| **Honesty** | scope disclosure, failure reporting, no-fabrication |
| **Observability** | can we see it fail, can we trace a complaint |
| **The gate** | regression suite + CI so it can't come back |

**Order by damage-prevented ÷ effort.** Name the cheap wins explicitly and
separate them — the config change that ships this afternoon should not be
buried among 8-point stories, or it waits two weeks for no reason.

### The ratchet

**Every finding becomes a permanent regression case, before it becomes a fix.**

- Write the failing test first. It proves the finding *and* the fix.
- The suite only grows. Never delete a case because the bug is fixed — that's
  when it's earning.
- The gate: **any change to a prompt, graph, or model config runs the suite or
  does not merge.**
- Set the threshold at the **measured baseline** and ratchet up. Starting at
  100% guarantees the gate is disabled within a week.

> A finding that doesn't become a test will be found again by a user. That is
> the whole reason this skill exists.

---

## Tone

- **Report honestly.** If it's broadly fine, say so plainly. If it's not,
  don't soften it. Both are useful; a hedge is not.
- **No blame.** "The retry loop relaxes the date filter" — not "someone wrote
  a bad retry". Findings describe code.
- **Distinguish measured from estimated**, everywhere, always.
- **Lead with the outcome.** The first sentence answers "what did you find",
  not "here is what I did".
- **Say the uncomfortable thing.** The most valuable finding is usually the
  one nobody wants in writing — the fix that shipped but never activated, the
  test suite that never ran, the guard that can't fire. Those are exactly the
  findings that prevent a repeat, and they're the reason you were asked.
