# Materials and print profiles

Choose from the current catalog using the request, preferences and suitable
defaults. See [API access](authentication.md) for the placeholder convention.

These reads return 200:

```bash
curl --include --fail-with-body \
  'https://<api-host>/v1/capabilities' \
  --header 'Authorization: Bearer <api-key>'
```

```bash
curl --include --fail-with-body \
  'https://<api-host>/v1/materials?manufacturing_method=fdm_3d_printing&in_stock=true&page_number=1&page_size=100' \
  --header 'Authorization: Bearer <api-key>'
```

```bash
curl --include --fail-with-body \
  'https://<api-host>/v1/profiles?manufacturing_method=fdm_3d_printing&status=active&page_number=1&page_size=100' \
  --header 'Authorization: Bearer <api-key>'
```

Follow `total_pages` by increasing `page_number`. Check capability `limits`,
`prohibitions`, `supported_file_formats` and method availability. The documented
flow uses STL and empty profile overrides.

Read the selected material and profile details (200):

```bash
curl --include --fail-with-body \
  'https://<api-host>/v1/materials/<material-id>' \
  --header 'Authorization: Bearer <api-key>'
```

```bash
curl --include --fail-with-body \
  'https://<api-host>/v1/profiles/<profile-id>' \
  --header 'Authorization: Bearer <api-key>'
```

Check method compatibility, stock, active profile status and limits. Include your
choices in the quote. Each RFQ item can use its own material, profile and quantity.
