# Get a quote

Use the [API access setup](authentication.md) and an [RFQ](rfqs.md) whose
`allowed_actions` includes `create_quote`.

Set `AF_RFQ_ID` to the saved RFQ ID. The admission response returns the quote ID.

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

Poll the same quote with GET while `calculating`. Use bounded backoff and report
pending work if the wait expires. On `failed`, report the reason; on `superseded`,
read the current RFQ. If the admission response was lost, recover the quote through
the RFQ's `active_quote_id` and `quotes` before making another POST.
