---
name: speckit-feature-lifecycle
description: Run a feature through a Spec Kit workflow in a governed repository — specify, clarify, plan, tasks, implement, analyze, converge — honouring a versioned constitution, an active-feature pointer, and merge gates that must stay in agreement. Use when starting a new feature, resuming one, or when a spec/plan/tasks artefact set must be produced or audited.
version: 2.0.0
---

# Spec Kit feature lifecycle

A feature is a **folder of artefacts** that a human can review without reading code, plus the code that
satisfies them. The order is not ceremony: each artefact removes a class of rework that is expensive
later.

## When to activate

- Starting a new feature ("build X").
- Resuming: `.specify/feature.json` names the active feature; read it before touching anything.
- Auditing a feature whose spec and code disagree.
- Amending the constitution (a governance change, not a feature).

Skip for a bug fix under ~3 files. A typo does not need a spec; a schema change does.

## The flow

| Stage | Command | Produces | Gate before moving on |
|---|---|---|---|
| Specify | `/speckit.specify` | `spec.md` — user stories, requirements (FR-###), success criteria (SC-###) | No implementation detail; every requirement testable |
| Clarify | `/speckit.clarify` | amendments inside `spec.md` | Ambiguities are questions, not assumptions |
| Plan | `/speckit.plan` | `plan.md`, `research.md`, `data-model.md`, `contracts/`, `quickstart.md` | Constitution check passes; every contract names its route, props and test |
| Tasks | `/speckit.tasks` | `tasks.md` — ordered, one-test-one-task | Every requirement maps to ≥ 1 task |
| Implement | `/speckit.implement` | code + tests in the same commit | `bin/ci` equivalent gates pass |
| Analyze | `/speckit.analyze` | findings report | No requirement without a test, no test without a requirement |
| Converge | `/speckit.converge` | drift report | Spec, plan, tasks and code agree |

## The artefact contract

```
specs/NNN-kebab-name/
  spec.md          user stories, FR-### requirements, SC-### criteria, out-of-scope
  plan.md          technical approach, constitution check, phases
  research.md      decisions with the option that was rejected and why
  data-model.md    tables, columns, indexes, authorization scopes
  contracts/       one file per surface: routes.md, authorization.md, ai.md, design-language.md
  tasks.md         ordered tasks, each naming its test
  quickstart.md    how to see the feature working, by hand
  checklists/      review checklists for this feature
```

The numbers are the contract: a task cites an `FR-###`, a test names the `FR-###`/`SC-###` it proves,
and a contract quotes the requirement it implements. That is what makes `/speckit.analyze` mechanical
instead of a matter of opinion.

## The constitution is normative

Read it before planning, and quote it in the plan's constitution check:

```
I.   Inertia monolith, server-driven UI    -> no JSON API; props built server-side
II.  Skinny controllers, explicit boundaries -> one domain call per action
III. Test-first is mandatory (NON-NEGOTIABLE) -> test in the same commit, never deferred
IV.  Security and quality gates block merge  -> every gate green; nothing weakened, nothing suppressed
V.   Simplicity before abstraction           -> no dependency, cache or single-caller abstraction
```

A plan that violates a principle is rejected, not negotiated. Cutting scope is legal; relaxing a
principle requires an **amendment**: a separate commit that bumps the version and records a sync impact
report.

### Amendment format

```markdown
## Amendment 2026-11-08

**Principle III — Test-First.** Browser-level assertions may use Capybara selectors rather than
Inertia helpers only for interactions that have no prop (an overlay opening, a keystroke).
**Rationale:** the alternative was asserting a component's internal state, which is not a contract.
**Impact:** no existing test changes; the rule now says what the suite already does.
```

## Gates: one change, two files

The constitution requires every gate to exist in `bin/ci` (`config/ci.rb`) **and** in
`.github/workflows/ci.yml` in the same change. A gate added to one and not the other is a local green
build and a red pull request — see `merge-gate-parity`.

## Tasks that are actually implementable

A task names the file, the behaviour and the test:

```markdown
- [ ] T012 `app/services/tickets/reopen.rb` — refuse a reopen outside `reopen_days` with
      `:reopen_window_closed`; test in `test/services/tickets/reopen_test.rb` (FR-038)
```

Not implementable: "add reopen support". The task list is the plan's proof that the plan is complete.

## Out of scope is a deliverable

Every spec ends with what it will **not** build. The alternative is a feature that quietly grows, and a
placeholder in the navigation that eventually links to a 500. If the requirement lists it as out of
scope, it is not built — a visibly non-interactive marker ("segera") is the honest placeholder.

## Resuming a feature

1. `cat .specify/feature.json` — which feature is active.
2. `specs/<active>/tasks.md` — which tasks are unchecked.
3. `git log --oneline -10` — what has landed since the tasks file was written.
4. Reconcile before coding: a task that is already done is marked done, not re-implemented.

## References

- [references/artefact-contract.md](references/artefact-contract.md) — what each artefact must contain, with a skeleton
- [references/gates-and-ci.md](references/gates-and-ci.md) — the gate list, parity rule, and how to add one
