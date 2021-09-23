# Vipps Recurring API Checklist

API version: 2.0

Document version 1.2.1.

For examples of requests and responses, see the Postman collection in [tools](tools/)

## Checklist

- [ ] Integrate _all_ the [API endpoints](https://github.com/vippsas/vipps-recurring-api/blob/master/vipps-recurring-api.md) described in our guide:
    - [ ] Initiate [`POST:/recurring/v2/agreements`](https://vippsas.github.io/vipps-recurring-api/#/Agreement%20Endpoints/draftAgreement)
    - [ ] Check status of agreement [`GET:/recurring/v2/agreements/{agreementId}`](https://vippsas.github.io/vipps-recurring-api/#/Agreement%20Endpoints/getAgreement)
    - [ ] Update an agreement [`PATCH:/recurring/v2/agreements/{agreementId}`](https://vippsas.github.io/vipps-recurring-api/#/Agreement%20Endpoints/updateAgreement)
    - [ ] Create Charge [`POST:/recurring/v2/agreements/{agreementId}/charges`](https://vippsas.github.io/vipps-recurring-api/#/Charge%20Endpoints/createCharge)
    - [ ] GET Charge [`GET:/recurring/v2/agreements/{agreementId}/charges/{chargeId}`](https://vippsas.github.io/vipps-recurring-api/#/Charge%20Endpoints/getCharge)
    - [ ] Cancel Charge [`DELETE:/recurring/v2/agreements/{agreementId}/charges/{chargeId}`](https://vippsas.github.io/vipps-recurring-api/#/Charge%20Endpoints/cancelCharge)
    - [ ] Refund Charge [`POST:/recurring/v2/agreements/{agreementId}/charges/{chargeId}/refund`](https://vippsas.github.io/vipps-recurring-api/#/Charge%20Endpoints/refundCharge)
    - [ ] Capture reserved initial Charge [`POST:/recurring/v2/agreements/{agreementId}/charges/{chargeId}/capture`](https://vippsas.github.io/vipps-recurring-api/#/Charge%20Endpoints/captureCharge)
- [ ] Avoid Integration pitfalls
    - [ ] The Merchant _must not_ rely on `Redirect` alone
    - [ ] The Vipps branding must be according to the [Vipps design guidelines](https://github.com/vippsas/vipps-design-guidelines)
    - [ ] The merchant must have a way for the user to manage and stop subscription in merchantAgreementUrl in their Agreement. This should result in a timely update of the Vipps Agreement.
    - [ ] Vipps never generates charges automatically on behalf of the Merchant. Every charge is the responsibility of the Merchant.

## Flow to go live for direct integrations

1. The merchant orders [Vipps på Nett](https://www.vipps.no/produkter-og-tjenester/bedrift/ta-betalt-paa-nett/ta-betalt-paa-nett/).
2. Vipps completes customer control (KYC, PEP, AML, etc).
3. The merchant receives an email from Vipps saying that they can log in with bankID on [portal.vipps.no](https://portal.vipps.no) and retrieve API keys.
4. The merchant completes all checklist items.
5. The merchant [contacts Vipps](https://github.com/vippsas/vipps-developers/blob/master/contact.md) with the MSN to check.
   Vipps will check agreements and charges in the [Vipps test environment](https://github.com/vippsas/vipps-developers#the-vipps-test-environment-mt), showing that all checklist items have been fulfilled:
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
