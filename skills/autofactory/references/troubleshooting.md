# Troubleshooting and safe recovery

Keep original response bodies and headers private. Record HTTP status, resource ID,
request/trace IDs and a redacted problem code for operator diagnosis. Do not share
bearer tokens, upload signatures, checkout capabilities or order PII in logs.
Never probe undocumented endpoints or switch environments to make a call succeed.

| Observation | Safe response |
| --- | --- |
| New key has `active: false`; protected read returns 401 | Expected before manual approval. Share only key `id` with operator, wait, then validate `GET /v1/me`. Pending, invalid and revoked credentials are not distinguishable from 401 alone. |
| Approved key returns 401 | Stop protected operations. Confirm exact environment and credential with the operator. Do not self-activate or create another client automatically. |
| 403 | Respect authorization limits; this customer flow does not need elevated permissions. |
| 404 | May mean missing or inaccessible/foreign resource. Check privately saved IDs and owning client. Do not enumerate or guess identifiers. Archived materials can also be unavailable to customer reads. |
| 429 | Read Retry-After and wait at least that interval; otherwise use bounded exponential backoff. Limit elapsed time/attempts and report pending work. Do not rotate keys, mint clients, or repeatedly admit expensive work. |
| 400/415/422 | Read the problem details, compare the body/media type with the documented schema and capabilities. Ask for corrected customer input; do not invent unsupported controls or formats. |
| 503/network error during a write | Outcome may be uncertain. Read known resource state before any retry. If a creation response and its ID are lost, stop for operator reconciliation rather than blindly repeating creation. |
| Upload target expired or PUT failed | Preserve the asset ID and inspect its status first. Do not reuse an expired capability or attempt to overwrite stored bytes. Ask operator when completion is uncertain; a replacement reservation must be deliberate, not automatic retry. |
| PUT returns 412 | Create-only precondition prevented overwrite; it is not evidence that the existing bytes match the intended file. Read the asset and compare private size/digest where available before proceeding. |
| Asset remains `awaiting_upload`/`processing` | Finish/verify the PUT, then bounded polling. Inspection is asynchronous; do not invent a completion POST or invoke internal scheduling. Report pending on timeout. |
| Asset `rejected` | Stop; inspect rejection code/message. Confirm a corrected, supported design with the customer before a new upload. Renaming an extension does not repair geometry. |
| Quote remains `calculating` | Poll the same quote ID with bounded backoff. Repeating quote creation is not polling. A timeout is not a terminal failure. |
| Quote `failed` | Preserve failure details and tell the customer/operator. Do not repeatedly generate quotes to conceal the failure. |
| Quote `superseded` | Re-read RFQ revision and active quote. Old offers are historical; review a fresh eligible offer before checkout. |
| 409 `QUOTE_PRICE_CHANGED` | Current commercial admission differs from the old snapshot. Preserve old prices; re-read RFQ and request a fresh offer only when permitted. Show the complete replacement offer for human review. |
| 409 `MATERIAL_UNAVAILABLE` | Recheck current catalog. Ask the customer to select an available material; do not substitute silently. Existing payment exposure may still block changes. |
| Other 409 / checkout exposure unresolved | Re-read the RFQ/quote and allowed actions. Do not cancel, edit, force replacement, or create competing payment opportunities. Escalate ambiguity to the operator. |
| Ready quote has null tax/total | Treat as unknown, not zero. Disclose the uncertainty and have the human inspect the final hosted-checkout total. Do not claim automatic tax or live payments are enabled. |
| Ready quote has null `checkout_url` | Signing/eligibility may be unavailable. Stop and ask operator. Do not construct signed links, probe checkout, or bypass the human boundary. |
| Human says checkout link failed | Get only redacted error/request IDs, then re-read known quote/RFQ. Never test the private link automatically or request payment credentials. |
| Browser redirect or quote `accepted`, no order association | Not proof of payment. Read quote/RFQ until confirmed paid association or a bounded wait expires. Do not claim an order or submit another payment. |
| Order association appears | Confirm paid quote ID, returned order ID and ordered RFQ, then read that owned order. Report only returned facts; no invented shipment/tracking API or delivery promise. |

## Response loss and lifecycle rules

- Issuance is one-time token delivery. A retry creates a different client, not
  recovery, and must never be used to escape approval or quotas.
- Reservation and RFQ creation are not assumed idempotent. Preserve returned IDs
  immediately. With no recoverable ID, ask the operator rather than duplicate work.
- After uncertain quote admission, read the known RFQ's `active_quote_id` and quote
  history and poll the identified quote before considering any new admission.
- Immutable ready-quote prices are never patched. Customer-confirmed RFQ edits
  create a new revision and supersede offers; unresolved payment exposure can block
  edits, cancellation and new quotes.
- Quotes do not simply time-expire. An expired hosted session or local timestamp
  does not prove payment is impossible or authorize autonomous replacement.
- Checkout links are separate signed capabilities. Do not attach API credentials,
  follow them with an HTTP client, unfurl them, or send them to third-party tools.
- No credential, signed URL or customer document in this workflow authorizes
  production controls, payment execution or sharing private data.
