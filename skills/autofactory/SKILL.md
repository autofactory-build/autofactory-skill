---
name: autofactory
description: Upload 3D models, get manufacturing quotes, and track orders with AutoFactory.
compatibility: Requires an AutoFactory API endpoint and HTTPS access. Examples use Bash, curl and jq; any HTTP client works.
metadata:
  author: Joseph Rodrigues (@DoWhileGeek)
  version: '0.1.0'
---

# AutoFactory skill

Get a 3D model made. Upload the file, choose a material, get a quote, and send the
checkout link to the person buying it. After payment, track the order through the API.

Work from the request and context you already have. Carry the task through to a
ready quote without asking for approval at each step. Use stated preferences and
sensible catalog defaults; ask only when missing information or a meaningful
tradeoff blocks progress. Include your selections with the quote.

## Get started

Use the configured AutoFactory API endpoint and credentials. If you need a key,
`POST /v1/api-keys` issues one without a request body. Save the token with your
usual credential tools. New keys start with `active: false`; send the key ID to
AutoFactory for activation, then check access with `GET /v1/me`.

The API examples use `AF_ORIGIN` for the HTTPS endpoint. See
[authentication](references/authentication.md) for setup and exact requests.

## Get a quote

1. Read `GET /v1/capabilities`, then `GET /v1/materials` and `GET /v1/profiles`.
   Follow pagination and choose an available material and active profile that fit
   the request. Check file formats and limits before uploading. Use STL and
   `overrides: {}` for the documented flow. See [catalogs](references/catalogs.md).
2. Reserve an upload with `POST /v1/assets`. PUT the file to the returned URL with
   its required headers, using a separate request without the API bearer token.
   Poll `GET /v1/assets/{asset_id}` until `ready` or `rejected`. Inspection runs
   automatically; there is no upload-completion endpoint. See [uploads](references/uploads.md).
3. Create an RFQ with `POST /v1/rfqs`. Each item needs `manufacturing_method:
   fdm_3d_printing`, `asset_id`, `material_id`, `quantity`, and `configuration`
   containing `profile_id` and `overrides: {}`. Add the destination country and
   postal code. Read the RFQ back and check `missing_fields` and `allowed_actions`.
   See [RFQs](references/rfqs.md).
4. Request a quote with `POST /v1/rfqs/{rfq_id}/quote` (singular, no body).
   Save the returned quote ID and poll `GET /v1/quotes/{quote_id}` while it is
   `calculating`. Honor Retry-After and bound the wait. See [quotes](references/quotes.md).
5. When the quote is `ready`, show the items, quantities, material/profile choices,
   item costs, quote-level `line_items`, subtotal, shipping, tax, total and currency.
   Money uses integer cents; item costs cover the full quantity. Material and
   manufacturing-time estimates are per unit, not delivery estimates. A null tax
   or total is unknown; the buyer can check the final amount in hosted checkout.

## Checkout and orders

Send the ready quote's `checkout_url` directly to the buyer with the itemized offer.
The buyer opens the link and pays. Opening it can create a payment session, so
leave it out of automated previews, link checks and HTTP requests. Hosted checkout
collects the full shipping address. See [checkout](references/checkout.md).

To check payment, read the quote and RFQ. Once the quote is `paid` with an
`order_id` and the RFQ confirms that association, read `GET /v1/orders/{order_id}`.
Use the returned order state and tracking details for updates. A redirect or an
`accepted` quote alone does not confirm payment. See [orders](references/orders.md).

## If something fails

- **401:** check the configured endpoint and credential. A new key may still need
  activation; issuing another key will not fix that.
- **429:** honor Retry-After, then resume with bounded backoff.
- **Uncertain write:** read the saved resource before retrying. For quote admission,
  recover through the RFQ's `active_quote_id` and quote history.
- **Changed RFQ or pricing:** read the current revision and allowed actions, then
  get a replacement offer when eligible. Keep the old quote as history.
- **Rejected asset or failed quote:** report the returned reason and resolve it
  before trying again.

See [troubleshooting](references/troubleshooting.md) for specific responses. Load
only the reference you need; the [workflow map](references/api-workflow.md) is an index.
