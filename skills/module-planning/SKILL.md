---
name: module-planning
description: >
  Turn a feature/module idea or a design brief into an enterprise-grade
  DESIGN PLAN (markdown source-of-truth + HTML viewer + Word doc) and then an
  IMPLEMENTATION BACKLOG (epics → stories → tickets, Jira-ready). Use when the
  user asks to "plan", "design", "make an enterprise plan for", or "turn this
  brief into a plan/backlog" for a module, capability, or new system. Grounded
  in the target repo's real architecture, with decisions made — not deferred.
---

# Module planning — design plan → action plan

A repeatable process for taking a capability from idea/brief to a reviewed
design and a ticketed backlog. Two artifacts, in order:

> **The design plan is the spec. The backlog is the action plan that makes it
> real.** Keep them as separate documents.

## When to use

- A design brief or requirement set exists (or the user describes one) for a
  new module/capability.
- The user wants an enterprise plan, an architecture, and/or a Jira backlog.
- Not for small bug-fixes or single-PR changes — that's normal dev work.

## Inputs to gather first

1. **The brief / requirements** — read it completely, end to end.
2. **The principals' intent** — who asked, and why (exec/PM notes). *The
   origin in their own words* is what makes the executive summary land.
3. **Hard constraints** — "cloud-native on X", "must not slip deadline Y",
   "quality > latency > cost". **Honour these as decisions, not options.**
4. **The current architecture** — explore the repo so the plan is grounded.
   The plan must say how this module fits *this* system, not a generic one.

## The process

### Step 0 — Set up the workspace
- Work in a dedicated folder (a git-ignored scratch area is ideal for
  in-progress planning).
- If rendering to HTML/Word, copy the build scripts from a sibling effort and
  update the title/header defaults.

### Step 1 — Explore & ground (do NOT skip)
- Read the brief; list requirements, consumers, constraints, phasing, success
  metrics.
- Explore the codebase for the **integration seams**: how do the existing
  agents/tools/modules work, what already exists (even placeholders), what
  will this replace or extend?
- Explore **sister repos/services** if the module is shared — confirm how they
  would consume it.
- Capture findings as **`file:line` references**. Every "current state" claim
  in the plan must be backed by real code.

### Step 2 — Write the design plan (markdown is the source of truth)

A proven section skeleton (`## 0` … `## 15`):

0. **Executive summary** + **0.1 Key terms (glossary)** — define every acronym
   in plain language so the doc stands alone.
1. **Why now — current state** — grounded in the repo (`file:line`), what's
   broken without this, a diagram of today's mess.
2. **What we are building** — one module, many consumers; design principles.
3. **System context** — consumers table + context diagram.
4. **The architecture decision** — present the candidates, but **make the
   call** with rationale, honouring the hard constraint. Keep one swap open
   behind the contract.
5. **Content / domain model** (if data-bearing) — what it holds; global vs
   tenant scope.
6. **Module architecture & components** — component diagram + responsibility
   table.
7. **The public contract** — endpoints, request/response schemas, in-process
   tool signatures. **The contract is the real product.**
8. **Internals** — how it actually works (pipeline, cascade, state machine),
   with diagrams.
9. **Domain-specific internals** — e.g. entity resolution, verification.
10. **Cross-cutting concerns** — reliability, security, multi-tenancy,
    versioning/reproducibility, latency/cost, observability.
11/12. **Evaluation & SLOs** — eval-gated: scorers, dataset, targets. Tie to
    the system-wide eval plan where they overlap.
13. **Integration & migration** — per consumer, with before/after diagrams;
    fold in known audit findings on the touched path.
14. **Phased roadmap** — eval-gated phases, each shipping a before/after
    number; a Mermaid `gantt`.
15. **Risks & open questions** — the scope drivers to resolve in design.

End with a **bottom-line** paragraph.

**Writing rules:**
- Ground every current-state claim in code (`file:line` links).
- **Make decisions** where the user gave a constraint. Don't leave it "to
  evaluate" if they've already told you the direction.
- Diagrams: Mermaid — `flowchart TB` for architecture, `sequenceDiagram` for
  flows, `stateDiagram-v2` for lifecycles, `gantt` for the roadmap. Use a
  consistent colour `classDef` palette.
