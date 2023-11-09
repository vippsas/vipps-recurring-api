<!-- START_METADATA
---
title: Recurring API checklist
sidebar_label: Checklist
sidebar_position: 40
description: Checklist for full integration with the Recurring API.
pagination_next: null
pagination_prev: null
---
END_METADATA -->

# Checklist

<!-- START_COMMENT -->

ℹ️ Please use the website:
[Vipps MobilePay Technical Documentation](https://developer.vippsmobilepay.com/docs/APIs/recurring-api).

<!-- END_COMMENT -->

API version: 3.0

## Endpoints to integrate

Integrate _all_ the [API endpoints](https://developer.vippsmobilepay.com/api/recurring/).
For examples of requests and responses, see the [Postman collection](/tools/recurring-v3-api-postman-collection.json) and [environment](https://developer.vippsmobilepay.com/internal-tools/global-postman-environment.json).

| Endpoint | Comment |
|----------|---------|
| Agreement: Create an agreement | [`POST:/recurring/v3/agreements`][draft-agreement-endpoint] |
| Agreement: Retrieve an agreement | [`GET:/recurring/v3/agreements/{agreementId}`][fetch-agreement-endpoint] |
| Agreement: Update an agreement | [`PATCH:/recurring/v3/agreements/{agreementId}`][update-agreement-patch-endpoint] |
| Agreement: List charges | [`GET:/recurring/v3/agreements/{agreementId}/charges`][list-charges-endpoint] |
| Charge: Create Charge | [`POST:/recurring/v3/agreements/{agreementId}/charges`][create-charge-endpoint] |
| Charge: Retrieve a charge | [`GET:/recurring/v3/agreements/{agreementId}/charges/{chargeId}`][fetch-charge-endpoint] |
| Charge: Cancel a charge | [`DELETE:/recurring/v3/agreements/{agreementId}/charges/{chargeId}`][cancel-charge-endpoint] |
| Charge: Refund a charge | [`POST:/recurring/v3/agreements/{agreementId}/charges/{chargeId}/refund`][refund-charge-endpoint] |
| Charge: Capture a reserved charge | [`POST:/recurring/v3/agreements/{agreementId}/charges/{chargeId}/capture`][capture-charge-endpoint] |

When the checklist is completed, notify Vipps MobilePay Integration Service
at [developer@vippsmobilepay.com](mailto:developer@vippsmobilepay.com).
Include examples from the test environment, pilot customer info, and a description of the implemented solution.

We will verify the integration and contact you.
After the checklist is approved, we'll send you the information you need to get started.

## Quality assurance

| Action | Comment |
|--------|---------|
|     Handle errors | Monitor the error messages Vipps MobilePay returns from the APIs and correct problems as quickly as possible. It's recommended to log errors emitted by our APIs. This makes it easier to pinpoint what has gone wrong. |
|     Include standard HTTP headers | Send the [HTTP headers](https://developer.vippsmobilepay.com/docs/knowledge-base/http-headers) in all API requests for better tracking and troubleshooting (mandatory for partners and platforms, who must send these headers as part of the checklist approval). |

## Avoid integration pitfalls

| Action | Comment   |
|--------|-----------|
| Poll for the agreement status | The Merchant _must not_ rely on `Redirect` alone, but must always poll [`GET:/recurring/v3/agreements/{agreementId}`][fetch-agreement-endpoint] to check the status of the agreement. |
| Provide subscription tools | The merchant must have a way for the user to manage and stop subscription in `merchantAgreementUrl` in their agreement. This should result in a timely update of the Vipps MobilePay Agreement. |
|     Follow design guidelines| The Vipps MobilePay branding must be according to the [design guidelines](https://developer.vippsmobilepay.com/docs/design-guidelines). |
|     Educate customer support| Make sure your customer service, etc. has all the tools and information they need available in _your_ system, through the APIs listed in the first item in this checklist, and that they do not need to visit [portal.vipps.no](https://portal.vipps.no) for normal work. |

## Flow to go live for direct integrations

1. The merchant orders [Vipps på Nett](https://www.vipps.no/produkter-og-tjenester/bedrift/ta-betalt-paa-nett/ta-betalt-paa-nett/).
2. Vipps MobilePay completes customer control (KYC, PEP, AML, etc.).
3. The merchant receives an email from Vipps MobilePay saying that they can log in with BankID on [portal.vipps.no](https://portal.vipps.no) and retrieve API keys.
4. The merchant completes all checklist items.
5. The merchant verifies the integration in the test environment by checking that
   there are test agreements and charges in the
   [test environment](https://developer.vippsmobilepay.com/docs/test-environment),
   with the following states:
   * A complete Agreement ending in `ACTIVE`.
   * A complete Agreement ending in `STOPPED`.
   * A complete `"type":"RECURRING"` Charge ending in  `CHARGED`.
   * A complete `"type":"RECURRING"` Charge ending in  `REFUNDED`.
6. The recurring merchant starts using the MSN and API keys for the production environment.
7. The merchant verifies that what worked in the test environment also work in the production environment.
   **Please note:** Vipps MobilePay does not do any kind of activation or make any changes based on this checklist.
   The API keys for the production environment are made available on
   [portal.vipps.no](https://portal.vipps.no)
   as soon as the customer control (see step 2) is completed, independently of this checklist.
8. The merchant goes live 🎉


[draft-agreement-endpoint]: https://developer.vippsmobilepay.com/api/recurring#tag/Agreement-v3-endpoints/operation/DraftAgreementV3
[fetch-agreement-endpoint]: https://developer.vippsmobilepay.com/api/recurring#tag/Agreement-v3-endpoints/operation/FetchAgreementV3
[update-agreement-patch-endpoint]: https://developer.vippsmobilepay.com/api/recurring#tag/Agreement-v3-endpoints/operation/UpdateAgreementPatchV3
[force-accept-agreement-endpoint]: https://developer.vippsmobilepay.com/api/recurring#tag/Agreement-v3-endpoints/operation/acceptUsingPATCHV3
[list-charges-endpoint]: https://developer.vippsmobilepay.com/api/recurring#tag/Charge-v3-endpoints/operation/ListChargesV3
[create-charge-endpoint]: https://developer.vippsmobilepay.com/api/recurring#tag/Charge-v3-endpoints/operation/CreateChargeV3
[fetch-charge-endpoint]: https://developer.vippsmobilepay.com/api/recurring#tag/Charge-v3-endpoints/operation/FetchChargeV3
[cancel-charge-endpoint]: https://developer.vippsmobilepay.com/api/recurring#tag/Charge-v3-endpoints/operation/CancelChargeV3
[capture-charge-endpoint]: https://developer.vippsmobilepay.com/api/recurring#tag/Charge-v3-endpoints/operation/CaptureChargeV3
[refund-charge-endpoint]: https://developer.vippsmobilepay.com/api/recurring#tag/Charge-v3-endpoints/operation/RefundChargeV3
[userinfo-endpoint]: https://developer.vippsmobilepay.com/api/userinfo#operation/getUserinfo
[access-token-endpoint]: https://developer.vippsmobilepay.com/api/access-token#tag/Authorization-Service/operation/fetchAuthorizationTokenUsingPost
[vipps-test-environment]: https://developer.vippsmobilepay.com/docs/test-environment
