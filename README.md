# Friendbuy (friendbuy)

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
