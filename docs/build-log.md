# Build log: where the tools did the work

This log records each place Apso, Resend or the UI SDK saved work on open-newsletter, what the tool produced, and what was left to write by hand.

Two rules apply to every entry. An entry marked **Observed** is something we ran and saw. An entry marked **Docs state** repeats what the vendor's documentation says, and we have not tested it yet. The log holds no time-saved estimates, because we timed nothing against a hand-built version.

## Session 1: spec and draft schema (2026-11-17)

No application code was written in this session. We wrote the spec, drafted the Apso schema, and ran the Apso CLI against the draft in a scratch folder to see what it generates. We deployed nothing, sent no email and created no account.

Environment: `@apso/cli` 0.47.1 on macOS, installed with Homebrew. The CLI reported that 0.50.1 was available, so some of the rough edges below may already be fixed.

### Apso

**Observed: the schema file is the whole backend spec.** [apsorc.draft.json](apsorc.draft.json) declares 15 entities, 106 fields and 12 relationships. That covers publication settings, subscribers, lists, tags, posts, sends, email events, RSS feeds, import jobs and the four auth tables.

**Observed: `apso schema validate` and `apso schema lint` reviewed the draft before any code existed.** The first draft passed validation with 10 warnings, per the CLI output:

- Seven foreign keys had no index. The CLI named each one and gave the fix: set `"index": true` on the relationship.
- Two entities did not track `updated_at`.
- One entity, `User`, maps to a reserved word in Postgres.

We fixed the seven index warnings and one timestamp warning by editing the JSON. Two warnings remain by choice: `EmailEvent` rows never change, and `User` is the name the Better Auth adapter expects.

**Observed: `apso generate` wrote 107 TypeScript files, 9,608 lines, under `src/autogen/`.** Each entity got an entity class, a module, a service, a controller, DTOs, and spec files for the service and the controller. The CLI log reported the build of all entities at 118.80 ms on one run and 130.81 ms on another.

**Observed: `apso test smoke` passed 62 checks on the generated service.** It started the service on a local PGlite database, called the list endpoint for all 15 entities, then created, read and deleted one row per entity. It needed no Docker and no Apso account, because we created the project with `apso init --skip-platform`.

**Observed: the template serves OpenAPI docs.** The generated `src/config/nest-app.config.ts` mounts Swagger at `/_docs`.

**Docs state, not yet tested:**

- Each entity gets `GET`, `POST`, `PATCH`, `PUT` and `DELETE` routes plus a bulk create route, with filtering, sorting, pagination, field selection and joins on the list route. Source: `develop/api-reference` in the Apso docs at https://docs.apso.ai.
- `apso migrate` writes the migration for a schema change and tests it on a local copy. Source: the CLI's own help text.
- `apso deploy` pushes the service to Apso Cloud. The docs name Lambda, API Gateway and a Postgres database as the resources behind a service.
- `@apso/sdk` gives a Next.js server a client for the generated API. The docs example declares the TypeScript interface for each entity by hand, so types are not generated from the schema.
- `@apso/better-auth-adapter` lets Better Auth store users and sessions through the generated API.

**Left to write by hand, per the Apso docs:**

- Scheduled jobs and queues. The docs describe no cron or queue feature and say to implement workers in `extensions/`.
- Inbound webhook receivers. The docs list "Webhook handlers and background job entry points" as extension work.
- Outbound webhooks. The API reference says "the CRUD API does not emit webhooks". Domain events are a separate opt-in library.
- CSV import and export, email sending and file uploads. We searched the docs for each and found no feature.
- Every rule that spans entities: confirming a subscriber, suppressing a bounced address, counting opens on a send.

**Rough edges observed:**

- With `"primaryKeyType": "uuid"` on the four auth entities, the smoke test failed on PGlite with Postgres error 42883 on `uuid_generate_v4()`. With the default integer keys it passed. The draft uses integer keys for now. Issue #1 retests this on the current CLI.
- `apso generate` printed `Skipped formatting ([object Object])`. The generated code still compiled and passed the smoke test.

### Resend

