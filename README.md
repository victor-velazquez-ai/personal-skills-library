# personal-skills-library

Reusable [Claude Code](https://claude.com/claude-code) **skills** — playbooks
that capture *how* to do recurring heavy-lift engineering work, so the process
is repeatable instead of reinvented.

Each skill is a folder with a `SKILL.md` (`name` + `description` frontmatter)
and optional `references/` for depth. Drop a folder into `.claude/skills/` and
Claude Code will load it when the description matches what you're asking for.

---

## Skills

| Skill | Use it when… |
|---|---|
| **[agentic-system-test](skills/agentic-system-test/SKILL.md)** | You need to exhaustively test an agentic / LLM system the way real users break it — recon, full scenario space, metamorphic + simulated-user testing, adversarial and chaos layers → findings report + ticketed backlog. |
| **[module-planning](skills/module-planning/SKILL.md)** | You need to turn a feature idea or design brief into an enterprise **design plan** and then an **implementation backlog** (epics → stories → tickets, Jira-ready). |

---

## agentic-system-test — the short pitch

Agentic systems ship bugs that testing never caught, because they aren't
deterministic and the interesting failures **aren't crashes**. A crash
announces itself. An agentic system instead returns a **confident,
well-formatted, wrong answer** — and nothing in the stack notices. So the bugs
get found by users, in production, one trust-destroying incident at a time.

Traditional testing can't reach them:

- **No exact oracle.** You can't assert `output == expected` when the wording
  changes every run — so teams test the plumbing and never the answer.
- **Combinatorial surface.** persona × phrasing × data state × history ×
  permissions. Users explore it exhaustively; test suites sample it.
- **Silent degradation.** Partial results look complete. Empty results look
  like answers. An error string can become narrative prose.
- **Coverage lies.** Tests can be green on code that cannot execute.

**The counter-move:**

> You cannot assert exact outputs. So assert three other things:
> **invariants** that hold for every input, **relations between outputs**
> (metamorphic testing — the answer to "no oracle"), and **honesty** — that
> the system's claims about its own scope, coverage and failures are true.
>
> And hunt the right bug class: **silent wrongness beats crashes.**

The skill runs: **recon** (understand the app first — non-negotiable) →
**scenario space** → **11 test layers** → **adversarial verification of every
finding** → **report** → **backlog with executable success criteria**. It fans
out sub-agents to drive the product as personas and to attack the prompts.

**Contents:**

| File | What's in it |
|---|---|
| [`SKILL.md`](skills/agentic-system-test/SKILL.md) | The process, end to end |
| [`recon-checklist.md`](skills/agentic-system-test/references/recon-checklist.md) | Understand the app before testing it — plus the "recon smells" that are findings on their own |
| [`test-taxonomy.md`](skills/agentic-system-test/references/test-taxonomy.md) | The 11 layers, what oracle each uses, what belongs in CI |
| [`metamorphic-relations.md`](skills/agentic-system-test/references/metamorphic-relations.md) | 14 relations that catch real bugs with **no expected answer needed** |
| [`failure-patterns.md`](skills/agentic-system-test/references/failure-patterns.md) | Field guide to how LLM systems actually fail, and how to detect each |
| [`simulated-users.md`](skills/agentic-system-test/references/simulated-users.md) | Persona agents that drive the real product and judge the transcript |
| [`report-and-backlog.md`](skills/agentic-system-test/references/report-and-backlog.md) | Turning findings into an action plan with testable success criteria |

### A taste — metamorphic testing

The technique most under-used on LLM products. Instead of asserting *what* the
output is, assert a **relation between the outputs of two related inputs**. You
never need to know the right answer:

> If `ask("who was on site today")` and `ask("list the people present today")`
> return different rows, one is wrong. **You don't need to know which** to know
> you have a bug.

Relations that repeatedly find real production bugs:

- **Scope monotonicity** — adding a filter must never *increase* results.
  Catches retry loops that quietly relax constraints until something returns.
- **Decomposition** — `ask(A and B)` == `ask(A) ∪ ask(B)`. Catches silently
  dropped entities in multi-entity requests.
- **Surface parity** — the same question via chat / API / scheduled must return
  the same data. Background paths lack context the interactive path has, and
  nobody watches them.
- **Temporal stability** — a relative window ("today") resolved at different
  clock times *within* that window must resolve identically. Catches the whole
  timezone-drift bug family.
- **Permission monotonicity** — narrower permissions never return more.

They're deterministic, cheap, can't hallucinate, and — unlike golden answers —
**don't rot as the data changes**. Which is why they belong in CI on day one.

---

## Using a skill

```bash
# per project
mkdir -p .claude/skills
cp -r skills/agentic-system-test .claude/skills/

# or globally
cp -r skills/agentic-system-test ~/.claude/skills/
```

Then just ask — *"test this agent exhaustively and give me a report and a
backlog"* — and Claude Code loads it from the description. Or invoke directly
with `/agentic-system-test`.

The skills are plain markdown: useful to a human reading them as a playbook,
even with no agent involved.

---

## Conventions

- **`SKILL.md`** = the spine: when to use, the process, outputs, definition of
  done, gotchas. Keep it scannable.
- **`references/`** = the depth, loaded on demand rather than up front.
- **Ground claims in evidence** (`file:line`), separate **diagnosis** from
  **cure**, and make **success criteria executable**.
- Skills are **general** — no employer, client, or project specifics.

## License

MIT — see [LICENSE](LICENSE).
