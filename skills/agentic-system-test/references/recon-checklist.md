# Phase 0 — Recon checklist

**You cannot test an app you don't understand.** Skipping this produces a
report full of noise: findings that are working-as-intended, missed bug
classes, and a test plan that fits some other system.

Ground every answer in real code/config with `file:line`. Assumptions here
become false findings later.

**Deliverable:** `system-map.md`. Everything downstream is derived from it and
must be traceable to it.

---

## 1. The product

- [ ] **What does it promise?** One paragraph, in the user's words.
- [ ] **Who uses it?** Roles, expertise, permission scopes.
- [ ] **What do they do with the output?** Decisions? Money? Compliance? This
      sets severity: a wrong number in a billed report is not a wrong number
      in a demo.
- [ ] **What does "wrong" cost?** The blast radius of a silent error.
- [ ] **Known complaints?** Support tickets, meeting notes, bug reports. **The
      best test cases are the ones users already found** — and they come with
      a reproduction and a stakeholder.

## 2. Surfaces (entry points)

For **each** — chat/UI, REST/GraphQL, scheduled/cron, batch, webhook, CLI:

- [ ] What can be asked through it?
- [ ] What context does it carry? (auth, tenant, timezone, locale, history)
- [ ] **What context does it *lack* that another surface has?** ← the parity
      bug lives here (MR-5). Background paths typically lack auth/locale
      context the interactive path has, and nobody watches them.
- [ ] Who sees the output, and how fast would they notice it was wrong?

> **Rank surfaces by (trust × unwatchedness).** A scheduled report emailed to
> a customer is trusted absolutely and read by no engineer before them. That's
> where the damage is. Test it first, not last.

## 3. The graph — what actually executes

- [ ] Nodes, edges, routing decisions. **Draw it** (mermaid).
- [ ] **Where are the edges declared?** If routing hides inside node code
      rather than the graph definition, the stated architecture and the real
      one will differ.
- [ ] **Which nodes are unreachable?** Registered-but-dead code is extremely
      common and inflates apparent complexity. Confirm each node is routed to.
- [ ] Cycles? Recursion/iteration limits? Can it spin?
- [ ] Retry loops — **what do they change on retry?** (See A4: retries that
      relax constraints are answer-changing machines.)
- [ ] What happens to partial state on a mid-graph exception?

## 4. Non-determinism inventory

**Every** model client, not just the obvious one:

| Call site | Model | Temp/sampling | Job (route/classify/generate) | Structured output? |
|---|---|---|---|---|

- [ ] **Any unpinned sampling?** Especially on routing/classification. One
      unpinned client among many pinned ones is a classic (C1).
- [ ] **How many LLM *gates* per request?** They multiply (C2). Five at 0.95
      ≈ 0.77 end-to-end. This is usually the flake budget.
- [ ] Free-text parsing of model output (`json.loads` + fence-stripping)? (C3)
- [ ] Prompt sizes; how much history is injected? Oversized prompts degrade
      instruction-following, especially on routing calls.
- [ ] Which decisions are LLM-made that **could be deterministic**? (date
      windows, output format, routing on an explicit flag) Each one is a
      removable coin flip.

## 5. Truth: the data model

- [ ] What does the store actually hold? Granularity, units, **timezone
      basis** (stored local or UTC?), soft deletes, NULLs.
- [ ] **What does the system claim it can answer vs what the schema can
      support?** Questions unanswerable from the data are a distinct bug class
      — and the system will usually answer them anyway, emptily.
- [ ] How is the schema described *to the model*? A raw dump, or a curated
      description? **An inaccurate schema description produces
      plausible-but-empty queries**, and looks like a data problem.
- [ ] Boundary semantics: how is a record spanning midnight stored? Which day
      owns it?

## 6. Tenancy & permissions

- [ ] Isolation boundary — tenant/project/company/user.
- [ ] **Where is it enforced?** Prompt rule / query filter / DB role / RLS.
      *A prompt rule is not access control.*
