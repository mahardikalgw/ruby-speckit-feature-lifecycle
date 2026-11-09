# Gates and CI parity

## The gate list

```ruby
# config/ci.rb — evaluated top to bottom; the first failure stops the run
CI.run do
  step "Setup",                     "bin/setup --skip-server"
  step "Style: Ruby",               "bin/rubocop"
  step "Style: UI components",      "bin/check-ui-components"
  step "Style: Theme tokens",       "bin/check-theme"
  step "Security: Gem audit",       "bin/bundler-audit"
  step "Security: JS dependency audit", "npm audit --audit-level=high"
  step "Security: Brakeman code analysis", "bin/brakeman --quiet --no-pager --exit-on-warn --exit-on-error"
  step "Tests: Rails",              "bin/rails test"
  step "Tests: System",             "bin/rails test:system"
  step "Tests: Seeds",              "env RAILS_ENV=test bin/rails db:seed:replant db:test:prepare"
end
```

Order matters: setup (so a fresh clone can run everything), style (cheap, fails fast), security, then
tests (expensive), then seeds (mutates the test database, so nothing after it).

## The parity rule

Every gate exists in **three** places, updated in the same change:

| File | Purpose |
|---|---|
| `config/ci.rb` | the developer-facing `bin/ci` run |
| `.github/workflows/ci.yml` | the pull-request gate |
| `bin/ci` | the entry point (usually just `CI.run`) |

A gate in only one of them means "green locally, red on the PR" — or worse, the reverse: a check that
only ever runs when someone remembers to run it.

## Job layout for the workflow

```yaml
jobs:
  scan_ruby:      # brakeman + bundler-audit
  scan_js:        # npm audit --audit-level=high
  lint:           # rubocop -f github, check-ui-components, check-theme
  test:           # db:test:prepare test, on pgvector/pgvector:pg16
  system-test:    # db:test:prepare test:system, uploads tmp/screenshots on failure
```

Split by *concern*, not by speed: a security finding and a lint finding should be two red checks, not one
ambiguous one.

## Adding a gate

1. Write the script. It must be non-interactive, print `path:line: message`, and exit 0/1.
2. Add it to `config/ci.rb` in the right phase.
3. Add the same command to `.github/workflows/ci.yml` in the matching job.
4. Update the checklist in the repo's agent briefing and in this skill's table.
5. Run `bin/ci` locally before pushing.

## Removing or weakening a gate

Requires a constitution amendment: a separate commit that bumps the version, records why, and states
the test impact. A gate deleted inside a feature PR is how a repository loses its guarantees one
convenient exception at a time.

## `brakeman --exit-on-warn`

Warnings are findings. Fix them. A suppression is legal only with an inline justification:

```ruby
# brakeman:ignore [CommandInjection] Path is a constant from a whitelist, never user input.
```

The flag combination (`--exit-on-warn --exit-on-error`) is deliberate: in advisory mode nobody reads the
output, and the first real SQL injection ships unnoticed.

## The seeds trap

```ruby
# Replanting seeds in the test environment leaves their committed rows behind, and the next
# `bin/rails test` then fails fixture FK validation on the orphans.
step "Tests: Seeds", "env RAILS_ENV=test bin/rails db:seed:replant db:test:prepare"
```

Any step that writes to the test database from outside a transaction must finish by preparing it.

## Local pre-push

```bash
bin/ci
```

One command, identical to CI. A pre-push hook that runs it is worth the ninety seconds; a hook that runs
a *different* set of checks is worse than none, because it teaches the team to trust a green that means
nothing.
