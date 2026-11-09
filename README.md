# Spec Kit feature lifecycle

Run a feature through a Spec Kit workflow in a governed repository — specify, clarify, plan, tasks, implement, analyze, converge — honouring a versioned constitution, an active-feature pointer, and merge gates that must stay in agreement. Use when starting a new feature, resuming one, or when a spec/plan/tasks artefact set must be produced or audited.

One skill, one repository. Part of a twelve-skill pack extracted from a production Rails 8 +
Inertia + React helpdesk monolith — the opinionated layer no public Rails or shadcn skill can know.
Version `2.0.0` tracks the source constitution.

## Install

```jsonc
// skills-lock.json
{
  "version": 1,
  "skills": {
    "speckit-feature-lifecycle": {
      "source": "mahardikalgw/ruby-speckit-feature-lifecycle",
      "sourceType": "github",
      "skillPath": "SKILL.md"
    }
  }
}
```

Or clone and symlink:

```bash
git clone git@github.com:mahardikalgw/ruby-speckit-feature-lifecycle.git
ln -s "$PWD/ruby-speckit-feature-lifecycle" .claude/skills/speckit-feature-lifecycle
```

## When to activate

- Starting a new feature ("build X").
- Resuming: `.specify/feature.json` names the active feature; read it before touching anything.
- Auditing a feature whose spec and code disagree.
- Amending the constitution (a governance change, not a feature).

## What's inside

```
ruby-speckit-feature-lifecycle/
  SKILL.md
  bin/check-skill
  references/artefact-contract.md
  references/gates-and-ci.md
```

`SKILL.md` is a router (kept under 250 lines) — the depth lives in `references/`, and the
copy-in code lives in `assets/`.

## Validate

```bash
./bin/check-skill
```

Checks frontmatter, the activation section, link resolution, orphaned references, script
executability, leftover placeholders, and — when the source app is checked out next to this
repository — that vendored scripts still match it byte for byte. Runs in CI on every push.

## Related skills

- [`merge-gate-parity`](https://github.com/mahardikalgw/ruby-merge-gate-parity) — one gate list in every file, a script contract, and local/CI triage
- [`inertia-vertical-slice`](https://github.com/mahardikalgw/ruby-inertia-vertical-slice) — route → skinny controller → query/service → typed props → Inertia page → test
- [`scoped-authorization`](https://github.com/mahardikalgw/ruby-scoped-authorization) — scopes, 404-not-403, explicit public opt-outs, isolation by construction

## Licence

MIT — see [LICENSE](LICENSE).
