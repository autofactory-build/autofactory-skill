# Send the quote to the buyer

Use a ready [quote](quotes.md) for the current RFQ revision. The buyer completes
checkout; the agent prepares the offer and sends the link.

Read the saved `quote.json` and compare its revision with the current RFQ.

```bash
jq -e '.state == "ready"' "$AF_PRIVATE_DIR/quote.json" >/dev/null
# Full itemized commercial review, with checkout capability deliberately omitted.
jq '{id,rfq_id,rfq_revision,state,destination,items,line_items,
     subtotal,shipping,tax,total}' "$AF_PRIVATE_DIR/quote.json"
```

Show quantities, material/profile choices, item costs, quote-level line items,
shipping, tax, total and currency. Item `cost` covers the full quantity; material
and manufacturing-time estimates are per unit. Money uses `amount_cents` and
`currency`. Use returned prices. A null tax or total is unknown; point the buyer
to the final hosted-checkout amount.

Send `checkout_url` directly to the buyer with the offer. It is a signed link that
can create a payment session when opened, so exclude it from automated previews,
link checks and HTTP requests. The buyer opens it and pays; hosted checkout
collects their full shipping address. If the link is null, check quote eligibility
and contact AutoFactory if needed.
