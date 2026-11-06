# API access

Replace `<api-host>` with your AutoFactory API host and `<api-key>` with your token.
Other angle-bracket placeholders stand for values from your request or API
responses. The examples show HTTP status and response headers with `--include`;
`--fail-with-body` reports HTTP errors. Any HTTP client works.

## Get a key

Skip issuance if you already have a key. This POST takes no body or bearer token.
Expected status: 201. Save the returned `token` and key `id` with your usual tools.

```bash
curl --include --fail-with-body --request POST \
  'https://<api-host>/v1/api-keys'
```

New keys have `active: false`. Only AutoFactory operators can activate or
deactivate them. Send the key ID to AutoFactory for activation; a customer token
cannot change `active`, and there is no customer activation endpoint.

## Check access

After operator activation, this returns 200 with your key ID, client ID and rank.

```bash
curl --include --fail-with-body \
  'https://<api-host>/v1/me' \
  --header 'Authorization: Bearer <api-key>'
```

A 401 can mean pending activation, an invalid key or disabled access. Check the
endpoint and credential first, then contact AutoFactory if needed. Issuing another
key creates another client; it does not recover or activate the first key.
