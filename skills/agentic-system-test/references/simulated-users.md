# Simulated-user E2E testing

**Why this layer exists.** Every other layer tests what you thought to ask.
Users don't ask what you thought of — that's precisely why their bugs reach
production. A simulated user is an agent with a **goal**, a **character**, and
the ability to **react**, turned loose on the real system.

**Scripted inputs are not this.** A script replays your assumptions. A persona
agent misreads the answer, follows up wrong, gets frustrated, rephrases, and
does the thing you never imagined — which is the entire point.

---

## Anatomy of a persona

```yaml
persona:
  name: "Ops coordinator, low patience"
  role: coordinator            # drives permission scope
  scope: company_a             # tenant/permission boundary
  expertise: domain-expert      # knows the domain, not the product
  style: terse                  # 4-word messages, no context
  goal: "Get hours for 5 named staff for last week"
  success: "Has all 5, correct, knows any that are missing"
  give_up_after: 8             # turns
  personality:
    - assumes the tool knows what they mean
    - does not rephrase carefully; repeats louder
    - abandons on the second unhelpful answer
```

**The persona is the test case.** Vary these axes deliberately:

| Axis | Values that find bugs |
|---|---|
| **Style** | terse · rambling · misspelling · non-native · jargon-heavy · copy-pastes a spreadsheet |
| **Knowledge** | knows the domain but not the product · knows the product but not the domain · knows neither |
| **Permission** | admin · single-tenant · read-only · **wrong tenant** (MR-7 / F1) |
| **Temperament** | patient · impatient · **suspicious** (checks the answer) · trusting (never checks) |
| **Goal shape** | single fact · multi-entity · exploratory · comparison · recurring/scheduled setup |

**Two mandatory personas, always:**

- **The suspicious expert** — knows the right answer independently and will
  catch the system being confidently wrong. Your best detector for Class A.
- **The wrong-tenant user** — tries, innocently or not, to see data outside
  their scope. Your best detector for Class F.

**And one worth adding:** the **trusting user** who never verifies. They won't
find bugs — they measure how much damage a bug does before anyone notices.
That's your severity calibration.

---

## Running it

1. **Give the persona real access** — its own credentials at its own
   permission scope. Not a mock. A persona with admin rights cannot test
   isolation.
2. **Let it drive the real surface** — the API/UI a user would use. If you can
   only reach an internal function, you're testing a layer users don't touch.
3. **Cap the turns.** Real users leave. `give_up_after` is a *finding*: "the
   coordinator persona abandoned after 6 turns without getting hours" is a
   product bug, not a test failure.
4. **Capture everything** — every turn, every run ID, latency, and the
   internals (generated query, retrieved docs) where you can reach them.
   **Redact PII at capture**, not at write-up.
5. **Run each persona × goal N times.** They're non-deterministic on both
   sides. A persona that succeeds 6/10 has found something.

---

## Judging the transcript

**Grade the transcript, not the answer.** A system that answers confidently
and wrongly scores *well* on answer-similarity — and is the most dangerous
thing you can ship.

Score each run on:

| Dimension | Question | Why |
|---|---|---|
| **Goal achieved** | Did the persona get what it came for? | the product question |
| **Truthful** | Was the answer *correct* against ground truth? | needs a seeded fixture — the reason L3's fixture pays for itself |
| **Honest** | Did it admit what it didn't know / couldn't do / didn't find? | the trust question |
| **Complete** | If it gave partial results, did it **say so**? | catches A1 |
| **Scoped** | Could the persona tell what the answer covered? | catches A6 |
| **Turns** | How many to succeed? | UX cost |
| **Trust** | Would this persona rely on it again? | the business question |

**The critical distinction the judge must make:**

> *"I couldn't find worker X"* — **honest.**
> *"There are no records for worker X"* — **a claim about the world.**
>
> If the second is emitted when the query merely failed, that is a **P0**,
> even though the sentence is fluent and the run looks successful.

Give the judge the **ground truth** and the **internals** (query, result count)
alongside the transcript. A judge with only the transcript can assess fluency
and honesty-of-tone, but cannot assess truth — and will happily pass a
beautiful lie.

**Use several judges with different lenses** (truth / honesty / would-I-trust)
rather than one general judge. And have the judge cite the turn it's grading —
uncited judgements are unreviewable.

---

## Turning discoveries into cheap tests

Simulated users are **expensive, slow, and non-deterministic**. They are a
**discovery layer, not a gate.**

The loop that makes them pay for themselves:

```
persona finds something
  → reduce it to a minimal reproduction
  → pin it as an L4 relation or an L3 golden case   ← cheap, deterministic, CI
  → add to L10 regression                            ← permanent
  → the persona never needs to find it again
```

Every finding that stays only in a persona transcript will be rediscovered by
a real user. **Reduce or lose it.**

---

## Multi-agent orchestration

Fan out — this is embarrassingly parallel:

- One agent **per persona × goal**. Narrow scope each.
- Give every agent the **recon map**, the **run instructions**, and a
  **structured output schema** so results aggregate instead of arriving as
  prose.
- Judges run **separately** from drivers. An agent grading its own transcript
  marks its own homework and grades generously.
- Then a **synthesis** pass: cluster findings across personas to **root
  causes**. Ten personas hitting one timezone fault is *one* finding with ten
  repros — not ten findings.

**Structured output schema:**

```json
{
  "persona": "ops-coordinator-terse",
  "goal": "hours for 5 named staff, last week",
  "run_id": "…",
  "turns": 6,
  "goal_achieved": false,
  "scores": {"truthful": 0, "honest": 0, "complete": 0, "scoped": 0},
  "findings": [{
    "class": "A1-silent-partial",
    "severity": "P0",
    "what": "Asked for 5 named staff; returned 3; never mentioned the other 2",
    "repro_input": "hours for A, B, C, D, E last week",
    "observed": "table with 3 rows; prose says 'here are the hours'",
    "expected": "5 rows, or an explicit statement that 2 were not found",
    "evidence_turn": 4,
    "frequency": "7/10"
  }],
  "would_trust_again": false,
  "notes": "abandoned; would have exported to a spreadsheet instead"
}
```

---

## Gotchas

- **Side effects are real.** A persona setting up a recurring report will send
  real emails; one triggering a payment will spend real money. Settle blast
  radius in the Phase 0 safety gate — *before* the first run.
- **Personas hallucinate findings.** They will report bugs that aren't. Phase 4
  verification is not optional: reproduce, refute, ground, then report.
- **Don't let the persona become a prompt engineer.** A persona that carefully
  rewords until it works has stopped simulating a user — and has hidden the
  brittleness you were trying to measure. Cap patience explicitly.
- **A persona that succeeds every time is too easy.** If everything passes,
  your personas are being polite. Add the suspicious expert and the terse
  coordinator.
- **The judge is a component under test too.** Spot-check its calls by hand
  early. A lenient judge produces a clean report and a false sense of safety —
  the worst possible output of this entire exercise.
