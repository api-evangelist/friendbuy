---
name: Read, adjust and redeem a Friendbuy loyalty ledger
description: >-
  Read a customer's loyalty balance, credit or debit it, list the redemption
  options configured for the account, redeem points for a reward, and reconcile
  the resulting coupon — using the Friendbuy Merchant API rewards and loyalty
  operations.
api: openapi/friendbuy-rewards-loyalty-api-openapi.yml
operations:
  - createAuthorization
  - getLedgerHeads
  - getLedgerBalance
  - getLedgerBalanceCustom
  - postLedgerAdjustment
  - postLedgerAdjustmentCustom
  - getRedemptionOptions
  - redeemReward
  - getCoupons
  - getMemberTierCustomer
generated: '2026-08-13'
method: generated
source: openapi/ + https://developers.friendbuy.com
---

# Read, adjust and redeem a loyalty ledger

Base URL: `https://mapi.fbot.me/v1`. Authenticate first with `createAuthorization`
(`POST /authorization`) and send `Authorization: Bearer <token>`; cache the token
until near its `expires` timestamp.

A ledger is keyed on **(customerId, currency)**. `currency` may be a real currency
such as `USD` or a points currency such as `Points`, depending on how the loyalty
program is configured in the Retailer App.

## 1. Read the balance — `getLedgerBalance`

`GET /ledger-balance?customerId=<your id>&currency=USD` returns
`{customerId, balance, currency}`.

**A 404 is not an error here.** The documentation states it "takes place when a
user does not have a ledger" — the customer has simply never earned loyalty
currency. Treat it as a balance of zero and continue. Failing the whole flow on
this 404 is the most common integration mistake against this API.

If your join key is an email or another alternate identifier rather than the
Friendbuy `customerId`, use `getLedgerBalanceCustom`
(`GET /ledger-balance-custom?customCustomerId=<value>&currency=<currency>`).

For the account-level picture across currencies, `getLedgerHeads`
(`GET /analytics/loyalty/ledger-heads?currency=<currency>`) returns the head
balances.

## 2. Credit or debit — `postLedgerAdjustment`

`POST /postLedgerAdjustment` with `{"customerId": "<id>", "amount": <number>, "reason": "<string>"}`.
`amount` is **positive to credit, negative to debit**. Use
`postLedgerAdjustmentCustom` when addressing the customer by a custom identifier.

**This operation is not idempotent.** Friendbuy documents no idempotency key and
no replay window. If the request times out you do not know whether the ledger
moved. Do not retry blind:

1. Re-read the balance with `getLedgerBalance`.
2. Compare it against the balance you observed before the write.
3. Only re-send if the balance is unchanged.

The authoritative confirmation is the `ledgerTransaction` webhook, which carries
`transactionId`, `value`, the resulting `ledgerBalance`, `sourceName` and `note`.
If you have that webhook configured, key your reconciliation on `transactionId`.
Remember the webhook `data` field is an **array** — several transactions can
arrive in one call.

## 3. List what points can buy — `getRedemptionOptions`

`GET /reward/redemption-options` returns the redemption options configured for the
account. Each carries the `redemptionOptionId` you need in the next step. Never
hard-code a redemption option id; merchants change them in the Retailer App.

## 4. Redeem — `redeemReward`

`POST /reward/redeem` with `{"customerId": "<id>", "redemptionOptionId": "<id>"}`.

Same non-idempotency warning applies, and it is more expensive here: a duplicate
redemption both debits the ledger twice and issues two coupons. Confirm through
`getLedgerBalance` and `getCoupons` before re-sending.

## 5. Reconcile the coupon — `getCoupons`

`GET /reward/coupons?customerId=<id>` lists the coupons issued to the customer.
This is how you verify a redemption actually produced something the customer can
use.

## 6. Tiers — `getMemberTierCustomer`

`GET /getMemberTierCustomer` returns the customer's loyalty member tier. Tier
changes also arrive on the `customerUpdate` webhook as `memberTierDetails`
(`memberTierId`, `memberTierNameCached`, `memberTierPrecedenceCached`,
`memberTierUpdatedOn`) alongside `loyaltyDetails.loyaltyOptInStatus`.

## Errors

Flat envelope `{error, message, code, reference}`.

| Status | Do this |
|---|---|
| 401 | Re-authorize and retry once. |
| 404 | On a ledger read, treat as zero balance. On a customer read, sync the customer first with `postCustomer`. |
| 422 | Validation failure — will fail identically on retry. Fix the field in `message`. |
| 429 | Exponential backoff with jitter; no documented limit or `Retry-After`. |
| 500 | Retry with backoff; quote `reference` to support. |
