<!-- START_METADATA
---
title: V2 to V3 migration guide
sidebar_position: 100
---
END_METADATA -->

# Vipps Recurring API V2

V2 API is now deprecated. 
Please check the [migration guide](#migration-guide) to see the differences between Recurring API v2 and v3

## Migration guide

### Campaign
The Recurring API V3 adds new campaign types.
See [Campaigns][campaigns] in the API Guide

### Reserve capture
The Recurring API V3 adds the functionality to do reserve and capture on recurring charges. 
(Only available on initial charge in the Recurring API V2)
See [Reserve capture][reserve-capture] in the API Guide

### Partial capture
The Recurring API V3 adds the functionality to do partial capture on reserved charges.
See [partial capture][partial-capture] in the API Guide

### Response statuses

The API V3 returns different response status for some endpoints:

| Endpoint                                        | V2 response status | V3 response status |
|-------------------------------------------------|--------------------|--------------------|
| [`update agreement`][update-agreement-endpoint] | `200 OK`           | `204 No Content`   | 
| [`cancel charge`][cancel-charge-endpoint]       | `200 OK`           | `204 No Content`   | 
| [`capture charge`][capture-charge-endpoint]     | `200 OK`           | `204 No Content`   | 
| [`refund charge`][refund-charge-endpoint]       | `200 OK`           | `204 No Content`   | 

### Error responses

In V3, HTTP responses for errors follow the [RFC 7807](https://www.rfc-editor.org/rfc/rfc7807) standard.
For example, if the agreement is not active the [`update agreement`][update-agreement-endpoint] endpoint will return:

```json
{
    "type": "https://vipps.no/problems/recurring/illegal-agreement-update",
    "title": "Bad Request",
    "status": 400,
    "detail": "Illegal update",
    "instance": "/vipps-recurring-merchant-api/v3/agreements/agr_nmgWS4e",
    "contextId": "ef087f56-4281-494d-9591-5e4cf6fe05b5",
    "extraDetails": [
        {
            "status": "Cannot modify an agreement which is not active."
        }
    ]
}
```

### New price representation 

The Recurring API V3 introduces a new JSON representation for agreement price.

There is no change on the charge limits rules. Going forward, we will look into how we can implement charge limits in a better way, that takes care of the merchants needs.

To draft an agreement with a fixed amount with the same charge limit as in V2, `pricing.type` should be set to `LEGACY`.

**Note**: `pricing.type` is an optional field. If not provided in the request the type will be defaulted to `LEGACY`.

Truncated example of request body for the [draft agreement]() endpoint from V2 and the equivalent in V3:

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
    "amount": 180000,
    "currency": "NOK"
  },
  "productName": "MyNews Digital",
  "customerPhoneNumber": "45678272",
  "...": "..."
}
```

To draft agreement with a [variable amount](https://vippsas.github.io/vipps-developer-docs/docs/APIs/recurring-api/vipps-recurring-api#recurring-agreements-with-variable-amount), `pricing.type` should be set to `VARIABLE`

Truncated example of request body for the [draft agreement]() endpoint from V2 and the equivalent in V3:

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
    "suggestedMaxAmount": 180000,
    "currency": "NOK"
  },
  "productName": "MyNews Digital",
  "customerPhoneNumber": "45678272",
  "...": "..."
}
```

### New interval representation
The Recurring API V3 introduces a new JSON representation for agreement interval.

Truncated example of request body for the [draft agreement]() endpoint from V2 and the equivalent in V3:

V2 request body
```json
{
  "interval": "MONTH",
  "intervalCount": 1,
  "productName": "MyNews Digital",
  "customerPhoneNumber": "45678272",
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
  "customerPhoneNumber": "45678272",
  "...": "..."
}
```

### More details on Charges




### Idempotency key

`Idempotency-Key` is now required for the following endpoints:
- [`draft agreement`][draft-agreement-endpoint]
- [`create charge`][create-charge-endpoint]
- [`capture charge`][capture-charge-endpoint]
- [`refund charge`][refund-charge-endpoint] 


### Product description guidelines
We do not recommend you to use `Product Description` for agreements with a campaign.
We see that the user experience is not optimal when a lot of text is "squeezed" in the purple bubble displaying an agreement.
`Product description` will be, at a point in the future, phased out in order to improve user experience.


[draft-agreement-endpoint]: https://vippsas.github.io/vipps-developer-docs/api/recurring/#tag/Agreement-v3-endpoints/operation/DraftAgreementV3
[update-agreement-endpoint]: https://vippsas.github.io/vipps-developer-docs/api/recurring/#tag/Agreement-v3-endpoints/operation/UpdateAgreementPatchV3
[create-charge-endpoint]: https://vippsas.github.io/vipps-developer-docs/api/recurring/#tag/Charge-v3-endpoints/operation/CreateChargeV3
[cancel-charge-endpoint]: https://vippsas.github.io/vipps-developer-docs/api/recurring#tag/Charge-v3-endpoints/operation/CancelChargeV3
[capture-charge-endpoint]: https://vippsas.github.io/vipps-developer-docs/api/recurring#tag/Charge-v3-endpoints/operation/CaptureChargeV3
[refund-charge-endpoint]: https://vippsas.github.io/vipps-developer-docs/api/recurring#tag/Charge-v3-endpoints/operation/RefundChargeV3

[partial-capture]: https://vippsas.github.io/vipps-developer-docs/docs/APIs/recurring-api/vipps-recurring-api/#partial-capture-coming-soon
[reserve-capture]: #todo-add-link
[campaigns]: #todo-add-link



