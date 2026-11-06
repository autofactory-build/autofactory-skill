# Track an order

Use the [API access setup](authentication.md) and the quote/RFQ IDs from the
[checkout offer](checkout.md).

Set `AF_QUOTE_ID` and `AF_RFQ_ID` to the saved IDs. Read the quote and RFQ to check
payment; a redirect or an `accepted` quote alone does not confirm it.

```bash
AF_STATUS=$(curl --disable --config "$AF_PRIVATE_DIR/api.curl" \
  --silent --show-error --fail-with-body --proto '=https' \
  --request GET "$AF_ORIGIN/v1/quotes/$AF_QUOTE_ID" \
  --dump-header "$AF_PRIVATE_DIR/quote.headers" \
  --output "$AF_PRIVATE_DIR/quote.json" \
  --write-out '%{http_code}') || exit 1
test "$AF_STATUS" = 200 || exit 1
jq -e '.state == "paid" and (.order_id | type == "string")' \
  "$AF_PRIVATE_DIR/quote.json" >/dev/null
AF_ORDER_ID=$(jq -er '.order_id' "$AF_PRIVATE_DIR/quote.json")
AF_STATUS=$(curl --disable --config "$AF_PRIVATE_DIR/api.curl" \
  --silent --show-error --fail-with-body --proto '=https' \
  --request GET "$AF_ORIGIN/v1/rfqs/$AF_RFQ_ID" \
  --dump-header "$AF_PRIVATE_DIR/rfq-paid.headers" \
  --output "$AF_PRIVATE_DIR/rfq-paid.json" \
  --write-out '%{http_code}') || exit 1
test "$AF_STATUS" = 200 || exit 1
AF_STATUS=$(curl --disable --config "$AF_PRIVATE_DIR/api.curl" \
  --silent --show-error --fail-with-body --proto '=https' \
  --request GET "$AF_ORIGIN/v1/orders/$AF_ORDER_ID" \
  --dump-header "$AF_PRIVATE_DIR/order.headers" \
  --output "$AF_PRIVATE_DIR/order.json" \
  --write-out '%{http_code}') || exit 1
test "$AF_STATUS" = 200 || exit 1
```

Check that the RFQ is ordered and refers to the same paid quote and order.
Use the returned order state and any tracking details for updates. If tracking or
a delivery estimate is absent, say so.
