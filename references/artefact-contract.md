# What each artefact must contain

## `spec.md`

```markdown
# Feature 011 — Notification digest

## User stories
- **US1 (P1)** As a requester I get one daily summary instead of one e-mail per reply.
- **US2 (P2)** As a supervisor I can turn the digest off per workspace.

## Functional requirements
- **FR-001** A digest covers notifications created since the previous digest, per user.
- **FR-002** A user with no new notifications receives no e-mail.
- **FR-003** A digest respects the same visibility rules as the in-app inbox.
- **FR-004** Turning the digest off stops the e-mail without deleting the rows.

## Success criteria
- **SC-001** A user with 5 new notifications receives exactly 1 e-mail.
- **SC-002** No digest contains an internal note body (asserted on the rendered mail).

## Out of scope
- Per-notification muting.
- A digest for administrators.
```

Rules: a requirement is one sentence, testable, and numbered. No implementation nouns (`Resque`, `props`,
`index`) — those belong in the plan. A requirement that needs two tests is two requirements.

## `plan.md`

```markdown
## Constitution check
| Principle | How this plan complies |
|---|---|
| I Server-driven UI | Digest page is an Inertia page; the mailer reuses `Tickets::Payload` |
| II Skinny controllers | one `DigestsController#show` delegating to `Digests::ForUser` |
| III Test-first | service test + mailer test + one integration test land in the same commit |
| IV Gates | no gate weakened; seeds unchanged |
| V Simplicity | no gem; `DeliverDigestJob` on the existing Solid Queue |

## Phases
1. Data: `digest_settings` on `helpdesk_settings` (boolean, default on).
2. Query: `Digests::ForUser` returning the scoped notification list.
3. Job + mailer, scheduled daily.
4. Page + prop contract.
5. Tests, in the same commits as 2–4.
```

## `research.md`

One entry per decision that could have gone either way:

```markdown
## D3 — Where the "since last digest" boundary lives
Decision: a `last_digested_at` column on the user row.
Alternatives: a separate `digest_runs` table (rejected: one row per user per day, unread); a Redis key
(rejected: the app has no Redis, and Solid Cache is not authoritative).
Impact: one migration, one index; no change to the notification ledger.
```

A decision without the rejected alternative is a note, not research.

## `data-model.md`

```markdown
### notifications
| column | type | notes |
|---|---|---|
| workspace_id | bigint | not null, FK |
| user_id | bigint | not null, FK, indexed with created_at |
| ticket_id | bigint | FK |
| kind | string | ticket_created, ticket_reply, information_requested, ticket_resolved, ticket_closed |
| read_at | datetime | null = unread |

### Authorization scopes
- `visible_to(user)` — a requester sees their own tickets; an agent sees their teams' + assigned.
- `manageable_by(user)` — staff only, same relation.
- A notification belongs to exactly one user; there is no cross-user relation.
```

## `contracts/`

One file per surface, each naming the requirement it implements:

```markdown
# routes.md
| Method | Path | Controller#action | Auth | Requirement |
|---|---|---|---|---|
| GET | /digests/latest | digests#show | signed-in | FR-001 |
| PATCH | /admin/settings | admin/settings#update | administrator | FR-004 |
```

```markdown
# authorization.md
| Actor | May read | May manage | Denial |
|---|---|---|---|
| requester | own tickets and own notifications | own comments | 404 on a foreign id |
| agent | teams' + assigned, not restricted | same | 404 / 403 for a closed action |
```

## `tasks.md`

```markdown
- [x] T001 migration + model column, `test/models/helpdesk_setting_test.rb` (FR-004)
- [ ] T002 `app/queries/digests/for_user.rb`, `test/queries/digests/for_user_test.rb` (FR-001, FR-003)
- [ ] T003 `app/jobs/deliver_digest_job.rb`, `test/jobs/deliver_digest_job_test.rb` (FR-002)
- [ ] T004 `app/views/digest_mailer/daily.*` + `test/mailers/digest_mailer_test.rb` (SC-002)
- [ ] T005 page + prop contract + `test/controllers/digests_controller_test.rb` (FR-001)
```

Every task names a file and a test. A task with no test is either a refactor (say so) or unfinished
thinking.

## `quickstart.md`

```markdown
## See it working
1. `bin/rails db:migrate db:seed`
2. Sign in as `mahasiswa@example.test`
3. `bin/rails "digest:send[1]"` — one e-mail lands in `tmp/mails`
4. `open tmp/mails/*` — one digest, no internal note body
```

If the feature cannot be demonstrated in four steps, the feature has no observable outcome yet.
