# Vipps Recurring API Checklist

API version: 2.0

Document version 1.1.1

For examples of requests and responses, see the Postman collection in [tools](tools/)

# Checklist

- [ ] Integrate _all_ the [API endpoints](https://github.com/vippsas/vipps-recurring-api/blob/master/vipps-recurring-api.md) described in out guide:
    - [ ] Initiate [`POST:/v2/agreements`](https://vippsas.github.io/vipps-recurring-api/#/Agreement%20Controller/draftAgreement)
    - [ ] Check status of agreement [`GET:/v2/agreements/{agreementId}`](https://vippsas.github.io/vipps-recurring-api/#/Agreement%20Controller/getAgreement)
    - [ ] Update an agreement [`PATCH:/v2/agreements/{agreementId`](https://vippsas.github.io/vipps-recurring-api/#/Agreement%20Controller/updateAgreement)
    - [ ] Create Charge [`POST:/v2/agreements/{agreementId}/charges`](https://vippsas.github.io/vipps-recurring-api/#/Charge%20Controller/createCharge)
    - [ ] GET Charge [`GET:/v2/agreements/{agreementId}/charges/{chargeId}`](https://vippsas.github.io/vipps-recurring-api/#/Charge%20Controller/getCharge)
    - [ ] Cancel Charge [`DELETE:/v2/agreements/{agreementId}/charges/{chargeId}`](https://vippsas.github.io/vipps-recurring-api/#/Charge%20Controller/cancelCharge)
    - [ ] Refund Charge [`POST:/v2/agreements/{agreementId}/charges/{chargeId}/refund`](https://vippsas.github.io/vipps-recurring-api/#/Charge%20Controller/refundCharge)
- [ ] Avoid Integration pitfalls
    - [ ] The Merchant _must not_ rely on `Redirect` alone
    - [ ] The Vipps branding must be according to the [Vipps design guidelines](https://github.com/vippsas/vipps-design-guidelines)
    - [ ] The merchant must have a way for the user to manage and stop subscription in merchantAgreementUrl in their Agreement. This should result in a timely update of the Vipps Agreement.
    - [ ] Vipps never generates charges automatically on behalf of the Merchant. Every charge is the responsibility of the Merchant.

# Flow to go live for direct integrations

1. The merchant orders [Vipps på Nett](https://www.vipps.no/produkter-og-tjenester/bedrift/ta-betalt-paa-nett/ta-betalt-paa-nett/).
2. Vipps completes customer control (KYC, PEP, AML, etc).
3. The merchant receives an email from Vipps saying that they can log in with bankID on [portal.vipps.no](https://portal.vipps.no) and retrieve API keys.
4. The merchant completes all checklist items.
5. The merchant [contacts Vipps](https://github.com/vippsas/vipps-developers/blob/master/contact.md) with test IDs for agreeement(`agreementId`) and charges (`chargeId`) in the [Vipps test environment](https://github.com/vippsas/vipps-developers#the-vipps-test-environment-mt), showing that all checklist items have been fulfilled.
    - A complete Agreement ending in `ACTIVE`.
    - A complete Agreement ending in `STOPPED`.
    - A complete Charge ending in  `CHARGED`.
    - A complete Charge ending in  `REFUNDED`.
6. The merchant receives an email from Vipps saying that the orders are OK.
7. The recurring functionality is activated in production.
8. The Merchant [contacts Vipps](https://github.com/vippsas/vipps-developers/blob/master/contact.md) to verify the integration in the production environment:
    - A complete Agreement ending in `ACTIVE`.
    - A complete Agreement ending in `STOPPED`.
    - A complete Charge ending in  `CHARGED`.
    - A complete Charge ending in  `REFUNDED`.
9. The Merchant goes live 🎉
