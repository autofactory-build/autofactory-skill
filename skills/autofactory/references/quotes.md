# Get a quote

Use an [RFQ](rfqs.md) whose `allowed_actions` includes `create_quote`.
This POST takes no body or Content-Type. Expected status: 202.

```bash
curl --include --fail-with-body --request POST \
  'https://autofactory-api-staging.dowhilegeek.workers.dev/v1/rfqs/<rfq-id>/quote' \
  --header 'Authorization: Bearer <api-key>'
```

Save the quote `id`, then read it (200):

```bash
curl --include --fail-with-body \
  'https://autofactory-api-staging.dowhilegeek.workers.dev/v1/quotes/<quote-id>' \
  --header 'Authorization: Bearer <api-key>'
```

Poll the same quote with GET while `calculating`. Honor Retry-After and use bounded
backoff. Report pending work if the wait expires. On `failed`, report the reason;
on `superseded`, read the current RFQ. If the admission response was lost, recover
through the RFQ's `active_quote_id` and `quotes` before making another POST.

When `ready`, [send the offer](checkout.md) with all items and quantities.
