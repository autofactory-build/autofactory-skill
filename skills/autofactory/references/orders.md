# Track an order

Use the saved quote and RFQ IDs from the [checkout offer](checkout.md). Check the
quote (200):

```bash
curl --include --fail-with-body \
  'https://<api-host>/v1/quotes/<quote-id>' \
  --header 'Authorization: Bearer <api-key>'
```

When it is `paid` with an `order_id`, confirm the RFQ association (200):

```bash
curl --include --fail-with-body \
  'https://<api-host>/v1/rfqs/<rfq-id>' \
  --header 'Authorization: Bearer <api-key>'
```

Check that the RFQ is ordered and refers to the same paid quote and order.
Then read the returned order ID (200):

```bash
curl --include --fail-with-body \
  'https://<api-host>/v1/orders/<order-id>' \
  --header 'Authorization: Bearer <api-key>'
```

Use the returned order state and any tracking details for updates. A redirect or
an `accepted` quote alone does not confirm payment. If tracking or a delivery
estimate is absent, say so.
