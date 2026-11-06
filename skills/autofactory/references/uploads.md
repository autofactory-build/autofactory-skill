# Upload a model

Use the [API access setup](authentication.md). Check [catalogs](catalogs.md) for
supported file formats and limits.

Set `AF_FILE` to the local STL path and check that it exists. Reservation takes no
body or Content-Type.

```bash
: "${AF_FILE:?local STL file required}"
test -f "$AF_FILE"
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

The optional filename is a display label. Use the returned upload URL before it
expires, with its required headers and without the API bearer token.
`If-None-Match: *` makes the upload create-only. After PUT, inspection runs
automatically; there is no upload-completion endpoint.

Repeat only the asset GET with bounded backoff until `state == "ready"`; stop on
`rejected` and inspect `rejection`. Honor Retry-After if present. On a lost PUT
response, read asset status before deciding whether another upload is appropriate.
Do not create an RFQ referencing `awaiting_upload` or `processing` assets.

```bash
jq -e '.state == "ready"' "$AF_PRIVATE_DIR/asset.json" >/dev/null
```
