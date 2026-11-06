# Create a manufacturing request

Use the [API access setup](authentication.md), a ready [asset](uploads.md), and
material/profile selections from the [catalog](catalogs.md).

Use the saved `AF_ASSET_ID`, selected `AF_MATERIAL_ID` and `AF_PROFILE_ID`, integer
`AF_QUANTITY`, and destination `AF_COUNTRY`/`AF_POSTAL`.

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

Check the returned items, destination and revision against the request. RFQ edits
create a new revision and supersede earlier quotes. Update the RFQ when the request
changes; keep earlier offers as history.
