# Explicit environment and private storage

Optional Bash/curl/jq examples; no scripts or SDK are installed. Read only this task and any needed prerequisites. Never run the full flow unattended.

The operator must set `AF_ORIGIN` to the exact approved HTTPS origin (no path,
query, fragment, credentials or trailing slash), `AF_PRIVATE_DIR` to a new private
absolute directory outside any repository/shared workspace, and `AF_FILE` to the
customer-authorized STL path. There is no default production origin. A contract
hostname or a staging deployment does not establish live-payment availability.

Disable command tracing, verbose HTTP logs, response capture in shared tools and
link unfurling. Never paste the key into a command or chat. Existing approved
credentials should be supplied through private storage as `key.json`, containing
`token` and the credential `id`; never overwrite an existing credential file.

```bash
set +x
set -euo pipefail
umask 077
: "${AF_ORIGIN:?operator-provided HTTPS origin required}"
: "${AF_PRIVATE_DIR:?private absolute directory required}"
: "${AF_FILE:?authorized local STL file required}"
case "$AF_ORIGIN" in https://?*) ;; *) exit 1 ;; esac
AF_AUTHORITY=${AF_ORIGIN#https://}
case "$AF_AUTHORITY" in *[/?#@]*|'') exit 1 ;; esac
case "$AF_PRIVATE_DIR" in /*) ;; *) exit 1 ;; esac
mkdir -p "$AF_PRIVATE_DIR"
chmod 700 "$AF_PRIVATE_DIR"
test -f "$AF_FILE"
```

Verify the hostname with the operator, not merely the scheme check above.
Do not use `curl -L`, `--location-trusted`, `-v` or shell tracing with credentials.
Use a curl build supporting `--fail-with-body`. Each invocation disables default
curl config loading, saves response headers/body privately and requires the exact
expected status. Never add redirect-following options. A transport/HTTP failure
or unexpected status stops the task; consult [troubleshooting](troubleshooting.md)
before retrying, especially after writes. Keep these shell settings and private
configuration when running a task-specific example.

### Optional new-client onboarding — only once, with authorization

Skip this block when an approved key already exists. No Authorization header,
body, Content-Type or query parameters are sent. Save the full response privately;
only the key ID is suitable to share with the operator. A timeout can still mean
issuance succeeded: do not automatically repeat it.

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

Store the token in the agent's secret store once. There is no token recovery
endpoint. Each issuance creates a different client, not rotation or recovery.
**Stop here until the operator approves the returned key ID.** Do not self-activate.

### Prepare an authenticated client after approval

This private curl config avoids placing the bearer token directly in command
arguments. Do not display the file. Never use it for an upload or checkout link.

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

401 cannot distinguish pending approval from invalid/revoked/disabled access.
Stop and ask the operator; do not run onward or create replacement clients.
`--fail-with-body` fails on HTTP errors, but not 3xx: inspect saved HTTP status
headers and reject unexpected redirects before accepting any response.
