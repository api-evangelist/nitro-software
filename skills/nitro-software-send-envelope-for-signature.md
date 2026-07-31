---
name: Send an envelope for signature with Nitro Sign
description: Create a Nitro Sign envelope, attach a PDF, add a signer, place a signature field, and send it for signing.
api: openapi/nitro-software-openapi-original.json
base_url: https://api.gonitro.dev
operations: [createEnvelope, createDocument, createParticipant, createField, sendForSigning, listSignerSigningUrls]
---

# Send an envelope for signature

Use the Nitro Sign API to build and dispatch a signature request. Requires a Nitro Sign
Enterprise application (Client ID + Secret from the Admin Portal).

## Auth
Get a bearer token first: `POST /oauth/token` with `{ "clientID", "clientSecret" }`.
Use the returned `accessToken` as `Authorization: Bearer <token>` on every call. Cache it to
`expiresIn` and renew on a `401`.

## Steps
1. **createEnvelope** — `POST /sign/envelopes` with a name and notification/signing settings.
   Capture the returned `envelopeID`.
2. **createDocument** — `POST /sign/envelopes/{envelopeID}/documents` as `multipart/form-data`
   with the PDF. Optionally pass `nitroDocumentID` (query) to correlate your own id. Max 15
   documents per envelope; envelope-document uploads must be PDF (use `/sign/conversions` or the
   PDF Services conversions to convert other formats first). Capture `documentID`.
3. **createParticipant** — `POST /sign/envelopes/{envelopeID}/participants` with the signer's
   email and role `SIGNER` (accepts a single object or an `items[]` array for bulk). Capture `participantID`.
4. **createField** — `POST /sign/envelopes/{envelopeID}/documents/{documentID}/fields` placing a
   signature field via bounding-box coordinates (page, x, y, width, height in PDF points) and
   assigning it to `participantID`. Accepts a single field or `{ items: [...] }`.
5. **sendForSigning** — `POST /sign/envelopes/{envelopeID}:send-for-signing`. The envelope moves
   to `SENT`. A `409` here usually means it was already sent.
6. **listSignerSigningUrls** *(optional, embedded signing)* — `GET /sign/envelopes/{envelopeID}/participants/signing-urls`
   returns each SIGNER's signing URL + expiry. Only valid while the envelope is `SENT`.

## Rules
- Errors are RFC 9457 `application/problem+json`; read `type`, `detail`, and `extensions`.
- Respect `429` `RateLimitExceeded` — wait `extensions.retry_after` seconds, exponential backoff.
- Prefer webhooks over polling to learn when signing completes (see conventions + webhooks artifacts).
- Do not enforce strict response-schema validation; ignore unknown/new optional fields.
