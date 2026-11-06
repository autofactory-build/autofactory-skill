# Human review and checkout boundary

Optional Bash/curl/jq examples; no scripts or SDK are installed. Load [authentication](authentication.md) first if its private directory, approved origin and bearer config are not already prepared. Requires a ready [quote](quotes.md) for the intended RFQ revision.

Use the saved ready `quote.json` and its known `AF_RFQ_ID`. This task only
reviews the offer and hands checkout to the human; it never executes payment.

```bash
jq -e '.state == "ready"' "$AF_PRIVATE_DIR/quote.json" >/dev/null
# Full itemized commercial review, with checkout capability deliberately omitted.
jq '{id,rfq_id,rfq_revision,state,destination,items,line_items,
     subtotal,shipping,tax,total}' "$AF_PRIVATE_DIR/quote.json"
```

Check this quote's RFQ ID and revision against a fresh RFQ read. Show quantity,
material/profile/configuration, every item cost and line item, separate shipping,
tax, total and currency. Item `cost` is for its full quantity; slicer estimates are
single-unit. Money uses `amount_cents` and `currency`. Never hardcode a minimum,
shipping rate or tax value. A null total/tax is unknown, not free: disclose it and
require the human to inspect the final hosted total before paying. Test pricing
is not approval for real payments or evidence of enabled production tax handling.

Read `checkout_url` from the private quote JSON using the agent's private state
access, without printing it into shared tool output. After human review, deliver
it only through an approved private channel. If no such channel exists, stop.
Do not curl, prefetch, preview, auto-open or test the link. It can create a payment
session when opened. The human opens it and completes any payment themselves.
A null link means stop and ask the operator, not invent another checkout route.
