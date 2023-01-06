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
For example, when calling [`PATCH:/agreements/{agreementId}`][update-agreement-endpoint] endpoint with a stopped agreement, 
the response will be the following:

```json
{
    "type": "https://vipps.no/problems/recurring/illegal-agreement-update", // TODO change
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

This documents the possible problems that can be returned by the API. 

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

### Invalid agreement id

### Invalid json

### Not authorized

### Idempotency key header

### Charge amount too high

### Charge amount too high for interval

### Charge due too soon

### Charge due in too long

### Unsupported feature

### Illegal agreement update

### Invalid suggested max amount

### Cancel charge failed

### Charge creation failed

### Charge capture failed

### Missing request header
