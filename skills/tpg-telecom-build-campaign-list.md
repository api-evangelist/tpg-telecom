---
name: Build and maintain a campaign list
description: Create a contact list in the Vodafone Business Messaging Hub and manage its membership one contact at a time or in bulk, using the TPG Telecom Contacts API list operations.
api: blueprint/tpg-telecom-contacts-management-api.apib
base_url: https://api.messaging.tpgtelecom.com.au
operations:
  - POST /api/v1/contacts/lists
  - GET /api/v1/contacts/lists/{listId}
  - PATCH /api/v1/contacts/lists/{listIdToUpdate}
  - DELETE /api/v1/contacts/lists/{listIdToRemove}
  - GET /api/v1/contacts/lists
  - POST /api/v1/contacts/lists/{listId}/contacts/{contactToAdd}
  - DELETE /api/v1/contacts/lists/{listId}/contacts/{contactId}
  - PATCH /api/v1/contacts/lists/{listId}/contacts
generated: '2026-07-25'
method: generated
---

# Build and maintain a campaign list

Lists are how the Messaging Hub targets an SMS/MMS campaign. All eight operations below are declared in `blueprint/tpg-telecom-contacts-management-api.apib`.

Authenticate with Basic or HMAC as described in `authentication/tpg-telecom-authentication.yml`.

## 1. Create the list

`POST /api/v1/contacts/lists`

```json
{ "name": "My group", "alias": "Group1" }
```

- `name` is required; `alias` is an optional short handle.
- `201` returns the list with its `id` (UUID), `accountId` and `vendorId`. Keep the `id` — every membership call needs it.

## 2. Find an existing list first

Before creating, page `GET /api/v1/contacts/lists` with `pageSize` / `nextPageToken` / `prevPageToken` and the filters `listIds`, `alias` or `name`. There is no idempotency key on create, so a retried create makes a second list with the same name.

`GET /api/v1/contacts/lists/{listId}` reads one list.

## 3. Add and remove members

Three shapes, pick by volume:

- **One in:** `POST /api/v1/contacts/lists/{listId}/contacts/{contactToAdd}` → `204`.
- **One out:** `DELETE /api/v1/contacts/lists/{listId}/contacts/{contactId}` → `204`.
- **Many at once:** `PATCH /api/v1/contacts/lists/{listId}/contacts` — adds and removes multiple contacts in a single call. Prefer this for any batch; it is one request, one failure surface, and it avoids drift halfway through a loop.

You can also seed membership at contact-creation time by passing `lists[].id` on `POST /api/v1/contacts/contacts`.

## 4. Read the membership back

`GET /api/v1/contacts/contacts?listIds={listId}` pages the contacts on the list. Filter further with `channelSubscriptionState=SUBSCRIBED` to see who is actually messageable — an `UNSUBSCRIBED` contact stays on the list but must not be messaged.

## 5. Rename or delete

- `PATCH /api/v1/contacts/lists/{listIdToUpdate}` changes `name` / `alias` → `200`.
- `DELETE /api/v1/contacts/lists/{listIdToRemove}` removes the list → `204`. Deleting a list does not delete its contacts.

## Failure modes

- `400 validation` — check `invalidFields[]`; `invalidIds[]` names the contact or list ids that were rejected in a bulk call.
- `409 conflict` — a list with that identity already exists; search and reuse.
- `401` / `403` — empty bodies; credential or account-permission problem.
- `503` / `504` — retry with backoff and check the status page's REST API component.

## Before you message the list

Sending limits are volume-based, not rate-based: hitting the daily or monthly limit marks messages **Discarded** with delivery status code **301**, not an HTTP error. See `rate-limits/tpg-telecom-rate-limits.yml` and `errors/tpg-telecom-delivery-status-codes.yml`.

## Related

- `data-model/tpg-telecom-data-model.yml`
- `conventions/tpg-telecom-conventions.yml`