- [ ] Can the model construct a query that crosses it?
- [ ] Least-privileged and most-privileged personas, for MR-7.

## 7. Existing safety net — and whether it's real

- [ ] Tests: how many, testing what, **do they run** on the branch that ships?
- [ ] **Do they test reachable code?** (D1 — a test bypassing dispatch is
      worse than no test.)
- [ ] Evals: exist? gating? informational-only (`allow_failure`)? on the
      shipping branch?
- [ ] CI: **read the pipeline output, not the config.** Which jobs actually
      ran on the last merge to the deploy branch?
- [ ] **Branch topology:** does the branch that ships to production carry the
      CI/tests/tracing? (D3 — safety equipment on the wrong branch is a
      recurring, invisible, expensive failure.)
- [ ] Tracing/observability: can you take a past user complaint and retrieve
      that exact run — its inputs, its query, its result? **If not, testing
      finds bugs you cannot then diagnose.**
- [ ] Run history: is there a record of what each run returned, not just that
      it ran?

## 8. How to run it

Resolve **before** Phase 3. Discovering you can't drive the system mid-test
wastes the whole fan-out.

- [ ] The exact command / endpoint / UI path to submit a request.
- [ ] Auth: how to get a token; tokens for **each** persona.
- [ ] Environments: local, QA/staging, UAT, prod. Which are safe?
- [ ] Can you seed data? Reset it? Is data shared with other testers?
- [ ] How do you read the result *and* the internals (query, run ID, trace)?
- [ ] Rate limits, quotas, cost per call.
- [ ] Latency — a slow system caps how exhaustive you can be. Budget it.

## 9. Safety gate — settle in writing

- [ ] **Which environment am I allowed to touch?** Never prod without
      explicit, specific consent.
- [ ] **What has side effects?** Emails, payments, credits, notifications,
      pages, writes, third-party calls. **Establish blast radius before the
      first run.** Simulated users *will* trigger real actions.
- [ ] **Is the data real?** Real PII in transcripts becomes a leak in your
      report. Redact at capture.
- [ ] **Budget ceiling** — exhaustive × LLM = real money. Agree a number.
- [ ] **Who to tell** if you find something critical mid-run, and when to stop.

---

## Recon output template

```markdown
# System map — <product>

## What it is
<promise, users, stakes, cost of being wrong>

## Surfaces
| Surface | Intents | Context carried | Context missing | Watched? |

## Graph
<mermaid>
Unreachable nodes: <list>       # dead code inflates apparent complexity
Retry loops: <what changes on retry>

## Non-determinism
| Call site | Model | Temp | Job | Structured? |
LLM gates per request: <n> → naive end-to-end ≈ <product of p>
Unpinned: <list>                # usually the cheapest fix available

## Data model & truth
<granularity, timezone basis, NULLs, boundary semantics>
Claims vs schema gaps: <questions it can't actually answer>

## Tenancy
<boundary, enforcement point, is it real access control?>

## Existing safety net
Tests: <n> — reachable? <y/n> — run on shipping branch? <y/n>
Evals: <n> — gating? <y/n>
Tracing: can I retrieve a past run? <y/n>
Branch topology: does prod's branch carry the CI/tests? <y/n>

## How to run
<commands, auth per persona, safe env, how to read internals>

## Safety
Allowed env: <> · Side effects: <> · Real data: <> · Budget: <>

## Test plan implications
<the 5-10 things this map says to test first, and why>
```

---

## Recon smells — findings before you've run a test

Note these; several are P0 on their own:

- A model client with no sampling setting, next to clients that have one.
- A retry that instructs the model to loosen constraints.
- A summary/narrative generated without access to the data it describes.
- An error path and an empty path converging on one branch.
- A failure path that returns before its audit write.
- A guard matching on error strings by prefix.
- A `return` below a call that always raises.
- A config-driven fix whose config is unset in the deployed env.
- A test calling an internal that no dispatch routes to.
- CI/tests/tracing living on a branch that never reaches production.
- A health endpoint returning a hardcoded literal.
- An isolation boundary enforced only by a prompt instruction.