Nothing was sent in this session. Every entry here is **Docs state**, from https://resend.com/docs read on 2026-11-17.

**What Resend provides:**

- A newsletter send is one API call. `POST /broadcasts` takes a `segment_id`, the content, and an optional `scheduled_at`. Resend handles the fan-out to each recipient.
- A per-recipient unsubscribe link, through the `{{{RESEND_UNSUBSCRIBE_URL}}}` placeholder, and a hosted preference page where a reader can opt out per topic or from everything.
- Scheduling for single emails up to 30 days ahead, and a batch endpoint that accepts up to 100 emails per call.
- Webhooks for `email.delivered`, `email.opened`, `email.clicked`, `email.bounced`, `email.complained` and more, signed with Svix headers. The payloads carry a `broadcast_id` and, for clicks, the link.
- A CSV contact import endpoint, `POST /contacts/imports`.
- The `react` parameter in the Node.js SDK, which accepts a React Email component as the message body.

**Left to write by hand:**

- Keeping Resend's contacts, segments and topics in step with the Apso database.
- `List-Unsubscribe` and `List-Unsubscribe-Post` headers on emails sent through the transactional API. Resend's guide says to add them yourself.
- A check of what Broadcasts do with those headers. We found no page that says Resend adds them to Broadcasts. Issue #6 tests it.
- Turning tracking on. The docs say open and click tracking is "disabled by default for all domains".
- The webhook receiver, signature check and event storage.

**One correction the docs forced:** the brief for this project said "audiences". Resend's docs say the Audiences API "is now deprecated and will be removed in the future" and that audiences are now called segments. The spec uses contacts, segments and topics.

### UI SDK

Nothing was built with it in this session. Every entry is **Docs state**, from https://ui-sdk.dev/docs and the README at https://github.com/vercel-labs/ui read on 2026-11-17. On that date npm listed `@ui-sdk/react` and `@ui-sdk/core` at version 0.1.0-beta.6, per `npm view`.

**What it provides:** unstyled React components and behaviors that the admin needs, including Dialog, Menu, Select, Combobox, Tabs, Toast, Pagination, Date Picker, File Input and Field with validation errors.

**Left to write by hand:**

- All styling. The docs describe the components as "Unstyled React components".
- The subscriber table. The component list has no table or data grid.
- Charts for the analytics page.

**Risk to track:** the README says "UI SDK is in beta" and that its APIs may change before general availability.

### Vercel

**Docs state:** scheduled jobs are entries in `vercel.json` that call a route on a cron expression, and a `CRON_SECRET` environment variable protects the route. The docs call Cron Jobs a legacy feature that is moving to Schedules. Source: https://vercel.com/docs/schedules/cron-jobs/quickstart.

**Left to write by hand:** the route that each schedule calls.

## Planned split of the MVP

This table is the plan, not a result. Later sessions replace each row with what happened.

| Area | Apso generates | Resend or UI SDK provides | Hand-written |
|---|---|---|---|
| Data model and CRUD API | Entities, REST routes, DTO validation, OpenAPI docs, migrations | | Schema file |
| Signup and double opt-in | `Subscriber` and `Subscription` storage | Confirmation email delivery, form fields | Form handler, token check, pages |
| Subscriber admin | List route with filters, sorting and pagination | Menus, dialogs, pagination behavior | Table, CSV import and export |
| Post editor and archive | `Post` storage | Tabs, dialogs, field validation | Markdown rendering, public pages, SEO |
| Sends | `Send` storage | Broadcast fan-out and scheduling, unsubscribe link | Send flow, contact sync |
| Tracking | `EmailEvent` storage | Webhook delivery and signatures | Receiver, counters, bounce rules |
| RSS digest | `Feed` and `FeedItem` storage | Broadcast delivery | Feed fetch, digest builder, schedule route |
| Admin auth | Auth tables and routes | | Better Auth config, sign-in page |

## How to add an entry

Add a section per working session. For each tool, write what it produced, what you still wrote by hand, and whether you observed it or read it in the docs. Record a number only when a command output or a document gives it, and name that source in the same sentence.
