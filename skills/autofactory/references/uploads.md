# Reserve, PUT raw bytes, then await inspection

Optional Bash/curl/jq examples; no scripts or SDK are installed. Load [authentication](authentication.md) first if its private directory, approved origin and bearer config are not already prepared. Load [catalogs](catalogs.md) if supported input limits/formats are not known.

Use the customer-authorized local STL path in `AF_FILE`, already checked in
authentication setup. Reservation is bodyless and sends no Content-Type.

```bash
AF_STATUS=$(curl --disable --config "$AF_PRIVATE_DIR/api.curl" \
  --silent --show-error --fail-with-body --proto '=https' \
  --request POST "$AF_ORIGIN/v1/assets" \
  --dump-header "$AF_PRIVATE_DIR/upload.headers" \
  --output "$AF_PRIVATE_DIR/upload.json" \
  --write-out '%{http_code}') || exit 1
test "$AF_STATUS" = 201 || exit 1
AF_ASSET_ID=$(jq -er '.asset_id' "$AF_PRIVATE_DIR/upload.json")
jq -e '.method == "PUT" and (.url | startswith("https://"))' \
  "$AF_PRIVATE_DIR/upload.json" >/dev/null

# Each required upload header and the signed URL stay in a private config.
jq -er '(["url = " + (.url | @json)] +
  [.headers | to_entries[] | "header = " + ((.key + ": " + .value) | @json)])
  | join("\n")' "$AF_PRIVATE_DIR/upload.json" > "$AF_PRIVATE_DIR/upload.curl"

# Separate curl invocation: NO API bearer config or Authorization header.
AF_STATUS=$(curl --disable --config "$AF_PRIVATE_DIR/upload.curl" \
  --silent --show-error --fail-with-body --proto '=https' \
  --stderr "$AF_PRIVATE_DIR/put.errors" \
  --request PUT --upload-file "$AF_FILE" \
  --header 'Content-Disposition: attachment; filename="design.stl"' \
  --dump-header "$AF_PRIVATE_DIR/put.headers" \
  --output "$AF_PRIVATE_DIR/put.response" --write-out '%{http_code}') || exit 1
test "$AF_STATUS" = 200 || exit 1

AF_STATUS=$(curl --disable --config "$AF_PRIVATE_DIR/api.curl" \
  --silent --show-error --fail-with-body --proto '=https' \
  --request GET "$AF_ORIGIN/v1/assets/$AF_ASSET_ID" \
  --dump-header "$AF_PRIVATE_DIR/asset.headers" \
  --output "$AF_PRIVATE_DIR/asset.json" \
  --write-out '%{http_code}') || exit 1
test "$AF_STATUS" = 200 || exit 1
jq '{id,state,rejection,byte_size,file_format,dimensions_mm}' \
  "$AF_PRIVATE_DIR/asset.json"
```

The filename above is an optional generic display label, not a format assertion.
Validate the returned target as HTTPS, use its expiry and required headers exactly,
and never attach general API credentials. Current required headers include
`If-None-Match: *`; do not overwrite an existing object. A successful PUT is not
asset readiness. There is **no separate complete-upload endpoint**: scheduled
inspection finds the uploaded object.

Repeat only the asset GET with bounded backoff until `state == "ready"`; stop on
`rejected` and inspect `rejection`. Honor Retry-After if present. On a lost PUT
response, read asset status before deciding whether another upload is appropriate.
Do not create an RFQ referencing `awaiting_upload` or `processing` assets.

```bash
jq -e '.state == "ready"' "$AF_PRIVATE_DIR/asset.json" >/dev/null
```
