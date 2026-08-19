---
name: Service a CCPA/GDPR data request against Friendbuy
description: >-
  Handle a data-subject access or erasure request for a customer whose referral,
  loyalty and conversion history lives in Friendbuy — retrieve the stored data,
  delete it, and block a bad actor from campaigns.
api: openapi/friendbuy-customers-api-openapi.yml
operations:
  - createAuthorization
  - getUserData
  - deleteUserData
  - getCustomer
  - postBlockUsers
generated: '2026-08-13'
method: generated
source: openapi/ + https://developers.friendbuy.com
---

# Service a CCPA/GDPR data request

Friendbuy exposes data-subject rights as first-class API operations, which is
unusual and worth using rather than emailing support. Base URL
`https://mapi.fbot.me/v1`; authenticate with `createAuthorization` and send
`Authorization: Bearer <token>`.

The subject is identified by **email and/or customer id** — the same
merchant-supplied `customerId` used everywhere else in the platform. Supply both
when you have both; a subject who signed up under one email and purchased under
another may have more than one record.

## 1. Access request — `getUserData`

`GET /getUserData` with the subject's email and/or customer id as query
parameters. Returns the data Friendbuy holds for that subject.

A `404` means Friendbuy holds no record for that identifier. For an access request
that is a complete and reportable answer — "no data held" — not a failure. Record
the `reference` value from the error envelope as your evidence.

Confirm the identity resolution first with `getCustomer` if you need to be sure
you are addressing the right record before you act on it.

## 2. Erasure request — `deleteUserData`

`DELETE /deleteUserData` with the subject's email and/or customer id as query
parameters.

Two things to get right:

- **Deletion is not reversible and there is no idempotency contract.** Confirm the
  identifier resolves to exactly the intended subject *before* you send it.
- **Deletion breaks attribution.** The subject's referral links, attribution ids
  and reward history are part of the record. Export whatever you are legally
  required to retain — via `getUserData`, or the analytics reads
  (`getReferralRewards`, `getDistributedAdvocateRewards`) — before deleting.

Run the erasure against Friendbuy at the same time as your own systems, and
mirror any `emailOptOut` webhook you have received into your own suppression list
so the subject is not re-contacted from the other side.

## 3. Blocking a bad actor — `postBlockUsers`

`POST /postBlockUsers` adds users to a campaign block list. This is **not** a
privacy operation and must not be used as one: it suppresses a user from earning
and sharing, but it does not delete their data. Blocked referral codes then report
`status: "blocked"` from `getReferralStatus`, and `getReferralStatus` returns
`400` for a blocked code or advocate.

Use `postBlockUsers` for fraud and abuse. Use `deleteUserData` for erasure. They
answer different questions.

## Errors

Flat envelope `{error, message, code, reference}`.

| Status | Meaning here |
|---|---|
| 401 | Token expired — re-authorize and retry once. |
| 404 | No record for that identifier. For an access request this is a valid answer. |
| 422 | The identifier was malformed. Fix it; retrying unchanged fails identically. |
| 429 | Back off with jitter. |
| 500 | Retry with backoff; quote `reference` to Friendbuy support. |

Keep the `reference` value from every response as part of your compliance audit
trail — it is the only correlation identifier this API emits.
