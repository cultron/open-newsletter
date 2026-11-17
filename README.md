# open-newsletter

An open-source newsletter service you run on your own accounts. It covers signup forms, double opt-in, a post editor, a public archive, sends through Resend, and an RSS digest that turns a site's `rss.xml` into a weekly email.

**Status: spec only.** There is no application code in this repo yet. The work is written up as [GitHub issues](../../issues), grouped into the [MVP](../../milestone/1) and [v1](../../milestone/2) milestones.

## Why build this

**You own the list.** Subscribers live in your own Postgres database. You can export them as CSV at any time, and no vendor sits between you and the table.

**No platform fee on top of your costs.** Hosted newsletter products charge by subscriber count, by revenue, or both. These figures come from each vendor's own pricing page, read on 2026-11-17:

| Product | What the pricing page says |
|---|---|
| Substack | "Writers keep 90% of their revenue minus credit card fees." ([source](https://substack.com/going-paid)) |
| Beehiiv | Free up to 2,500 subscribers. "Paid plans start at $59/month." It takes 0% of paid subscription revenue. ([source](https://www.beehiiv.com/pricing)) |
| Kit | "Free - up to 10,000 subscribers." The Creator plan is $33 a month for 1,000 subscribers, billed yearly. ([source](https://kit.com/pricing)) |

Running this project still costs money. You pay Resend for sending, Vercel for the site, and a host for the backend. Per [Resend's pricing page](https://resend.com/pricing) on 2026-11-17, its free marketing plan covers 1,000 contacts and its free transactional plan covers 3,000 emails a month with a limit of 100 a day.

**Open-source options exist, on other stacks.** [Listmonk](https://listmonk.app/) is written in Go and [Keila](https://www.keila.io/) in Elixir, both under AGPL v3. [Ghost](https://ghost.org/) is a full publishing platform on Node.js under MIT. This project is MIT licensed and built as a Next.js app, so a team that already ships Next.js on Vercel can read all of it and change it. The feature comparison is in [docs/reference-products.md](docs/reference-products.md).

**It is also a worked example of Apso.** Apso generates the backend from a schema file. [docs/build-log.md](docs/build-log.md) records where Apso, Resend and the UI SDK did the work and what stayed hand-written.

## Inspiration: SignuDoc

The model for this project is SignuDoc, the e-signature app Kevin Rose announced on November 16, 2026. His post, as quoted by [Protos](https://protos.com/digg-founder-kevin-rose-just-cloned-docusign-with-ai/), reads: "So I built SignuDoc: open source, one-click deploy to your own @vercel account, $0 per seat." The [SignuDoc repo](https://github.com/NYLLON-SOFTWARE/SignuDoc) is MIT licensed. Its README lists Next.js 16, Neon Postgres, Vercel Blob, Resend, and a daily Vercel cron job.

We borrow three things from it: the code is open, it deploys to accounts you own, and Resend carries the email.

What we could not confirm:

- We could not open the original X post. The quote above comes from Protos.
- We found no source that describes a newsletter or signup system built by Kevin Rose. SignuDoc signs documents. On 2026-11-17 the feed at `kevinrose.com/feed` reported `Ghost 6.78` as its generator, so his own newsletter appears to run on Ghost.

## Stack

| Layer | Tool | Notes |
|---|---|---|
| Site and admin | [Next.js](https://nextjs.org/) app router | Public pages, admin, route handlers for forms, webhooks and scheduled jobs |
| UI behaviors | [UI SDK](https://ui-sdk.dev/docs) (`@ui-sdk/react`, `@ui-sdk/core`) | Unstyled components and behaviors from Vercel Labs. Its README says it is in beta. Its component list has no table or data grid, so the subscriber table is hand-built |
| Email | [Resend](https://resend.com/docs) | Transactional API for confirmation and test emails. Broadcasts for newsletter sends. Webhooks for delivery, open, click, bounce and complaint events |
| Backend | [Apso](https://docs.apso.ai) | Generates a NestJS and TypeORM service on Postgres from a `.apsorc` schema: entities, REST endpoints, DTO validation and OpenAPI docs |
| Admin auth | [Better Auth](https://www.better-auth.com/) with `@apso/better-auth-adapter` | Stores users and sessions through the Apso API |
| Hosting | Vercel for the Next.js app, Apso Cloud or any Docker host for the backend | The generated backend is a standard NestJS project |

## Architecture

```
 Reader's browser                      Admin's browser
        |                                     |
        v                                     v
+---------------------------------------------------------------+
|  Next.js app on Vercel                                        |
|  Public: /subscribe, /confirm, /preferences, /p/[slug],       |
|          /rss.xml, embed script                               |
|  Admin:  /admin (posts, subscribers, sends, analytics)        |
|  Server: route handlers for form posts, Resend webhooks       |
|          and scheduled jobs (hand-written)                    |
+------------------+----------------------------+---------------+
                   | @apso/sdk (server only)    | Resend SDK
                   v                            v
+------------------------------+   +----------------------------+
|  Apso service (generated)    |   |  Resend                    |
|  REST CRUD, OpenAPI docs     |   |  Transactional emails      |
|  Postgres: subscribers,      |   |  Broadcasts and scheduling |
|  lists, posts, sends, events |   |  Contacts, segments, topics|
+------------------------------+   +-------------+--------------+
                   ^                             |
                   |   webhooks: delivered, opened, clicked,
                   +---- bounced, complained (via the Next.js app)
```

The draft schema is in [docs/apsorc.draft.json](docs/apsorc.draft.json). It has 15 entities:

| Entity | Holds |
|---|---|
| `Publication` | One row of settings: name, from address, reply-to, physical mailing address, double opt-in switch |
| `Subscriber` | Email, name, status, confirm and manage tokens, consent record, Resend contact id |
| `List`, `Subscription` | Lists a reader can join, and one row per reader per list |
| `Tag` | Admin-only labels on subscribers |
| `Post` | Markdown body, rendered HTML, status, publish date, SEO fields |
| `Send` | One email send of a post or digest: subject, schedule, Resend broadcast id, cached counts |
| `EmailEvent` | One row per Resend webhook event, keyed by the webhook id so replays do not double count |
| `Feed`, `FeedItem` | RSS sources for the digest and the items already seen |
| `ImportJob` | A record of each CSV import, with the consent note the admin entered |
| `User`, `Session`, `Account`, `VerificationToken` | The tables Better Auth needs |

## Assumptions

We picked each of these as a default without asking the project owner. An issue can overturn any of them.

1. **One publication per deployment.** Multiple publications are a v1 issue.
2. **The Apso database is the system of record.** Resend holds a copy of the contacts for sending. Unsubscribes and bounces flow back through webhooks.
3. **Newsletter sends use Resend Broadcasts.** One API call hands Resend the fan-out, the schedule and the per-recipient unsubscribe link. A direct driver on the batch API is a v1 issue.
4. **Hand-written server logic lives in the Next.js app.** The Apso service stays fully generated, so it can be regenerated when the schema changes. If a job needs to run next to the database, Apso's `src/extensions/` folder is the fallback.
5. **Scheduled work runs on Vercel's scheduler.** Apso's docs describe no cron or queue feature.
6. **The editor is Markdown in the MVP.** A block editor can come later.
7. **Integer primary keys, with random tokens in public URLs.** The draft passed Apso's local smoke test with integer keys. With `uuid` keys it failed on the local PGlite database under CLI 0.47.1, per the build log.
8. **Open and click tracking is opt-in.** Resend's docs say tracking is off by default for every domain.

## MVP scope

In the MVP (29 issues):

- Signup: hosted subscribe page (#7), embeddable form (#8), double opt-in (#9)
- Subscribers: admin table (#10), CSV import (#11) and export (#12), lists and tags (#13), sync to Resend (#14)
- Publishing: Markdown editor (#15), email template with CAN-SPAM footer (#16), public archive with SEO (#17)
- Sending: test sends (#18), send and schedule through Broadcasts (#19), RSS-to-email digest (#24)
- Tracking: webhook receiver (#20), bounce and complaint handling (#21), dashboard and analytics (#25)
- Compliance: one-click unsubscribe (#22), preference center (#23), domain authentication guide (#26)
- Platform: Apso schema and backend (#1, #2), Next.js scaffold (#3), admin sign-in (#4), settings (#5), API (#27), outbound webhooks (#28), deploy guide (#29)
- One spike before the send work starts: confirm how Resend Broadcasts handle unsubscribe headers and events (#6)

In v1 (8 issues): saved segments (#30), welcome email (#31), referral program (#32), recommendations (#33), paid subscriptions (#34), one-click deploy to Vercel (#35), a direct send driver (#36) and multiple publications (#37).

Out of scope for now: an ad network, A/B subject tests, a drag-and-drop email builder, and SMS.

## How to read the issues

Every issue has a user story, acceptance criteria, and a "Who does the work" section that separates what Apso generates, what Resend or the UI SDK provides, and what is hand-written.

Sizes are estimates for one developer working with a coding agent. Nothing here has been built, so they are not measurements.

| Size | Meaning |
|---|---|
| S | Under one day |
| M | One to three days |
| L | More than three days |

## Docs

- [docs/build-log.md](docs/build-log.md): where Apso, Resend and the UI SDK did the work, and what was left
- [docs/reference-products.md](docs/reference-products.md): what Beehiiv, Kit, Substack, Listmonk, Keila and Ghost ship, with sources
- [docs/apsorc.draft.json](docs/apsorc.draft.json): the draft Apso schema

## License

[MIT](LICENSE)
