# Upload models

Reserve an asset for each distinct model file. The same ready asset can be used
for multiple copies or different material/profile choices in an RFQ. Check
[catalog capabilities](catalogs.md) for formats and limits.

## Reserve an upload

This POST has no body or Content-Type. Expected status: 201.

```bash
curl --include --fail-with-body --request POST \
  'https://autofactory-api-staging.dowhilegeek.workers.dev/v1/assets' \
  --header 'Authorization: Bearer <api-key>'
```

Save `asset_id`, `method`, `url`, `headers` and `expires_at` from the response.

## Upload the file

Replace `<signed-upload-url>` with the returned URL and the path with your local
file. Use the returned method and all returned headers before `expires_at`.
The current response requires `If-None-Match: *`. Expected status: 200.

```bash
curl --include --fail-with-body --request PUT \
  '<signed-upload-url>' \
  --header 'If-None-Match: *' \
  --upload-file '/path/to/model.stl'
```

This signed request uses no API bearer token. An optional
`Content-Disposition: attachment; filename="model.stl"` supplies a display name.

## Wait for inspection

Read the asset (200):

```bash
curl --include --fail-with-body \
  'https://autofactory-api-staging.dowhilegeek.workers.dev/v1/assets/<asset-id>' \
  --header 'Authorization: Bearer <api-key>'
```

Poll this GET with bounded backoff until `ready` or `rejected`; honor Retry-After
when present. Inspection runs automatically, with no upload-completion endpoint.
On a lost PUT response, inspect asset status before deciding whether to upload
again. Use ready assets in the [RFQ](rfqs.md).
