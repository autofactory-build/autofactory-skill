# Request one quote and poll that same resource

Optional Bash/curl/jq examples; no scripts or SDK are installed. Load [authentication](authentication.md) first if its private directory, approved origin and bearer config are not already prepared. Requires a verified, actionable [RFQ](rfqs.md).

Set `AF_RFQ_ID` to the previously verified owned RFQ ID; this task does not
create an RFQ. The quote ID is read from the saved admission response below.

```bash
# Naked POST: no body and no Content-Type. Expected HTTP status: 202.
AF_STATUS=$(curl --disable --config "$AF_PRIVATE_DIR/api.curl" \
  --silent --show-error --fail-with-body --proto '=https' \
  --request POST "$AF_ORIGIN/v1/rfqs/$AF_RFQ_ID/quote" \
  --dump-header "$AF_PRIVATE_DIR/quote.headers" \
  --output "$AF_PRIVATE_DIR/quote.json" \
  --write-out '%{http_code}') || exit 1
test "$AF_STATUS" = 202 || exit 1
AF_QUOTE_ID=$(jq -er '.id' "$AF_PRIVATE_DIR/quote.json")
# After the saved Retry-After interval, or bounded backoff if absent:
AF_STATUS=$(curl --disable --config "$AF_PRIVATE_DIR/api.curl" \
  --silent --show-error --fail-with-body --proto '=https' \
  --request GET "$AF_ORIGIN/v1/quotes/$AF_QUOTE_ID" \
  --dump-header "$AF_PRIVATE_DIR/quote.headers" \
  --output "$AF_PRIVATE_DIR/quote.json" \
  --write-out '%{http_code}') || exit 1
test "$AF_STATUS" = 200 || exit 1
jq '{id,rfq_id,rfq_revision,state,failure}' "$AF_PRIVATE_DIR/quote.json"
```

Repeat the quote GET, not the POST, while `calculating`. Bound elapsed time and
report pending status if the bound expires. Stop on `failed` or `superseded` and
explain the returned failure/state. Never treat timeout as permission to retry an
ambiguous admission. Read the known RFQ's `active_quote_id` and `quotes` to recover.
