# Submit the exact RFQ body and read it back

Optional Bash/curl/jq examples; no scripts or SDK are installed. Load [authentication](authentication.md) first if its private directory, approved origin and bearer config are not already prepared. Requires a ready asset from [uploads](uploads.md) and customer-confirmed selections from [catalogs](catalogs.md).

Provide the previously verified ready `AF_ASSET_ID`, customer-selected
`AF_MATERIAL_ID` and `AF_PROFILE_ID`, confirmed integer `AF_QUANTITY`, and
destination `AF_COUNTRY`/`AF_POSTAL`. These are inputs, not examples to invent.

```bash
jq -n --arg asset "$AF_ASSET_ID" --arg material "$AF_MATERIAL_ID" \
  --arg profile "$AF_PROFILE_ID" --argjson quantity "$AF_QUANTITY" \
  --arg country "$AF_COUNTRY" --arg postal "$AF_POSTAL" \
  '{items:[{manufacturing_method:"fdm_3d_printing",asset_id:$asset,
    material_id:$material,quantity:$quantity,
    configuration:{profile_id:$profile,overrides:{}}}],
    destination:{country_code:$country,postal_code:$postal}}' \
  > "$AF_PRIVATE_DIR/rfq-request.json"
AF_STATUS=$(curl --disable --config "$AF_PRIVATE_DIR/api.curl" \
  --silent --show-error --fail-with-body --proto '=https' \
  --request POST "$AF_ORIGIN/v1/rfqs" \
  --dump-header "$AF_PRIVATE_DIR/rfq.headers" \
  --output "$AF_PRIVATE_DIR/rfq.json" \
  --header 'Content-Type: application/json' \
  --data-binary "@$AF_PRIVATE_DIR/rfq-request.json" \
  --write-out '%{http_code}') || exit 1
test "$AF_STATUS" = 201 || exit 1
AF_RFQ_ID=$(jq -er '.id' "$AF_PRIVATE_DIR/rfq.json")
AF_STATUS=$(curl --disable --config "$AF_PRIVATE_DIR/api.curl" \
  --silent --show-error --fail-with-body --proto '=https' \
  --request GET "$AF_ORIGIN/v1/rfqs/$AF_RFQ_ID" \
  --dump-header "$AF_PRIVATE_DIR/rfq-readback.headers" \
  --output "$AF_PRIVATE_DIR/rfq-readback.json" \
  --write-out '%{http_code}') || exit 1
test "$AF_STATUS" = 200 || exit 1
jq -e '.missing_fields == [] and
  (.allowed_actions | index("create_quote") != null)' \
  "$AF_PRIVATE_DIR/rfq-readback.json" >/dev/null
```

Verify all returned item specifications, destination and revision against the
customer's request. An RFQ is mutable, not a paid order. Changes require renewed
customer confirmation and supersede earlier quotes; do not mutate old prices.
