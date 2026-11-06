# API access

Use your configured AutoFactory HTTPS endpoint and existing key. Manage credentials
with your usual tools. The examples below use a private curl config; adapt them to
your environment or use another HTTP client.

For these examples, set `AF_ORIGIN` to the API origin without a trailing slash,
`AF_PRIVATE_DIR` to an absolute directory for credentials and responses, and
`AF_FILE` to the local STL path. Only uploads need `AF_FILE`. Store an existing key as `key.json` with `token` and `id` fields.

```bash
set +x
set -euo pipefail
umask 077
: "${AF_ORIGIN:?AutoFactory HTTPS origin required}"
: "${AF_PRIVATE_DIR:?private absolute directory required}"
case "$AF_ORIGIN" in https://?*) ;; *) exit 1 ;; esac
AF_AUTHORITY=${AF_ORIGIN#https://}
case "$AF_AUTHORITY" in *[/?#@]*|'') exit 1 ;; esac
case "$AF_PRIVATE_DIR" in /*) ;; *) exit 1 ;; esac
mkdir -p "$AF_PRIVATE_DIR"
chmod 700 "$AF_PRIVATE_DIR"
```

These examples need curl with `--fail-with-body`. Each request checks the expected
HTTP status and saves its response. They do not follow redirects. Keep the bearer
config for API calls; upload requests use their own config.

## Get a key

Skip this step if you already have a key. `POST /v1/api-keys` takes no body or
Authorization header. It returns the token once, so save the response.

```bash
test ! -e "$AF_PRIVATE_DIR/key.json"
AF_STATUS=$(curl --disable --silent --show-error --fail-with-body --proto '=https' \
  --request POST "$AF_ORIGIN/v1/api-keys" \
  --dump-header "$AF_PRIVATE_DIR/issuance.headers" \
  --output "$AF_PRIVATE_DIR/key.json" --write-out '%{http_code}') || exit 1
test "$AF_STATUS" = 201 || exit 1
jq -e '.active == false and (.token | type == "string")' \
  "$AF_PRIVATE_DIR/key.json" >/dev/null
jq -r '.id' "$AF_PRIVATE_DIR/key.json"
```

New keys need activation. Send the returned key ID to AutoFactory and resume after
activation. Another issuance creates another client; it does not recover or
activate the first key.

## Check access

```bash
jq -er '"header = " + (("Authorization: Bearer " + .token) | @json)' \
  "$AF_PRIVATE_DIR/key.json" > "$AF_PRIVATE_DIR/api.curl"

AF_STATUS=$(curl --disable --config "$AF_PRIVATE_DIR/api.curl" \
  --silent --show-error --fail-with-body --proto '=https' \
  --request GET "$AF_ORIGIN/v1/me" \
  --dump-header "$AF_PRIVATE_DIR/me.headers" \
  --output "$AF_PRIVATE_DIR/me.json" \
  --write-out '%{http_code}') || exit 1
test "$AF_STATUS" = 200 || exit 1
jq -e '.id and .api_client_id and .rank' "$AF_PRIVATE_DIR/me.json" >/dev/null
```

A 401 can mean pending activation, an invalid key or disabled access. Check the
configured endpoint and credential first; contact AutoFactory if activation or
access needs attention. See [troubleshooting](troubleshooting.md) for recovery.
