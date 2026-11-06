# Customer API workflow map

Load only the reference for the current task and missing prerequisites. Do not
execute the full workflow unattended. These are optional Bash/curl/jq examples;
the skill installs no scripts or SDK and may use another private HTTPS client.

| Task | Reference | Prerequisites / completion boundary |
| --- | --- | --- |
| Establish private state, request approval, validate key | [Authentication](authentication.md) | Explicit operator-confirmed HTTPS origin; existing approved key preferred. |
| Discover limits and select material/profile | [Catalogs](catalogs.md) | Authenticated client; customer-confirmed selections. |
| Reserve and upload design | [Uploads](uploads.md) | Authorized supported file; complete PUT then await ready asset. |
| Create and verify request | [RFQs](rfqs.md) | Ready asset, catalog selections, quantity and destination. |
| Admit calculation and poll offer | [Quotes](quotes.md) | Actionable RFQ; poll the same quote ID. |
| Review complete itemized offer | [Checkout](checkout.md) | Ready current quote; stop for private human-only checkout. |
| Verify payment and read order | [Orders](orders.md) | Actual paid association; never infer payment from redirects. |
| Diagnose failures and response loss | [Troubleshooting](troubleshooting.md) | Preserve private evidence; no blind write retries. |

Signed upload targets and checkout links are private capabilities, separate from
API authentication. Never use the bearer config on either. No fixed price,
production availability, enabled tax handling, delivery date or live-payment
approval is implied by these examples.
