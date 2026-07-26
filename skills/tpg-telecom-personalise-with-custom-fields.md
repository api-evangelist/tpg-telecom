---
name: Personalise messages with custom fields
description: Define account-level custom fields and merge tags in the Vodafone Business Messaging Hub, set their values on contacts, and use them to personalise SMS content.
api: blueprint/tpg-telecom-contacts-management-api.apib
base_url: https://api.messaging.tpgtelecom.com.au
operations:
  - POST /api/v1/contacts/custom-fields
  - GET /api/v1/contacts/contacts/{customFieldId}
  - PATCH /api/v1/contacts/contacts/{customFieldIdToUpdate}
  - DELETE /api/v1/contacts/contacts/{customFieldIdToDelete}
  - GET /api/v1/contacts/custom-fields
  - POST /api/v1/contacts/contacts
  - PATCH /api/v1/contacts/contacts/{contactIdToUpdate}
generated: '2026-07-25'
method: generated
---

# Personalise messages with custom fields

Custom fields are the Messaging Hub's metadata model: a field is defined once at the **account** level, then carries a **value per contact**, and its `mergeTag` is the token used to personalise message content. Every operation below is declared in `blueprint/tpg-telecom-contacts-management-api.apib`.

## 1. Define the field

`POST /api/v1/contacts/custom-fields`

```json
{ "label": "Pet name", "mergeTag": "pet_name", "maxLength": 30, "type": "TEXT" }
```

All four fields are required. `type` is one of `DATE`, `NUMBER`, `PHONE`, `TEXT`, `URL`, `ZIP_CODE`, `NAME`, `EMAIL`.

- `201` returns the definition with its `id` (UUID).
- **`409 conflict` means the custom field already exists.** This operation is one of the six that declare 409 — treat it as "look it up and reuse", never as a retryable error.

## 2. List and look up definitions

`GET /api/v1/contacts/custom-fields` pages with `pageSize` / `nextPageToken` / `prevPageToken` and filters `customFieldIds`, `label`, `mergeTag`. Do this **before** creating, to avoid the 409 round-trip and duplicate near-identical fields.

`GET /api/v1/contacts/contacts/{customFieldId}` reads a single definition. (Note the path: the blueprint routes single-custom-field reads, updates and deletes under `/api/v1/contacts/contacts/…`, not under `/custom-fields/…`. Follow the specification, not the resource name.)

## 3. Set values on contacts

Pass `customFields[]` on create or update:

```json
{ "customFields": [ { "id": "025e93d3-051b-43f9-b12e-4b5842228dee", "value": "John" } ] }
```

- On `POST /api/v1/contacts/contacts` at creation time.
- On `PATCH /api/v1/contacts/contacts/{contactIdToUpdate}` for an existing contact.

Reads return the value enriched with the definition's `mergeTag` and `type`, so a single contact fetch gives you everything needed to render personalised content.

## 4. Respect the constraints

- `maxLength` is declared on the definition — truncate or reject client-side rather than letting the API return `400 validation`.
- The `type` is advisory metadata for how the value is rendered; values still travel as strings.
- Personalisation that fails to resolve shows up as missing content in the sent message, not as an API error — verify values are populated before sending a campaign.

## 5. Maintain and retire

- `PATCH /api/v1/contacts/contacts/{customFieldIdToUpdate}` updates a definition.
- `DELETE /api/v1/contacts/contacts/{customFieldIdToDelete}` removes it → `204`. Retiring a field that campaigns still reference breaks their personalisation; audit templates first.

## Errors

Standard envelope `{ uuid, type, title, detail, invalidFields[] }`: `400 validation`, `401 unauthorized` (empty body), `403 forbidden` (empty body), `409 conflict`, `500/501/502/503/504`. Full catalogue in `errors/tpg-telecom-problem-types.yml`.

## Related

- `data-model/tpg-telecom-data-model.yml` — CustomField and ContactCustomFieldValue
- `conventions/tpg-telecom-conventions.yml` — the metadata model and pagination
- <https://support.messaging.tpgtelecom.com.au/hc/en-us/articles/9329591939087-Using-contact-fields>
