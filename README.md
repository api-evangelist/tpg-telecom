# TPG Telecom (tpg-telecom)

TPG Telecom Limited is Australia's second-largest telecommunications company and an ASX-listed mobile network operator and fixed broadband carrier, formed by the 2020 merger of Vodafone Hutchison Australia and TPG Corporation. It runs its own mobile network reaching 98.4% of the Australian population and sells under the Vodafone, TPG, iiNet, Internode, Lebara and felix brands, with business, enterprise, government and wholesale services consolidated under Vodafone Business.

**APIs.json:** [https://raw.githubusercontent.com/api-evangelist/tpg-telecom/refs/heads/main/apis.yml](https://raw.githubusercontent.com/api-evangelist/tpg-telecom/refs/heads/main/apis.yml)

## Tags

- Telecommunications
- Australia
- Mobile Network Operator
- Broadband
- Messaging
- SMS
- IoT
- 5G
- Partner Gated

## Timestamps

- **Created:** 2026-07-25
- **Modified:** 2026-07-25

## API Posture

TPG Telecom publishes **no first-party developer portal and no OpenAPI**. Every probed developer hostname either fails to resolve (`developer.tpgtelecom.com.au`, `developers.tpgtelecom.com.au`, `docs.tpgtelecom.com.au`, `opengateway.tpgtelecom.com.au`, `developer.vodafone.com.au`) or returns 404 (`/developer`, `/api`, `/opengateway` on the corporate site; `api.tpgtelecom.com.au` root).

The one public, callable developer surface is the **Vodafone Business Messaging Hub** — and it is not TPG's own software. `messaging.tpgtelecom.com.au` returns HTTP 200 with the page title *"Login to Sinch Engage | Messaging Platform for Growing Businesses"*: a white-labelled Sinch MessageMedia CPaaS console on a TPG-branded host. The API host `api.messaging.tpgtelecom.com.au` is live, returning 401 on `/v1/messages`, `/v1/replies`, `/v1/delivery_reports`, `/v1/webhooks` and `/api/v1/contacts/contacts`. Documentation lives in a Zendesk help centre, and the Contacts API reference is an Apiary API Blueprint whose project owner field reads *"MessageMedia"*. TPG's own credential article closes by telling the reader to "connect with your Sinch MessageMedia account."

### CAMARA and GSMA Open Gateway

**Non-participant.** No CAMARA API is exposed, no Open Gateway portal exists, and TPG is not on the operator side of Aduna. In Australia, Telstra went live first with Number Verification and SIM Swap delivered to Aduna Global, and Optus signalled it would follow. TPG Telecom's public position, from its senior customer security, fraud and scam governance manager, is that it is *"closely watching developments like GSMA Open Gateway, but our priority right now is delivering practical, locally-focused scam prevention measures."* That is a watching brief — not an implementation, and not even a press release to discount.

No TM Forum Open API conformance certification was found. No NEF/SCEF, network-slicing or edge/MEC API is published. The Managed IoT Connectivity page mentions "a catalogue of API's" without naming, linking or documenting one. TPG runs Google Cloud Apigee internally — an API programme with no public face.

## APIs

### TPG Telecom Messaging Hub Contacts API

Contacts, lists and custom fields for the Vodafone Business Messaging Hub — 18 operations across `/api/v1/contacts/contacts`, `/api/v1/contacts/lists` and `/api/v1/contacts/custom-fields`.

- **Human URL:** [https://contactsapiv1tgp.docs.apiary.io/](https://contactsapiv1tgp.docs.apiary.io/)
- **Base URL:** `https://api.messaging.tpgtelecom.com.au`

#### Properties

- [API Blueprint](blueprint/tpg-telecom-contacts-management-api.apib) — harvested verbatim, API Blueprint 1A (not OpenAPI)
- [API Reference](https://contactsapiv1tgp.docs.apiary.io/)
- [Documentation](https://support.messaging.tpgtelecom.com.au/hc/en-us/articles/11469281100559-Contacts-API)
- [Authentication](https://support.messaging.tpgtelecom.com.au/hc/en-us/articles/4750274170383-Creating-new-API-credentials)

### TPG Telecom Messaging Hub REST API

The SMS/MMS REST API of the Messaging Hub, sold as "Access to our REST API" on every plan tier. Live but undocumented in public spec form.

- **Human URL:** [https://support.messaging.tpgtelecom.com.au/hc/en-us/sections/4750925238671-REST-API-Documentation](https://support.messaging.tpgtelecom.com.au/hc/en-us/sections/4750925238671-REST-API-Documentation)
- **Base URL:** `https://api.messaging.tpgtelecom.com.au`

#### Properties

- [Documentation](https://www.vodafone.com.au/business/messaging-hub)
- [Webhooks](https://support.messaging.tpgtelecom.com.au/hc/en-us/articles/4693850901263-Create-manage-webhooks)
- [Sign Up / Console](https://messaging.tpgtelecom.com.au/login)

## Authentication

HTTP Basic (`Base64(api_key:api_secret)`), HMAC (`algorithm="hmac-sha1"`), and a legacy username/password scheme. Keys are minted inside the console by account administrators only — there is no public self-serve developer signup. No OAuth2, no OIDC, and **no CIBA** (both `/.well-known/openid-configuration` and `/.well-known/oauth-authorization-server` return 404).

## Webhooks

Console-configured outbound callbacks with a templated JSON body. Events: SMS received, opt-out occurred, message delivered, message expired. No AsyncAPI is published.

## Links

- [Website](https://www.tpgtelecom.com.au/)
- [About](https://www.tpgtelecom.com.au/about-us)
- [Messaging Hub support](https://support.messaging.tpgtelecom.com.au/hc/en-us)
- [Vulnerability disclosure](https://www.tpgtelecom.com.au/.well-known/security.txt)
- [GitHub](https://github.com/tpgtelecom) (0 public repositories)
- [LinkedIn](https://www.linkedin.com/company/tpg-telecom/)
- [Investor relations](https://www.tpgtelecom.com.au/investor-relations)

## Review

See [review.yml](review.yml) for the full reviewer findings, probe log with HTTP statuses, and provenance for the harvested API Blueprint.
