---
name: Run an async Nitro PDF Services job
description: Submit an asynchronous PDF Services conversion/transformation, then poll or receive a callback for the result.
api: openapi/nitro-software-openapi-original.json
base_url: https://api.gonitro.dev
operations: [forwardConversionsRequest, forwardTransformationsRequest, getJobResult, getJobStatus_1]
---

# Run an async PDF Services job

The PDF Services / Document Intelligence Platform exposes grouped endpoints —
`/conversions`, `/transformations`, `/extractions`, `/generations` — each selecting a `method`
(e.g. `optimize`, `ocr`, `merge`, `redact`, `pdftoms`). Jobs can run sync or async.

## Auth
`Authorization: Bearer <token>` from `POST /oauth/token` (client-credentials).

## Steps
1. **Submit async** — e.g. `POST /transformations` (forwardTransformationsRequest) or
   `POST /conversions` (forwardConversionsRequest) with the file, the `method`, and set header
   `Prefer: respond-async`. Optionally include a `delivery.callback.URL` to be POSTed once the
   job is accepted, or `uploadResultTo` / `uploadResultsTo` to have Nitro deliver the output.
2. **Callback (optional)** — Nitro POSTs `{ jobID, location }` once, at acceptance (not completion).
3. **getJobStatus_1** — `GET /jobs/{jobID}/status` to poll progress.
4. **getJobResult** — `GET /jobs/{jobID}` to fetch the final result / pre-signed download once complete.

## Rules
- A `422 InvalidPlatformMethodError` lists the `allowed_methods` for the group — pick one of those.
- Size/page limits: up to 100 MB and 500 pages per document; `413 FileTooLarge` otherwise.
- `410 Gone` on a job result means it expired — re-submit the original request.
- Errors are RFC 9457 `application/problem+json`; back off on `429` using `extensions.retry_after`.
