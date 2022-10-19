<!-- START_METADATA
---
title: Checklist
sidebar_position: 40
---
END_METADATA -->

# Vipps Recurring API Checklist

<!-- START_COMMENT -->

ℹ️ Please use the new documentation:
[Vipps Technical Documentation](https://vippsas.github.io/vipps-developer-docs/).

<!-- END_COMMENT -->

API version: 3.0

Document version 3.0.0.

## Checklist

- [ ] Integrate _all_ the [API endpoints](vipps-recurring-api.md) described in our guide:
    - [ ] Agreement: Create an agreement [`POST:/agreements`][draft-agreement-endpoint]
    - [ ] Agreement: Retrieve an agreement [`GET:/agreements/{agreementId}`][fetch-agreement-endpoint]
    - [ ] Agreement: Update an agreement [`PATCH:/agreements/{agreementId}`][update-agreement-patch-endpoint]
    - [ ] Agreement: List charges [`GET:/agreements/{agreementId}/charges`][list-charges-endpoint]
    - [ ] Charge: Create Charge [`POST:/agreements/{agreementId}/charges`][create-charge-endpoint]
    - [ ] Charge: Retrieve a charge [`GET:/agreements/{agreementId}/charges/{chargeId}`][fetch-charge-endpoint]
    - [ ] Charge: Cancel a charge [`DELETE:/agreements/{agreementId}/charges/{chargeId}`][cancel-charge-endpoint]
    - [ ] Charge: Refund a charge [`POST:/agreements/{agreementId}/charges/{chargeId}/refund`][refund-charge-endpoint]
    - [ ] Charge: Capture a reserved charge [`POST:/agreements/{agreementId}/charges/{chargeId}/capture`][capture-charge-endpoint]
    - For examples of requests and responses, see the [Postman collection](./tools/vipps-recurring-api-postman-collection.json) and [environment](https://github.com/vippsas/vipps-developers/blob/master/tools/vipps-api-global-postman-environment.json).
  - [ ] Send the [Vipps HTTP headers](https://github.com/vippsas/vipps-developers/blob/master/vipps-getting-started.md#vipps-http-headers)
        in all API requests for better tracking and troubleshooting
        (mandatory for partners and platforms, who must send these headers as part of the checklist approval):
      - [ ] `Merchant-Serial-Number`
      - [ ] `Vipps-System-Name`
      - [ ] `Vipps-System-Version`
      - [ ] `Vipps-System-Plugin-Name`
      - [ ] `Vipps-System-Plugin-Version`
- [ ] Avoid Integration pitfalls
    - [ ] The Merchant _must not_ rely on `Redirect` alone, but must always poll [`GET:/agreements/{agreementId}`][fetch-agreement-endpoint] to check the status of the agreement.
    - [ ] Monitor the error messages Vipps returns from the APIs, and correct problems as quickly as possible. It's recommended to log errors emitted by our APIs, this makes it easier to pinpoint what has gone wrong.
    - [ ] The Vipps branding must be according to the [Vipps design guidelines](https://github.com/vippsas/vipps-design-guidelines)
    - [ ] The merchant must have a way for the user to manage and stop subscription in merchantAgreementUrl in their Agreement. This should result in a timely update of the Vipps Agreement.
    - [ ] Vipps never generates charges automatically on behalf of the Merchant. Every charge is the responsibility of the Merchant.
    - [ ] Make sure your customer service, etc. has all the tools and information they need
          available in _your_ system, through the APIs listed in the first item in this checklist,
          and that they do not need to visit
          [portal.vipps.no](https://portal.vipps.no)
          for normal work.

## Flow to go live for direct integrations

1. The merchant orders [Vipps på Nett](https://www.vipps.no/produkter-og-tjenester/bedrift/ta-betalt-paa-nett/ta-betalt-paa-nett/).
2. Vipps completes customer control (KYC, PEP, AML, etc).
3. The merchant receives an email from Vipps saying that they can log in with bankID on [portal.vipps.no](https://portal.vipps.no) and retrieve API keys.
4. The merchant completes all checklist items.
5. The merchant [contacts Vipps](https://github.com/vippsas/vipps-developers/blob/master/contact.md) with the MSN to check.
   Vipps will check agreements and charges in the [Vipps test environment](https://github.com/vippsas/vipps-developers/blob/master/vipps-test-environment.md), showing that all checklist items have been fulfilled:
    - A complete Agreement ending in `ACTIVE`.
    - A complete Agreement ending in `STOPPED`.
    - A complete `"type":"RECURRING"` Charge ending in  `CHARGED`.
    - A complete `"type":"RECURRING"` Charge ending in  `REFUNDED`.
6. The merchant receives an email from Vipps saying that the MSN is OK.
7. The recurring merchant starts using the MSN and API keys for the production environment.
8. The merchant verifies that what worked in the test environment also work in the production environment.
   **Please note:** Vipps does not do any kind of activation or make any changes based on this checklist.
   The API keys for the production environment are made available on
   [portal.vipps.no](https://portal.vipps.no)
   as soon as the customer control (see step 2) is completed, independently of this checklist.
9. The Merchant goes live 🎉

## Questions?

We're always happy to help with code or other questions you might have!
Please create an [issue](https://github.com/vippsas/vipps-recurring-api/issues),
a [pull request](https://github.com/vippsas/vipps-recurring-api/pulls),
or [contact us](https://github.com/vippsas/vipps-developers/blob/master/contact.md).

Sign up for our [Technical newsletter for developers](https://github.com/vippsas/vipps-developers/tree/master/newsletters).

[draft-agreement-endpoint]: https://vippsas.github.io/vipps-developer-docs/api/recurring#tag/Agreement-v3-endpoints/operation/DraftAgreementV3
[fetch-agreement-endpoint]: https://vippsas.github.io/vipps-developer-docs/api/recurring#tag/Agreement-v3-endpoints/operation/FetchAgreementV3
[update-agreement-patch-endpoint]: https://vippsas.github.io/vipps-developer-docs/api/recurring#tag/Agreement-v3-endpoints/operation/UpdateAgreementPatchV3
[force-accept-agreement-endpoint]: https://vippsas.github.io/vipps-developer-docs/api/recurring#tag/Agreement-v3-endpoints/operation/acceptUsingPATCHV3
[list-charges-endpoint]: https://vippsas.github.io/vipps-developer-docs/api/recurring#tag/Charge-v3-endpoints/operation/ListChargesV3
[create-charge-endpoint]: https://vippsas.github.io/vipps-developer-docs/api/recurring#tag/Charge-v3-endpoints/operation/CreateChargeV3
[fetch-charge-endpoint]: https://vippsas.github.io/vipps-developer-docs/api/recurring#tag/Charge-v3-endpoints/operation/FetchChargeV3
[cancel-charge-endpoint]: https://vippsas.github.io/vipps-developer-docs/api/recurring#tag/Charge-v3-endpoints/operation/CancelChargeV3
[capture-charge-endpoint]: https://vippsas.github.io/vipps-developer-docs/api/recurring#tag/Charge-v3-endpoints/operation/CaptureChargeV3
[refund-charge-endpoint]: https://vippsas.github.io/vipps-developer-docs/api/recurring#tag/Charge-v3-endpoints/operation/RefundChargeV3
[userinfo-endpoint]: https://vippsas.github.io/vipps-developer-docs/api/recurring#tag/Userinfo-Endpoint/operation/getUserinfo
[access-token-endpoint]: https://vippsas.github.io/vipps-developer-docs/api/recurring#tag/Authorization-Service/operation/getAccessToken
[vipps-test-environment]: https://github.com/vippsas/vipps-developers/blob/master/vipps-test-environment.md
