---
name: Manage contacts in the Vodafone Business Messaging Hub
description: Create, retrieve, update and delete contacts (and their SMS/WhatsApp channels) in the TPG Telecom Messaging Hub Contacts API, honouring subscription state and the platform's non-idempotent write semantics.
api: blueprint/tpg-telecom-contacts-management-api.apib
base_url: https://api.messaging.tpgtelecom.com.au
operations:
  - POST /api/v1/contacts/contacts
  - GET /api/v1/contacts/contacts/{contactId}
  - PATCH /api/v1/contacts/contacts/{contactIdToUpdate}
  - DELETE /api/v1/contacts/contacts/{contactIdToDelete}
  - GET /api/v1/contacts/contacts
generated: '2026-07-25'
method: generated
---

# Manage contacts

Every operation below exists verbatim in `blueprint/tpg-telecom-contacts-management-api.apib`, the API Blueprint TPG Telecom publishes for the Vodafone Business Messaging Hub.

## Before you start

- **Get credentials.** Only a Messaging Hub account administrator can mint them, from the console under Settings > API Settings. Two schemes are supported:
  - Basic: `Authorization: Basic Base64(api_key:api_secret)`
  - HMAC: `Authorization: hmac username="<API KEY>", algorithm="hmac-sha1", headers="Date Content-MD5 request-line", signature="<SIGNATURE>"` — sign `Date`, `Content-MD5` (body requests only) and the request line joined by line breaks, then Base64 the HMAC-SHA1 digest.
- **There is no sandbox.** No test mode, no test key prefix, no magic numbers. Everything you do here touches live account data.
- **There is no idempotency key.** If a create times out, look the contact up with `GET /api/v1/contacts/contacts` filtered by `channelIds` before retrying — a blind retry creates a duplicate.

## Create a contact

`POST /api/v1/contacts/contacts` with `Content-Type: application/json`.

- `channels[]` is the only required field. Each channel needs `channelId` (E.164, e.g. `+61412345678`) and `type` (`SMS` or `WHATSAPP`); `subscriptionState` is `SUBSCRIBED` or `UNSUBSCRIBED`.
- Optional: `firstName`, `lastName`, `alias`, `dateOfBirth`, `country`, `state`, `location`, `note`, `lists[].id`, `customFields[]` (`id` + `value`).
- Success is `201` with the created contact, including its server-assigned `id` (UUID), `accountId`, `vendorId`, `createdDate` and `lastModifiedDate`.

**Consent rule you must not fight:** if the channel was previously `UNSUBSCRIBED`, the contact is created `UNSUBSCRIBED` regardless of what you sent. Re-subscription is a deliberate, separate act — never treat the create response's state as the state you asked for; read it back.

## Read a contact

`GET /api/v1/contacts/contacts/{contactId}` returns one contact by UUID.

## Search contacts

`GET /api/v1/contacts/contacts` pages through the account with cursor pagination:

- `pageSize` (default **1000**), `nextPageToken`, `prevPageToken`.
- Filters: `listIds`, `contactIds`, `channelIds`, `channelTypes` (`PHONE`, `SMS`, `EMAIL`), `channelSubscriptionState` (`SUBSCRIBED`, `UNSUBSCRIBED`).
- Response envelope: `{ content[], nextPageToken, prevPageToken, totalElements }`. With no filter, results are sorted by creation date.

Follow `nextPageToken` until it is absent; do not assume `totalElements` fits one page.

## Update a contact

`PATCH /api/v1/contacts/contacts/{contactIdToUpdate}` — partial update of the same fields as create. Returns `200`.

## Delete a contact

`DELETE /api/v1/contacts/contacts/{contactIdToDelete}` — returns `204` with no body. This is destructive and unrecoverable through the API.

## Handling errors

Every operation declares the same failure set. The envelope is `{ uuid, type, title, detail, invalidFields[] }` served as `application/json` (problem-details shaped, but not RFC 9457).

| Status | `type` | What to do |
|---|---|---|
| 400 | `validation` | Read `invalidFields[].name` + `.code` (e.g. `must_not_be_empty`) and fix the payload. Do not retry unchanged. |
| 401 | `unauthorized` | Empty body. Credentials missing, wrong, or the HMAC signing string is malformed. |
| 403 | `forbidden` | The key's account lacks permission. Empty body. |
| 409 | `conflict` | The resource already exists — fetch and update instead of recreating. |
| 500/501/502 | server | Do not retry writes blindly; there is no idempotency key. |
| 503/504 | server | Retry reads with backoff. Check <https://status.messaging.tpgtelecom.com.au> (REST API component) before escalating. |

Quote the `uuid` from the error body when you raise a support ticket — it is the platform's error correlation id.

## Related

- `conventions/tpg-telecom-conventions.yml` — pagination, identifiers, error envelope
- `errors/tpg-telecom-problem-types.yml` — full error catalogue
- `data-model/tpg-telecom-data-model.yml` — Contact / Channel / List / CustomField graph
