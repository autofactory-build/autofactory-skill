# Verify confirmed payment, then read the associated order

Optional Bash/curl/jq examples; no scripts or SDK are installed. Load [authentication](authentication.md) first if its private directory, approved origin and bearer config are not already prepared. Requires an actual human [checkout](checkout.md) attempt, not authorization to pay.

Set `AF_QUOTE_ID` and `AF_RFQ_ID` to the exact quote and RFQ reviewed by the
human. Do not guess IDs or create replacement resources in this task.

Only after the human's checkout attempt, re-read the same quote and RFQ. Do not
infer payment from a browser redirect or an `accepted` state.

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

Verify the RFQ's ordered state and the same paid quote/order association before
claiming an order. Keep order/contact/shipping JSON private. Report only fields
actually returned; no tracking or delivery guarantees are implied. The skill
neither pays nor contacts a printer or the payment provider directly.
