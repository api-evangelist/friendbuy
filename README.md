# Friendbuy (friendbuy)

<!-- API-EVANGELIST-PROVENANCE:BEGIN -->
> ### About this repository
>
> **This is not our API.** This repository is an independent, third-party profile of a company's
> **publicly available** API surface, maintained by [API Evangelist](https://apievangelist.com).
> API Evangelist does not operate, host, resell, or support this company's APIs, and is not
> affiliated with or endorsed by the company unless stated on the profile.
>
> **Where the information came from.** Everything here is assembled from material a member of the
> public can reach with a browser and no credentials — the company's own website, developer portal
> and documentation, the specifications it publishes for public use (OpenAPI, AsyncAPI, JSON Schema,
> `apis.json`, `llms.txt` and similar), its public repositories, and its public status, pricing and
> changelog pages. **Nothing here is obtained by breaching a system, defeating an access control, or
> using credentials of any kind.**
>
> **The rating is an independent assessment.** The Kin Score and Agent Readiness rating are
> independently calculated scores of a company's *public* API artifacts, produced by API Evangelist
> against a published rubric. They are not certifications, endorsements, security assessments, or
> audits, and they score published artifacts — not the quality, safety, or security of the software.
>
> **Corrections, re-scores, and removal are free.** No partnership, contract, or purchase is
> required, and you do not need to justify the request.
>
> - **Something wrong?** Open an issue on this repository, or email
>   [info@apievangelist.com](mailto:info@apievangelist.com).
> - **Published something new?** Ask for a re-score and we will re-run the rating.
> - **Want the listing taken down?** Say so and we will honor it. The profile is reduced to your
>   company name, a factual description, and a link to your own site, and the company is recorded as
>   **unrated** — never scored zero for having asked.
>
> **Response times.** Acknowledgement within **one business day**; removal or restriction within
> **two business days**; corrections and re-scores within **five business days**.
>
> **Not from the company, and here with a question?** You are welcome here — we would rather be the
> front line and point you the right way than have a good report go nowhere. What this repository
> can answer is narrow, though, so it is worth knowing who you are actually looking for:
>
> - **A question about how the API works, an account, billing, or a bug in the service** — that is
>   the company's own support, not us. We profile this API; we do not operate it and cannot see
>   your account.
> - **A bug in an open-source project we only catalog** — file it on that project's own repository.
>   This has happened with a real and correct bug report that reached us instead of the people who
>   could fix it, which helped nobody.
> - **Anything about this listing itself** — the description, the tags, the rating, a missing or
>   wrong artifact — is ours. Open an issue here.
> - **Not sure, or something general about API Evangelist or APIs.io** — open an issue on the
>   [APIs.io Inbox](https://github.com/api-search/inbox) and we will route it.
>
> **This repository contains no software, and we will never ask you to download anything.** There is
> no build, release, installer, or binary here — only text and machine-readable API descriptions, so
> there is nothing here that can be "corrupt" or need "repairing". Any issue, comment, or email
> claiming otherwise and offering a download link is not from us and is hostile. Do not follow the
> link; it is a lure. Report it to GitHub and, if you like, tell us at
> [info@apievangelist.com](mailto:info@apievangelist.com) so we can take it down.
>
> **On a security or compliance team?** Email
> [info@apievangelist.com](mailto:info@apievangelist.com) with *security* in the subject line and
> you will get a person, not a form. We will tell you exactly which public URLs this profile was
> built from so your team can see the same surface we did, and we will take the listing down on
> request while you work through it.
>
> Full detail: **[Where this data comes from](https://apievangelist.com/about/where-our-data-comes-from)**
<!-- API-EVANGELIST-PROVENANCE:END -->

Friendbuy is a referral and loyalty marketing platform for ecommerce and direct-to-consumer brands. Merchants launch referral, loyalty, and reward campaigns through on-site widgets and a no-code dashboard, and integrate server-to-server through the **Friendbuy Merchant API** (base `https://mapi.fbot.me/v1`). The Merchant API lets merchants sync customer records, generate personal referral links, track purchase / sign-up / custom conversion events, pull campaign and reward analytics, and manage loyalty ledger balances, adjustments, redemptions, and coupons.

**APIs.json:** [https://raw.githubusercontent.com/api-evangelist/friendbuy/refs/heads/main/apis.yml](https://raw.githubusercontent.com/api-evangelist/friendbuy/refs/heads/main/apis.yml)

## Access Model (Honest Note)

- **Documentation is public** at [developers.friendbuy.com](https://developers.friendbuy.com); anyone can read the Merchant API reference.
- **The API itself is gated.** Friendbuy uses quote-based, contact-sales pricing (reported tiers ~ $249/mo Starter, ~ $749/mo Deluxe, custom Enterprise, keyed to monthly company sales volume). There is no self-serve sign-up that hands out API keys.
- **Authentication is a two-step flow.** Exchange an account **key** and **secret** (issued from the Friendbuy Developer Center to paying accounts) at `POST /authorization` for a short-lived Bearer JWT, then send `Authorization: Bearer <token>` on every other call.
- **No public WebSocket API.** Friendbuy's own API is request/response REST. Real-time client behavior comes from the browser `friendbuy.js` widget SDK firing tracking events, not from a WebSocket. See `review.yml`.
- **Endpoint paths are confirmed** from the public docs; **request/response body schemas in `openapi/friendbuy-openapi.yml` are honestly modeled** from documented behavior and should be reconciled against the live reference before production use.

## Tags

- Referral Marketing
- Loyalty
- Rewards
- Ecommerce
- Marketing
- Advocacy

## Timestamps

- **Created:** 2026-07-10
- **Modified:** 2026-07-10

## APIs

### Friendbuy Customers API

Create or update customer records synced from your store, retrieve a customer by id or email, look up loyalty member tier, and service GDPR/CCPA data-access and erasure requests via `getUserData` and `deleteUserData`.

- **Human URL:** [https://developers.friendbuy.com](https://developers.friendbuy.com)
- **Base URL:** `https://mapi.fbot.me/v1`

### Friendbuy Referrals API

Generate unique personal referral links and codes for advocate customers - singly or in batch - and check the status of referrals tied to a customer or referral code.

- **Human URL:** [https://developers.friendbuy.com](https://developers.friendbuy.com)
- **Base URL:** `https://mapi.fbot.me/v1`

### Friendbuy Events API

Track conversion events server-to-server - purchase/order events, account sign-ups, and arbitrary custom events - so Friendbuy can attribute them to referrals and trigger advocate and friend rewards.

- **Human URL:** [https://developers.friendbuy.com](https://developers.friendbuy.com)
- **Base URL:** `https://mapi.fbot.me/v1`

### Friendbuy Analytics API

Pull date-ranged, paginated campaign analytics - widget views, shares, clicks, account sign-ups, purchases, distributed advocate rewards and friend incentives, combined referral rewards, email captures, and email metrics.

- **Human URL:** [https://developers.friendbuy.com](https://developers.friendbuy.com)
- **Base URL:** `https://mapi.fbot.me/v1`

### Friendbuy Rewards & Loyalty API

Manage the loyalty program - read individual and all-customer ledger balances, post point/credit adjustments (by customer id or custom identifier), list redemption options, redeem rewards, and list issued coupons.

- **Human URL:** [https://developers.friendbuy.com](https://developers.friendbuy.com)
- **Base URL:** `https://mapi.fbot.me/v1`

## Common Properties

- [LinkedIn](https://www.linkedin.com/company/friendbuy)
- [Website](https://friendbuy.com)
- [Documentation](https://developers.friendbuy.com)
- [Plans](plans/friendbuy-plans-pricing.yml)
- [Rate Limits](rate-limits/friendbuy-rate-limits.yml)
- [Fin Ops](finops/friendbuy-finops.yml)
- [Blog](https://friendbuy.com/blog)

## Maintainers

**FN:** Kin Lane
**Email:** kin@apievangelist.com
