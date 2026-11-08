# Create a manufacturing request

An RFQ has an `items` array. Add an item for each model/material/profile choice
and set its quantity to the number of copies. Upload each distinct file first;
reuse its ready asset ID wherever that model appears. Stay within capability
limits for item count and quantity.

This example requests ten copies of one model and three of another. Replace the
IDs with values from [uploads](uploads.md) and the [catalog](catalogs.md), and
replace the example quantities and destination with the request's values.
Expected status: 201.

```bash
curl --include --fail-with-body --request POST \
  'https://autofactory-api-staging.dowhilegeek.workers.dev/v1/rfqs' \
  --header 'Authorization: Bearer <api-key>' \
  --header 'Content-Type: application/json' \
  --data '{
    "items": [
      {
        "manufacturing_method": "fdm_3d_printing",
        "asset_id": "<first-asset-id>",
        "material_id": "<first-material-id>",
        "quantity": 10,
        "configuration": {"profile_id": "<first-profile-id>", "overrides": {}}
      },
      {
        "manufacturing_method": "fdm_3d_printing",
        "asset_id": "<second-asset-id>",
        "material_id": "<second-material-id>",
        "quantity": 3,
        "configuration": {"profile_id": "<second-profile-id>", "overrides": {}}
      }
    ],
    "destination": {"country_code": "US", "postal_code": "95814"}
  }'
```

Read the returned RFQ `id` (200):

```bash
curl --include --fail-with-body \
  'https://autofactory-api-staging.dowhilegeek.workers.dev/v1/rfqs/<rfq-id>' \
  --header 'Authorization: Bearer <api-key>'
```

Check the items, destination and revision. When `missing_fields` is empty and
`allowed_actions` includes `create_quote`, request a [quote](quotes.md).
RFQ edits create a new revision and supersede earlier offers; keep those offers
as history.
