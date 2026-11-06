# Materials and print profiles

Use the [API access setup](authentication.md) for these curl examples. Choose from
the current catalog using the request, stated preferences and suitable defaults.

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
availability. The documented flow uses STL and empty profile overrides.
Catalog presence does not guarantee quote admission for every material/profile.

Set `AF_MATERIAL_ID` and `AF_PROFILE_ID` from the selected catalog entries,
`AF_QUANTITY` to the requested positive integer quantity, and `AF_COUNTRY`/`AF_POSTAL`
to the destination. Use existing context; ask for destination details if missing.

```bash
: "${AF_MATERIAL_ID:?selected catalog material ID required}"
: "${AF_PROFILE_ID:?selected active profile ID required}"
: "${AF_QUANTITY:?integer quantity required}"
: "${AF_COUNTRY:?uppercase country code required}"
: "${AF_POSTAL:?postal code required}"
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

Check method compatibility, stock, active profile status and limits. Include your
choices in the quote. Profile config text is not an override schema.
