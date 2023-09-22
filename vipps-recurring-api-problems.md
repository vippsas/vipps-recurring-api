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

# Problems

In Recurring API, HTTP responses for errors follow the [RFC 7807](https://www.rfc-editor.org/rfc/rfc7807) standard.

See [Errors](https://developer.vippsmobilepay.com/docs/common-topics/errors) for more details about the format.

API version: 3.0.0.

<!-- START_COMMENT -->

ℹ️ Please use the website:
[Vipps MobilePay Technical Documentation](https://developer.vippsmobilepay.com/docs/APIs/recurring-api).

<!-- END_COMMENT -->

## Problem types

The `type` is the primary identifier for a problem. It is a URL to a page with more details regarding the problem.
Each problem type returned by the Recurring API starts with `https://developer.vippsmobilepay.com/docs/APIs/recurring-api/vipps-recurring-api-problems#` followed by a problem identifier.

Example: `https://developer.vippsmobilepay.com/docs/APIs/recurring-api/vipps-recurring-api-problems#invalid-agreement-id`



### Identifiers
 | Identifier                          | Description                                                                                                                                    |
|-------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------|
| validation-error                    | Missing data or inconsistencies in JSON. Please verify the request JSON body. See [API specs](https://developer.vippsmobilepay.com/api/recurring). |
| invalid-agreement-id                | Agreement ID is invalid. Please verify the agreement ID is present and in the correct format. See [API specs](https://developer.vippsmobilepay.com/api/recurring). |
| invalid-json                        | JSON body is invalid. Please verify the request JSON body.                                                                                     |
| unsupported-media-type              | Media type indicated in Content-type header is not supported. See [API specs](https://developer.vippsmobilepay.com/api/recurring).             |
| not-authorized                      | JWT token is invalid.                                                                                                                           |
| idempotency-key-header              | Missing or invalid Idempotency key header. See [API specs](https://developer.vippsmobilepay.com/api/recurring).                               |
| charge-amount-too-high              | Amount set for charge is too high.                                                                                                             |
| charge-due-too-soon                 | Charge date due must be a minimum of 2 days in the future. See [Create a charge](https://developer.vippsmobilepay.com/docs/APIs/recurring-api/vipps-recurring-api#create-a-charge). |
| charge-due-in-too-long              | Charge due date must be a maximum of 2 years in the future. See [Create a charge](https://developer.vippsmobilepay.com/docs/APIs/recurring-api/vipps-recurring-api#create-a-charge).      |
| unsupported-feature                 | Feature not supported. Please [contact us](https://developer.vippsmobilepay.com/docs/contact/).                                                                                                  |
| illegal-agreement-update            | Agreement cannot be updated. Might be stopped or pending.                                                                                      |
| invalid-suggested-max-amount        | Suggested max amount is invalid or too high. See [Recurring agreements with variable amount](https://developer.vippsmobilepay.com/docs/APIs/recurring-api/vipps-recurring-api#recurring-agreements-with-variable-amount). |
| cancel-charge-failed                | Could not cancel the charge.                                                                                                                   |
| charge-creation-failed              | Could not create the charge.                                                                                                                   |
| charge-capture-failed               | Could not capture the charge.                                                                                                                  |
| missing-request-header              | Missing headers. See [API specs](https://developer.vippsmobilepay.com/api/recurring).                                                          |
| invalid-sale-unit                   | The Sales unit is not allowed to use Recurring. See [update-agreement-patch-endpoint](https://developer.vippsmobilepay.com/api/recurring#tag/Agreement-v3-endpoints/operation/UpdateAgreementPatchV3) |
| resource-not-found                  | The specified resource was not found. See details for additional information.                                                                |
| operation-failed                    | The requested operation could not be performed. See details for additional information.                                                      |
| conflict                            | The requested operation could not be performed due to a conflicting request/state. See details for additional information.                    |
| idempotency-conflict                | The specified `idempotency-key` has previously been used in a different context. See details for additional information.                        |
