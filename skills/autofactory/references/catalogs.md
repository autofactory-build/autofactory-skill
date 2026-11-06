# Discover and choose, without guessing catalog IDs

Optional Bash/curl/jq examples; no scripts or SDK are installed. Load [authentication](authentication.md) first if its private directory, approved origin and bearer config are not already prepared. Confirm customer selections before any writes.

```bash
AF_STATUS=$(curl --disable --config "$AF_PRIVATE_DIR/api.curl" \
  --silent --show-error --fail-with-body --proto '=https' \
  --request GET "$AF_ORIGIN/v1/capabilities" \
  --dump-header "$AF_PRIVATE_DIR/capabilities.headers" \
  --output "$AF_PRIVATE_DIR/capabilities.json" \
  --write-out '%{http_code}') || exit 1
test "$AF_STATUS" = 200 || exit 1
AF_STATUS=$(curl --disable --config "$AF_PRIVATE_DIR/api.curl" \
  --silent --show-error --fail-with-body --proto '=https' \
  --request GET "$AF_ORIGIN/v1/materials?manufacturing_method=fdm_3d_printing&in_stock=true&page_number=1&page_size=100" \
  --dump-header "$AF_PRIVATE_DIR/materials.headers" \
  --output "$AF_PRIVATE_DIR/materials.json" \
  --write-out '%{http_code}') || exit 1
test "$AF_STATUS" = 200 || exit 1
AF_STATUS=$(curl --disable --config "$AF_PRIVATE_DIR/api.curl" \
  --silent --show-error --fail-with-body --proto '=https' \
  --request GET "$AF_ORIGIN/v1/profiles?manufacturing_method=fdm_3d_printing&status=active&page_number=1&page_size=100" \
  --dump-header "$AF_PRIVATE_DIR/profiles.headers" \
  --output "$AF_PRIVATE_DIR/profiles.json" \
  --write-out '%{http_code}') || exit 1
test "$AF_STATUS" = 200 || exit 1
```

Check `total_pages` and fetch remaining pages by incrementing `page_number`, saving
separate private files. Do not claim exhaustive selection from page 1 alone.
Read capability `limits`, `prohibitions`, `supported_file_formats` and method
availability. The initially proven flow uses STL and empty profile overrides.
Catalog presence does not guarantee quote admission for every material/profile.

Set `AF_MATERIAL_ID` and `AF_PROFILE_ID` from returned entries the customer selected;
set `AF_QUANTITY` to their positive integer quantity within the reported limit,
and `AF_COUNTRY`/`AF_POSTAL` to their quote destination. Do not use fabricated IDs.

```bash
: "${AF_MATERIAL_ID:?selected catalog material ID required}"
: "${AF_PROFILE_ID:?selected active profile ID required}"
: "${AF_QUANTITY:?confirmed integer quantity required}"
: "${AF_COUNTRY:?confirmed uppercase country code required}"
: "${AF_POSTAL:?confirmed postal code required}"
AF_STATUS=$(curl --disable --config "$AF_PRIVATE_DIR/api.curl" \
  --silent --show-error --fail-with-body --proto '=https' \
  --request GET "$AF_ORIGIN/v1/materials/$AF_MATERIAL_ID" \
  --dump-header "$AF_PRIVATE_DIR/material.headers" \
  --output "$AF_PRIVATE_DIR/material.json" \
  --write-out '%{http_code}') || exit 1
test "$AF_STATUS" = 200 || exit 1
AF_STATUS=$(curl --disable --config "$AF_PRIVATE_DIR/api.curl" \
  --silent --show-error --fail-with-body --proto '=https' \
  --request GET "$AF_ORIGIN/v1/profiles/$AF_PROFILE_ID" \
  --dump-header "$AF_PRIVATE_DIR/profile.headers" \
  --output "$AF_PRIVATE_DIR/profile.json" \
  --write-out '%{http_code}') || exit 1
test "$AF_STATUS" = 200 || exit 1
```

Verify method compatibility, material stock/archive state, active profile, limits
and customer intent. Do not infer arbitrary overrides from profile config text.
