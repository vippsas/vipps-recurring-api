<!-- START_METADATA
---
title: Recurring API V2 to V3 migration guide
sidebar_label: V2 to V3 migration guide
sidebar_position: 100
description: Recurring API V2 to V3 migration guide.
pagination_next: null
pagination_prev: null
---
END_METADATA -->

# Migration guide from V2 to V3

## What is new in V3

### Campaign

The Recurring API V3 adds new campaign types.
See [Campaigns](vipps-recurring-api.md#campaigns-in-v3-api) in the API Guide.

### Reserve capture

The Recurring API V3 adds the functionality to do reserve and capture on recurring charges.
See [Reserve capture](vipps-recurring-api.md#reserve-capture) in the API Guide.

### Partial capture

The Recurring API V3 adds the functionality to do partial capture on reserved charges.
See [partial capture](vipps-recurring-api.md#partial-capture) in the API Guide.

## What has changed in V3

## Response statuses

The API V3 returns different response status for some endpoints:

| Endpoint                                                                               | V2 response status | V3 response status                  |
|----------------------------------------------------------------------------------------|--------------------|-------------------------------------|
| [`PATCH:/agreements/{agreementId}`][update-agreement-endpoint]                         | `200 OK`           | `204 No Content` or `202 Accepted`* |
| [`DELETE:/agreements/{agreementId}/charges/{chargeId}`][cancel-charge-endpoint]        | `200 OK`           | `204 No Content` or `202 Accepted`* |
| [`POST:/agreements/{agreementId}/charges/{chargeId}/capture`][capture-charge-endpoint] | `200 OK`           | `204 No Content` or `202 Accepted`* |
| [`POST:/agreements/{agreementId}/charges/{chargeId}/refund`][refund-charge-endpoint]   | `200 OK`           | `204 No Content` or `202 Accepted`* |

`204 No Content` indicates that the request was successfully processed.

`202 Accepted` indicates also that the request was successful but the processing has not been completed yet.
The request is processed asynchronously and once it is completed, the agreement or charge will be updated.
Polling can be done to check the status. See [polling guidelines](vipps-recurring-api.md#polling-guidelines) in the API Guide.

**Please note:** Responses might include a `Retry-After`-header that will indicate the earliest time you should
retry the request or poll the resource to see if an operation has been performed.
This will follow the spec as defined [here](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Retry-After),
and will either be a http-date or a number indicating a delay in seconds.
This will mostly apply to 429 responses, but may also appear in certain other circumstances where it would be natural
to retry the request at a later point in time.

## Error responses

In V3, HTTP responses for errors follow the [RFC 7807](https://www.rfc-editor.org/rfc/rfc7807) standard.

See:
[Errors](https://developer.vippsmobilepay.com/docs/common-topics/errors).

## New price representation

The Recurring API V3 introduces a new JSON representation for agreement price.

There is no change on the charge limits rules. We will look into how we can implement charge limits in a better way, that takes care of the merchants needs.

### Fixed amount agreement
To draft an agreement with a fixed amount with the same charge limit as in V2, `pricing.type` should be set to `LEGACY`.
**Please note**: `pricing.type` is an optional field. If not provided in the request, `pricing.type` will be defaulted to `LEGACY`.

Truncated example of request body for the [`POST:/agreements`][draft-agreement-endpoint] endpoint from V2 and the equivalent in V3:

V2 request body

```json
{
  "currency": "NOK",
  "price": 100000,
  "productName": "MyNews Digital",
  "customerPhoneNumber": "45678272",
  "...": "..."
}
```

V3 request body

```json
{
  "pricing": {
    "type": "LEGACY",
    "amount": 100000,
    "currency": "NOK"
  },
  "productName": "MyNews Digital",
  "phoneNumber": "45678272",
  "...": "..."
}
```

### Variable amount

To draft agreement with a [variable amount](https://developer.vippsmobilepay.com/docs/APIs/recurring-api/vipps-recurring-api#recurring-agreements-with-variable-amount), `pricing.type` should be set to `VARIABLE`.

Truncated example of request body for the [`POST:/agreements`][draft-agreement-endpoint] endpoint from V2 and the equivalent in V3:

V2 request body

```json
{
  "currency": "NOK",
  "variableAmount": {
    "suggestedMaxAmount": 3000
  },
  "productName": "MyNews Digital",
  "customerPhoneNumber": "45678272",
  "...": "..."
}
```

V3 request body
```json
{
  "pricing": {
    "type": "VARIABLE",
    "suggestedMaxAmount": 3000,
    "currency": "NOK"
  },
  "productName": "MyNews Digital",
  "phoneNumber": "45678272",
  "...": "..."
}
```

## New interval representation

The Recurring API V3 introduces a new JSON representation for agreement interval.

Truncated example of request body for the [`POST:/agreements`][draft-agreement-endpoint] endpoint from V2 and the equivalent in V3:

V2 request body
```json
{
  "interval": "MONTH",
  "intervalCount": 1,
  "productName": "MyNews Digital",
  "phoneNumber": "45678272",
  "...": "..."
}
```

V3 request body

```json
{
  "interval": {
    "unit": "MONTH",
    "count": 1
  },
  "productName": "MyNews Digital",
  "phoneNumber": "45678272",
  "...": "..."
}
```

## More details on charges

In the API V3, the response from the [`GET:/agreements/{agreementId}/charges/{chargeId}`][fetch-charge-endpoint] endpoint
contains the history of the charge and not just the current status.
It also contains a summary of the total of amounts captured, refunded and cancelled.

Truncated example of the response from the [`GET:/agreements/{agreementId}/charges/{chargeId}`][fetch-charge-endpoint] endpoint:

````json
{
  "id": "chr_WCVbcA",
  "status": "REFUNDED",
  "amount": 1000,
  "type": "RECURRING",
  "transactionType": "RESERVE_CAPTURE",
  "...": "...",
  "summary": {
    "captured": 1000,
    "refunded": 600,
    "cancelled": 0
  },
  "history": [
    {
      "occurred": "2022-09-14T10:31:15Z",
      "event": "CREATE",
      "amount": 1000,
      "idempotencyKey": "e80bd8c6-3b83-4583-a49c-847021fcd839",
      "success": true
    },
    {
      "occurred": "2022-09-16T06:01:00Z",
      "event": "RESERVE",
      "amount": 1000,
      "idempotencyKey": "chr-4assY8f-agr_FJw2Anb-ProcessPayment",
      "success": true
    },
    {
      "occurred": "2022-09-18T06:01:00Z",
      "event": "CAPTURE",
      "amount": 1000,
      "idempotencyKey": "096b1415-2c77-4001-9576-531a856bbaf4",
      "success": true
    },
    {
      "occurred": "2022-09-20T06:01:00Z",
      "event": "REFUND",
      "amount": 600,
      "idempotencyKey": "0bc7cc3b-fdef-4d24-b4fe-49b7da40d22f",
      "success": true
    }
  ]
}
````

## Idempotency key

The misspelled `Idempotent-Key` header is deprecated.
The`Idempotency-Key` header is now required for the `POST` and `PATCH` endpoints.
See [Idempotency key header](https://developer.vippsmobilepay.com/docs/APIs/recurring-api/vipps-recurring-api#idempotency-key-header-v3-api-coming-soon) in the API Guide.

## Product description guidelines

We do not recommend you to use `Product Description` for agreements with a campaign.
We see that the user experience is not optimal when a lot of text is "squeezed" in the purple bubble displaying an agreement.
See [Campaigns in v3](vipps-recurring-api.md#campaigns-in-v3-api) in the API Guide.

## Update an agreement

In the API V3, it is possible to update the same fields on an agreement as in V2 except campaign. A campaigns can not be updated after creation.

| Field in V2          | Field in V3                 |
|----------------------|-----------------------------|
| productName          | productName                 |
| productDescription   | productDescription          |
| status               | status                      |
| merchantAgreementUrl | merchantAgreementUrl        |
| price                | pricing.amount              |
| suggestedMaxAmount   | pricing.suggestedMaxAmount  |
| campaign             | **Cannot be updated in V3** |

See the [`PATCH:/agreements/{agreementId}`][update-agreement-endpoint] endpoint.

Also, the API V3 returns different response status. It will return `204 No Content` or `202 Accepted`.
See [Response statuses](#response-statuses)


[draft-agreement-endpoint]: https://developer.vippsmobilepay.com/api/recurring#tag/Agreement-v3-endpoints/operation/DraftAgreementV3
[update-agreement-endpoint]: https://developer.vippsmobilepay.com/api/recurring#tag/Agreement-v3-endpoints/operation/UpdateAgreementPatchV3
[create-charge-endpoint]: https://developer.vippsmobilepay.com/api/recurring#tag/Charge-v3-endpoints/operation/CreateChargeV3
[cancel-charge-endpoint]: https://developer.vippsmobilepay.com/api/recurring#tag/Charge-v3-endpoints/operation/CancelChargeV3
[capture-charge-endpoint]: https://developer.vippsmobilepay.com/api/recurring#tag/Charge-v3-endpoints/operation/CaptureChargeV3
[refund-charge-endpoint]: https://developer.vippsmobilepay.com/api/recurring#tag/Charge-v3-endpoints/operation/RefundChargeV3
[fetch-charge-endpoint]: https://developer.vippsmobilepay.com/api/recurring#tag/Charge-v3-endpoints/operation/FetchChargeV3
