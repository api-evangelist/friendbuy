---
name: Receive and verify Friendbuy webhooks and callbacks
description: >-
  Stand up an endpoint that accepts Friendbuy's outbound reward, email, receipt,
  ledger and customer-update webhooks, verify the HMAC-SHA256 signature, handle
  the array envelope and the 24-hour retry window, and answer the two
  synchronous callbacks whose HTTP status code is the business decision.
api: asyncapi/friendbuy-webhooks.yml
operations:
  - advocateReward
  - friendIncentive
  - loyaltyReward
  - emailCapture
  - emailOptOut
  - receipt
  - ledgerTransaction
  - customerUpdate
  - emailAuth
generated: '2026-08-13'
method: generated
source: https://developers.friendbuy.com
---

# Receive and verify Friendbuy webhooks

Webhooks are configured per account on the **Webhooks & Callbacks** tab of the
Developer Center in the Retailer App (https://retailer.fbot.me). Friendbuy POSTs
JSON to the URL you register. The Reward Webhook is the documented *preferred*
mechanism for depositing credit or points into a customer's account in your own
system — it is not an optional analytics feed.

## 1. Verify the signature — do this before parsing

Every request carries:

```
X-Friendbuy-Hmac-SHA256: <base64(HMAC-SHA256(raw_request_body, your_friendbuy_secret))>
```

Compute the HMAC over the **raw body bytes**, before any JSON parsing or
re-serialisation, and compare in constant time. If it does not match, reject the
request. Do not trust any field in the payload until this passes.

## 2. Handle the envelope — `data` is an array

```json
{
  "id": "255e4e45-446d-499d-a680-44ab1eca73fb",
  "type": "advocateReward",
  "createdOn": "2019-11-05T01:07:38.509Z",
  "data": [ { ... }, { ... } ]
}
```

- `id` — the id of the webhook **call**, for troubleshooting. Not an event id.
- `type` — the discriminator. Branch on it.
- `data` — **always iterate**. Friendbuy batches events that occur in quick
  succession into a single call. Reading `data[0]` and returning silently drops
  real rewards.

## 3. Answer fast, and answer 200

- Timeout is **10 seconds**. Friendbuy aborts the request after that; the Retailer
  App test tool shows `504`.
- Any non-200 is retried **every 15 minutes for up to 24 hours**.
- Therefore your handler **must be idempotent**. Enqueue and acknowledge; do the
  work asynchronously. Deduplicate on the payload's own identifiers — `rewardId`,
  `eventId`, `transactionId`, `receiptId` — not on the envelope `id`.

Use the **Test Endpoint** button on the Webhooks & Callbacks tab to fire a test
request and read back the status your endpoint returned.

## 4. The event types

| `type` | Fires when | Key fields |
|---|---|---|
| `advocateReward` | A referred friend converts and the advocate earns a reward | `rewardId`, `rewardType`, `rewardUnit`, `rewardAmount`, `rewardTrigger`, `couponCode`, `customerId`, `friends[]` |
| `friendIncentive` | The referred friend earns their incentive | same shape, plus `advocateEmailAddress`, `advocateCustomerId` |
| `loyaltyReward` | A loyalty customer completes an earning event | `rewardId`, `rewardType: ledger`, `rewardUnit`, `rewardAmount` |
| `emailCapture` | An email is captured through a share or widget flow | `eventId`, `emailAddress`, `campaign{id,name}`, `referral{channel,code}`, `advocate{email,name}`, `attributionId`, `incentive` |
| `emailOptOut` | A friend clicks unsubscribe in a share email | `emailAddress`, `campaignId`, `campaignName` |
| `receipt` | A submitted receipt is processed | `receiptId`, `status`, `storeName`, `purchaseDate`, `subtotal`, `total`, `currency`, `products[]` |
| `ledgerTransaction` | A ledger balance is credited or debited | `transactionId`, `ledgerCurrency`, `value`, `ledgerBalance`, `sourceName`, `note` |
| `customerUpdate` | Loyalty opt-in or member tier changes | `loyaltyDetails{loyaltyOptInStatus,optedInOn,everOptedIn}`, `memberTierDetails{...}` |

The three reward types share one payload shape and can be pointed at one endpoint —
branch on `type` — or at three separate URLs, whichever you prefer.

Mirror `emailOptOut` into your own suppression list: Friendbuy stops sending share
emails to that address across all your campaigns, and your own systems should
respect the same signal.

## 5. The two synchronous callbacks — your status code is the answer

### Reward Validation Callback

Friendbuy POSTs the conversion detail (advocate data, purchase date, order id) to
the Validation URL configured under Reward Criteria, and reads your **HTTP status
code** as the decision:

- `200` → validate and fulfil the reward.
- `400` → invalidate; the reward is not fulfilled.
- anything else → treated as an error and retried **every 15 minutes for up to 72
  hours**, after which the reward is rejected and flagged as an error.

Use it to veto rewards for returns, cancellations, or your own fraud rules. Never
return 500 on an unknown customer — return 400 to reject, or 200 to allow.

### Email Recipient Authorization Callback

Fired after an outbound share email has already passed Friendbuy's own opt-out
checks. Payload:

```json
{ "id": "...", "type": "emailAuth", "data": ["allowed.friend@example.com", "blocked.friend@example.com"], "createdOn": "..." }
```

`data` is an array of **email address strings**, not objects. Respond with the
addresses you permit, applying your own suppression rules on top of Friendbuy's.
