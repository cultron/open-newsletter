# Reference products

We based the MVP feature list on what six existing products ship. Every row below comes from the product's own site or repo, read on 2026-11-17. Where a page did not state something, the cell says "not found" and we did not fill the gap from memory.

## Hosted products

| Feature | Beehiiv | Kit | Substack |
|---|---|---|---|
| Embeddable signup form | Inline, popup, slide and sticky layouts on all plans ([source](https://www.beehiiv.com/support/article/12977090590487-creating-an-embedded-subscribe-form)) | Inline, pop-up and slide-in forms ([source](https://kit.com/features/forms)) | not checked |
| Double opt-in | "Require double opt-in" is a per-form setting (same source) | "Block spammers and bots with double opt-in on all forms and landing pages." (same source) | not checked |
| Segments and tags | Custom fields on all plans. Segments: 3 on Free, 10 on Lite, unlimited on Pro ([source](https://www.beehiiv.com/pricing)) | Tagging and segmentation on all plans ([source](https://kit.com/pricing)) | not checked |
| CSV export | Subscriber export from Settings ([source](https://www.beehiiv.com/support/article/12258595483543-exporting-post-content-or-subscriber-data-from-beehiiv)) | not found | not checked |
| Web archive | Website builder on all plans (pricing page) | "Newsletter site" on all plans ([source](https://kit.com/features)) | Posts can be free or "only for paid subscribers" ([source](https://substack.com/going-paid)) |
| RSS to email | "RSS to Send", Pro plan only (pricing page) | RSS campaigns on Creator and Pro: "automatically create and send emails when you publish new blog content and your RSS feed updates" (pricing page) | not found |
| Welcome email | All plans (pricing page) | Sequences on Creator and Pro (pricing page) | not checked |
| Analytics | Campaign analytics on all plans (pricing page) | Per-email "opens, clicks, and sales reported" ([source](https://kit.com/features)) | not checked |
| Referral program | Lite and above ([source](https://www.beehiiv.com/features/referral-program)) | Pro (pricing page) | not found |
| Recommendations | Free recommendations on all plans ([source](https://www.beehiiv.com/features/recommendations)) | Free recommendations on all plans (pricing page) | not found |
| Paid subscriptions | "beehiiv takes 0% of your paid subscription revenue." (pricing page) | Paid newsletter on all plans ([source](https://kit.com/features)) | "Writers keep 90% of their revenue minus credit card fees." ([source](https://substack.com/going-paid)) |
| API and webhooks | API on Free without the Send API. Webhooks: 3 on Lite, unlimited on Pro (pricing page) | "API documentation" on all plans (pricing page). Webhooks: not found | not found |

Kit's help center describes two RSS modes. In digest mode it compiles several new posts into one email and you pick the days and time Kit drafts the email. In single mode it sends one email per post. This comes from a search result for [Kit's RSS help article](https://help.kit.com/en/articles/2502636-how-to-connect-your-rss-feed-to-kit). We did not open the article itself.

## Open-source products

| Feature | Listmonk | Keila | Ghost |
|---|---|---|---|
| License | AGPL v3 ([source](https://github.com/knadh/listmonk)) | AGPL v3 or later ([source](https://github.com/pentacent/keila)) | MIT ([source](https://github.com/TryGhost/Ghost)) |
| Stack | Go backend, Vue frontend, Postgres ([source](https://listmonk.app/docs/)) | Elixir ([source](https://www.keila.io/docs)) | Node.js (repo) |
| Opt-in | "single and double opt-in lists" ([source](https://listmonk.app/)) | "Double-Opt-In (Confirmed Opt-In)" ([source](https://www.keila.io/)) | Native signup forms. Double opt-in: not found ([source](https://ghost.org/features/)) |
| Segments | "Query and segment subscribers with SQL expressions." (homepage) | Visual segment editor and a segment language (homepage) | Member tagging on all hosted plans ([source](https://ghost.org/pricing/)) |
| CSV | not checked | "CSV import & export" (homepage) | Member import and export on all hosted plans (pricing page) |
| Editor | Visual builder, WYSIWYG, Markdown, raw HTML, plain text (homepage) | Block editor, Markdown, MJML, plain text (homepage) | Its own editor (features page) |
| Public archive | Yes, with an archive RSS feed at `/archive.xml` ([source](https://listmonk.app/docs/archives/)) | Public archive links (docs) | It is a publishing platform, with SEO tools on all hosted plans (pricing page) |
| RSS to email | not found | not found | not found |
| Analytics | "campaign performance, bounces, top links" (homepage) | "click & open tracking" and "Automatic bounce and complaint handling" (homepage) | Open and click rates on all hosted plans (pricing page) |
| Paid subscriptions | none listed | none listed | Yes, "without any additional transaction fees from Ghost" (pricing page) |
| API | "Extensive API coverage for all features." (homepage) | API for contacts and campaigns (homepage) | Content API on all plans. Admin API and webhooks on Publisher and above (pricing page) |

## What this means for the MVP

- **Every product ships signup forms and some form of opt-in control.** Double opt-in is in the MVP and on by default.
- **Tags, segments and CSV are standard.** Lists, tags, import and export are in the MVP. Saved segments wait for v1.
- **A public archive is standard.** The MVP has public post pages with SEO metadata and its own RSS feed.
- **RSS to email is a paid-plan feature on Beehiiv and Kit, and we found it in none of the three open-source products.** It is in the MVP because it is the project owner's own use case: `mattcullerton.com/rss.xml` becomes a weekly email.
- **Open, click, bounce and complaint tracking is standard.** The MVP gets it from Resend webhooks.
- **Referrals, recommendations and paid subscriptions sit on higher plans or in the hosted products only.** They are v1 issues.
