# API workflow

Open the reference for the task at hand. Each has optional curl examples; any HTTP
client works. Continue through quoting with the context available, asking only for
missing inputs that block progress. Your user completes checkout.

| Task | Reference | Inputs / result |
| --- | --- | --- |
| Set up access or get a key | [Authentication](authentication.md) | Configured endpoint; only AutoFactory operators activate keys. |
| Choose material and profile | [Catalogs](catalogs.md) | Request and preferences; use suitable catalog defaults. |
| Upload models | [Uploads](uploads.md) | Reserve each distinct file; poll each asset until ready. |
| Create an RFQ | [RFQs](rfqs.md) | Ready assets, per-item choices and quantities, destination. |
| Get a quote | [Quotes](quotes.md) | Actionable RFQ; poll the returned quote ID. |
| Send the offer and checkout link | [Checkout](checkout.md) | Ready current quote; user opens the link and pays. |
| Track an order | [Orders](orders.md) | API-confirmed payment and associated order ID. |
| Recover from errors | [Troubleshooting](troubleshooting.md) | Saved resource IDs and API responses. |

Upload requests use the returned signed target rather than API authentication.
Checkout links are for your user's browser. Prices, availability and order updates
come from API responses.
