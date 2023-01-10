<!-- START_METADATA
---
title: FAQ
sidebar_position: 90
pagination_next: null
---
END_METADATA -->

# Vipps Recurring API problems details

<!-- START_COMMENT -->

ℹ️ Please use the new documentation:
[Vipps Technical Documentation](https://vippsas.github.io/vipps-developer-docs/).

<!-- END_COMMENT -->

In Recurring api, HTTP responses for errors follow the [RFC 7807](https://www.rfc-editor.org/rfc/rfc7807) standard.
This document describes the possible problem types that can be returned by the API. 
For example, when calling [`PATCH:/agreements/{agreementId}`](update-agreement-endpoint) endpoint with a stopped agreement, 
the response will be the following:

```json
{
    "type": "https://vippsas.github.io/vipps-developer-docs/docs/APIs/recurring-api/vipps-recurring-api-problems#illegal-agreement-update",
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


API version: 3.0.0.

Document version 1.0.0.

<!-- START_TOC -->

## Table of Contents

- [Vipps Recurring API](#vipps-recurring-api-problems)
  - [Table of Contents](#table-of-contents)
  - [Problems](#problems)
    - [Validation error](#validation-error)
    - [Invalid agreement id](#invalid-agreement-id)
    - [Invalid json](#invalid-json)
    - [Unsupported media type](#unsupported-media-type)
    - [Not authorized](#not-authorized)
    - [Idempotency key header](#idempotency-key-header)
    - [Charge amount too high](#charge-amount-too-high)
    - [Charge amount too high for interval](#charge-amount-too-high-for-interval)
    - [Charge due too soon](#charge-due-too-soon)
    - [Charge due in too long](#charge-due-in-too-long)
    - [Unsupported feature](#unsupported-feature)
    - [Illegal agreement update](#illegal-agreement-update)
    - [Invalid suggested max amount](#invalid-suggested-max-amount)
    - [Cancel charge failed](#cancel-charge-failed)
    - [Charge creation failed](#charge-creation-failed)
    - [Charge capture failed](#charge-capture-failed)
    - [Missing request header](#missing-request-header)


<!-- END_TOC -->

## Problems

### Validation error
Missing data or inconsistencies in json. Please verify the request json body.
See [API specs](https://vippsas.github.io/vipps-developer-docs/api/recurring).

### Invalid agreement id
Agreement Id is invalid. Please verify the agreement Id is present and in the correct format. 
See [API specs](https://vippsas.github.io/vipps-developer-docs/api/recurring).

### Invalid json
Json body is invalid. Please verify the request json body.

### Unsupported media type
Media type indicated in Content-type header is not supported.
See [API specs](https://vippsas.github.io/vipps-developer-docs/api/recurring).

### Not authorized
JWT token is invalid.

### Idempotency key header
Missing or invalid Idempotency key header.
See [API specs](https://vippsas.github.io/vipps-developer-docs/api/recurring).

### Charge amount too high
Amount set for charge is too high.

### Charge amount too high for interval
Amount for interval is exceeded. Check if other charges has already been created for this interval. 

### Charge due too soon
Charge date due must be minimum 2 days in the future.
See [Create a charge](https://vippsas.github.io/vipps-developer-docs/docs/APIs/recurring-api/vipps-recurring-api#create-a-charge).

### Charge due in too long
Charge due date must be maximum 2 years in the future.
See [Create a charge](https://vippsas.github.io/vipps-developer-docs/docs/APIs/recurring-api/vipps-recurring-api#create-a-charge).

### Unsupported feature
Feature not supported. Please contact Vipps.

### Illegal agreement update
Agreement cannot be updated. Might be still stopped or pending.

### Invalid suggested max amount
Suggested max amount is invalid or too high. 
See [Recurring agreements with variable amount](https://vippsas.github.io/vipps-developer-docs/docs/APIs/recurring-api/vipps-recurring-api#recurring-agreements-with-variable-amount).

### Cancel charge failed
Could not cancel charge.

### Charge creation failed
Could not create charge.

### Charge capture failed
Could not capture charge.

### Missing request header
Missing headers. See [API specs](https://vippsas.github.io/vipps-developer-docs/api/recurring).
