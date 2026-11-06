# Send the quote to the buyer

Use a ready [quote](quotes.md) for the current RFQ revision. Compare its RFQ ID and
revision with the current RFQ before sending the offer.

Show every model, quantity, material/profile choice, item cost, quote-level line
item, shipping, tax, total and currency. Item `cost` covers its full quantity;
material and manufacturing-time estimates are per unit. Money uses `amount_cents`
and `currency`. Use returned prices. A null tax or total is unknown; point the
buyer to the final hosted-checkout amount.

Send `checkout_url` directly to the buyer with the offer. It is a signed link that
can create a payment session when opened, so exclude it from automated previews,
link checks and HTTP requests. The buyer opens it and pays; hosted checkout
collects their full shipping address. If the link is null, check quote eligibility
and contact AutoFactory if needed.