- Schemas: real JSON/SQL/code blocks for every contract surface.
- **Readability (applies to EVERY `.md` — plan and backlog):** the reader must
  be able to *scan*. One idea per line; a blank line between blocks. **Never**
  chain labelled fields inline (`**Why:** … **Scope:** …`) — each labelled field
  is its own line with its content beneath it. Put a `---` rule between repeated
  items (each ticket, each epic). Prefer short paragraphs + bullet lists over
  dense prose; fenced code/schema blocks get a blank line before and after. A
  wall of run-on text is a defect, not a style choice.

### Step 3 — Render the deliverables (optional but recommended)
- **HTML** = the review artifact (auto-TOC sidebar from `##` headings, Mermaid
  rendered). Verify the nav matches the body.
- **DOCX** = the delivery artifact (cover, real TOC, page numbers).
- Confirm the page count for one-pagers.

### Step 4 — Review & challenge loop
- The human reads the rendered doc, challenges, asks for changes.
- Iterate the **markdown** — never hand-edit generated HTML/DOCX — and
  rebuild. Keep MD ↔ HTML ↔ DOCX in sync by always regenerating.
- Common asks: define the jargon, add a diagram for a stage, explain each
  block, make sections consistent, add a missing capability.

### Step 5 — Translate to the implementation backlog (the action plan)

Only after the design is accepted.

- **Conventions block:** issue types (Epic/Story/Sub-task/Spike), Fibonacci
  points, P0–P3, components, labels, the assumed stack, a global
  Definition-of-Ready and Definition-of-Done.
- **Epic summary table** (epic | phase | goal | rolled-up points | priority) +
  a **dependency diagram** + a **traceability table** (design § → epic/story)
  that proves the backlog covers the whole design.
- **~6 epics**, each with a goal and success criteria, then **stories/tickets**.

  Every ticket uses this exact shape — each field a labelled block on its own
  line (never inline), and tickets are separated by a `---` rule:

  - **Title** + a one-line meta (`component · priority · points · depends`).
  - **Why** — the problem, grounded in `file:line`.
  - **Technical scope** — exactly what to build/change: real paths, code, commands.
  - **Tasks** — a sub-task checklist.
  - **Success criteria** — *measurable, testable* "done", distinct from the tasks:
    an outcome you can assert / `grep` / run, not a restatement of the work.
  - **Verification** — how a reviewer proves it (the exact command or check).

  A thin, generic ticket is a bug. If a field can't cite a path or state a
  measurable outcome, the ticket isn't ready.
- A **suggested sprint slicing** and a crisp **"module is live" exit bar**.

## Outputs

```
<workspace>/
  <module>-plan.md                        # design plan (source of truth)
  <module>-plan.html                      # review artifact
  <Module>.docx                           # delivery artifact
  <module>-implementation-backlog.md      # the action plan
  README.md                               # folder index
```

## Definition of done

- [ ] Design plan grounded in real code (`file:line`); constraints honoured as
      decisions; diagrams + schemas for every surface.
- [ ] Renders cleanly; nav/TOC matches the body.
- [ ] Reviewed & challenged by the human; feedback folded in.
- [ ] Evaluation section ties to the system-wide eval plan.
- [ ] Backlog covers the whole design (traceability table proves it), is
      Jira-ready, and names the critical-path dependency to schedule first.
- [ ] Every `.md` is **scannable**: labelled fields on their own lines, a `---`
      between tickets/epics, no run-on walls of text — and every ticket has a
      *measurable* **Success criteria** plus a **Verification** step.

## Tips & gotchas

- **Make the call.** A plan that lists options without deciding pushes the work
  back to the reader. If they gave you a constraint, it's a decision.
- **Keep the markdown canonical** — every other format is generated from it.
- **The contract is the product** — design so internals (vendor, model, store)
  can swap behind a stable API/tool contract.
- **Separate spec from plan.** The design says *what and why*; the backlog says
  *who does what, in what order, and how we know it's done*.
- **Glossary early.** If a reader hits three undefined acronyms, they stop
  reading and the plan doesn't get approved.
- **Reading a `.docx` brief on Windows:** Word COM —
  `$w=New-Object -ComObject Word.Application; $d=$w.Documents.Open($path,$false,$true); $d.Content.Text | Out-File ...; $d.Close($false); $w.Quit()`.
- **COM file locks:** if a build can't overwrite a `.docx`, it's open in Word.
  Don't kill the user's window — ask them to close it, or write a new filename.
- **Use absolute paths** with Word COM (`SaveAs2` rejects some relative paths).
