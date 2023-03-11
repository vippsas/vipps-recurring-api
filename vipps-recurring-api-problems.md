<!-- START_METADATA
---
title: Problem types
sidebar_label: Problem types
sidebar_position: 90
description: Problem types for the Recurring API can be found here.
pagination_next: null
pagination_prev: null
---
END_METADATA -->

# Problem types

In Recurring API, HTTP responses for errors follow the [RFC 7807](https://www.rfc-editor.org/rfc/rfc7807) standard.

See:
[Errors](https://vippsas.github.io/vipps-developer-docs/docs/vipps-developers/common-topics/errors)

API version: 3.0.0.

<!-- START_COMMENT -->

ℹ️ Please use the website:
[Vipps MobilePay Technical Documentation](https://vippsas.github.io/vipps-developer-docs/docs/APIs/recurring-api).

<!-- END_COMMENT -->

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

Agreement cannot be updated. Might be stopped or pending.

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

### Invalid sales unit
The Sales unit is not allowed to use Recurring.

[update-agreement-patch-endpoint]: https://vippsas.github.io/vipps-developer-docs/api/recurring#tag/Agreement-v3-endpoints/operation/UpdateAgreementPatchV3

### Resource not found
The specified resource was not found. See details for additional information.

### Operation failed
The requested operation could not be performed. See details for additional information.

### Conflict
The requested operation could not be performed due to a conflicting request/state. See details for additional information.

### Idempotency conflict
The specified idempotency-key has previously been used in a different context. See details for additional information.

### Invalid method
The method used for the request is not permitted for this endpoint.