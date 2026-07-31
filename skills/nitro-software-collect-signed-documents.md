---
name: Track and collect signed documents from Nitro Sign
description: Check an envelope's status and download the sealed documents and audit trail once signing completes.
api: openapi/nitro-software-openapi-original.json
base_url: https://api.gonitro.dev
operations: [getEnvelope, listDocuments, downloadSealedEnvelope, downloadSealedDocument, downloadAuditTrailOfEnvelope]
---

# Track and collect signed documents

Once an envelope has been sent, monitor it and retrieve the completed artifacts.

## Auth
`Authorization: Bearer <token>` from `POST /oauth/token` (client-credentials). Renew on `401`.

## Steps
1. **getEnvelope** — `GET /sign/envelopes/{envelopeID}` to read current status. The envelope
   reaches `EnvelopeSigningCompleted` then `EnvelopeSealed` once every signer has signed.
   (Prefer the `EnvelopeSealed` webhook event over polling.)
2. **listDocuments** — `GET /sign/envelopes/{envelopeID}/documents` to enumerate the documents
   and their `documentID`s.
3. **downloadSealedDocument** — `GET /sign/envelopes/{envelopeID}/documents/{documentID}:download-sealed`
   for a single sealed (signed) PDF.
4. **downloadSealedEnvelope** — `GET /sign/envelopes/{envelopeID}:download-sealed` for a ZIP of
   all sealed documents plus the audit trail.
5. **downloadAuditTrailOfEnvelope** — `GET /sign/envelopes/{envelopeID}:download-audit-trail`
   for the standalone audit-trail PDF (tamper-evident signing record).

## Rules
- Only sealed envelopes yield sealed documents; a `404`/`409` means it is not sealed yet.
- Result/download URLs are time-limited — a `410 Gone` means re-request.
- Events may arrive out of order; derive state from `getEnvelope`, not event sequence.
- Errors are RFC 9457 `application/problem+json`.
