---
name: Mint a Friendbuy referral link and track the conversion it produces
description: >-
  Authenticate against the Friendbuy Merchant API, sync a customer, generate that
  customer's personal referral link for a campaign, and report the purchase a
  referred friend makes so Friendbuy can attribute it and issue rewards.
api: openapi/friendbuy-referrals-api-openapi.yml
operations:
  - createAuthorization
  - postCustomer
  - postPersonalReferralLink
  - getReferralStatus
  - postPurchaseEvent
generated: '2026-08-13'
method: generated
source: openapi/ + https://developers.friendbuy.com
---

# Mint a referral link and track the conversion

Base URL: `https://mapi.fbot.me/v1`

## 1. Get a token — `createAuthorization`

`POST /authorization` with `{"key": "<account key>", "secret": "<account secret>"}`
and `Content-Type: application/json`.

The response is `{"tokenType": "Bearer", "token": "<jwt>", "expires": "<ISO 8601>"}`.
Send `Authorization: Bearer <token>` on every subsequent call.

**Cache the token until near `expires`.** Do not call `/authorization` per request —
it is the one operation that is guaranteed to cost you throughput, and 429 exists
on every endpoint with no published limit to plan against.

Credentials are issued by Friendbuy in the Developer Center of the Retailer App
(https://retailer.fbot.me). There is no self-serve key and no sandbox.

## 2. Make sure the advocate exists — `postCustomer`

`POST /postCustomer` with the customer body: `id` is **your** identifier for the
customer and is required; `email`, `firstName`, `lastName`, `isNewCustomer` are
optional. Friendbuy joins everything else in the platform on this id.

If you only need to confirm the record, `GET /getCustomer` (`getCustomer`) reads it
back. A 404 means the customer has never been synced.

## 3. Mint the link — `postPersonalReferralLink`

`POST /postPersonalReferralLink` with `{"customerId": "<your id>", "campaignId": "<uuid>"}`
plus optional `email`, `firstName`, `lastName`.

The response carries `link`, `referralCode`, `customerId` and `campaignId`.
`campaignId` comes from the campaign configured in the Retailer App — the API
cannot create campaigns.

**Minting many links: use `postPersonalReferralLinkBatch`, not a loop.**
`POST /postPersonalReferralLinkBatch` exists specifically so that generating links
for a list of customers is one request rather than N. Friendbuy publishes no rate
limit, so a loop is the fastest way to find the undocumented one.

## 4. Check a code before you trust it — `getReferralStatus`

`GET /getReferralStatus` returns `{code, status}` where `status` is `active` or
`blocked`. A `400` here means the code or the advocate is blocked, or the code is
otherwise invalid — it is a business answer, not a transport failure. Do not retry
it.

## 5. Report the conversion — `postPurchaseEvent`

`POST /postPurchaseEvent` with `id` (your order id), `amount`, `currency`, the
`customer` object, and optionally `couponCode` and `products[]`.

The response is `{"received": true, "eventId": "<uuid>"}`.

**There is no idempotency key.** If the call times out or fails ambiguously, you
cannot safely retry it: Friendbuy documents no request-replay window, so a second
send can be counted as a second purchase and can trigger a second reward. Record
the returned `eventId` against your order id and reconcile with
`getPurchasesAnalytics` before ever re-sending.

For sign-ups use `postSignUpEvent`; for anything else use `postCustomEvent` with
an `eventName` and a free-form `properties` object.

## 6. Watch for the reward

Rewards are not created through the API — they are evaluated by Friendbuy after a
conversion passes the campaign's business rules and fraud checks. You learn about
them two ways:

- **Webhooks** (preferred): `advocateReward` and `friendIncentive` deliver
  `rewardId`, `rewardAmount`, `rewardUnit`, `couponCode` and the friend/advocate
  linkage. See `asyncapi/friendbuy-webhooks.yml`.
- **Analytics reads**: `getDistributedAdvocateRewards`,
  `getDistributedFriendIncentives`, `getReferralRewards`.

## Errors

Every error is the flat envelope `{error, message, code, reference}` — not RFC 9457.

| Status | Do this |
|---|---|
| 401 | Token expired or malformed. Re-run `createAuthorization` and retry once. |
| 404 | The customer or code does not exist. Do not retry; create it first. |
| 422 | Validation failure. It will fail identically on retry — fix the field named in `message`. |
| 429 | Back off exponentially with jitter. No `Retry-After` or `RateLimit-*` header is documented. |
| 500 | Retry with backoff; quote `reference` to Friendbuy support if it persists. |
