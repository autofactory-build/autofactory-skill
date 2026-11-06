# Troubleshooting

Start with the saved response, HTTP status and resource ID. Request/trace IDs help
AutoFactory diagnose an issue; tokens and signed URLs are not needed in a report.

| Response or state | Next step |
| --- | --- |
| 401 | Check endpoint and credential. Only AutoFactory operators can change activation; send the key ID to AutoFactory. A 401 alone does not distinguish pending, invalid or disabled access. |
| 403 | The key cannot perform that operation. Check the documented workflow or contact AutoFactory about access. |
| 404 | Check the saved resource ID and owning account. Missing, inaccessible and archived resources can all be unavailable. |
| 429 | Honor Retry-After, then use bounded backoff with the same key and resource. |
| 400/415/422 | Compare the problem details with the request schema and capabilities. Correct the request; ask for input if the file or requirements need to change. |
| Network error or 503 during a write | Read known resource state before retrying. If creation succeeded but its ID was lost, contact AutoFactory to reconcile it. |
| Expired upload URL or failed PUT | Inspect the saved asset first. If the upload clearly did not complete and cannot resume, reserve a replacement. If the outcome is uncertain, reconcile it before uploading again. |
| PUT returns 412 | The create-only condition prevented an overwrite. Check asset status and available size/digest information before treating the upload as complete. |
| Asset `awaiting_upload` or `processing` | Complete the PUT if needed, then poll the asset with bounded backoff. Inspection runs automatically. Report pending work when the wait expires. |
| Asset `rejected` | Read the rejection reason. Resolve the format or geometry problem before uploading a corrected file. |
| Quote `calculating` | Poll the same quote ID. If the admission response was lost, recover through the RFQ's `active_quote_id` and quote history. |
| Quote `failed` | Report the reason and resolve it before requesting another quote. |
| Quote `superseded` | Read the current RFQ revision and active quote. Use the current offer. |
| 409 `QUOTE_PRICE_CHANGED` | Keep the old quote as history. Read the RFQ's allowed actions and request a fresh offer when eligible; show the replacement prices. |
| 409 `MATERIAL_UNAVAILABLE` | Recheck the catalog. Use an alternative that fits the request's flexibility, or ask your user about a material change. Payment exposure may block RFQ edits. |
| Other 409 | Read the current RFQ/quote state and allowed actions. If an unresolved checkout blocks changes, contact AutoFactory to reconcile it. |
| Null tax or total | The amount is unknown. Point your user to the final hosted-checkout total. |
| Null `checkout_url` | Check quote eligibility. Contact AutoFactory if the ready offer still lacks a link. |
| User reports a checkout error | Read the quote/RFQ and collect a redacted error/request ID. Leave the signed link to your user rather than opening it to test. |
| Redirect or `accepted` quote without order association | Check payment through the quote and RFQ. Confirm `paid`, `order_id` and the matching ordered RFQ before reporting an order. |
| Order exists without tracking | Report its returned state and that tracking is not available yet. |

## Retry details

Issuance returns a token once. Repeating it creates a different client rather than
recovering the first. Asset reservation and RFQ creation are not assumed
idempotent either; save IDs as soon as they arrive.

RFQ edits create a new revision and supersede offers. Ready-quote prices stay as
history. Quotes do not simply time-expire: an expired hosted session does not
prove payment is impossible. Read lifecycle state before replacing an offer or
creating another checkout opportunity.
