<!-- START_METADATA
---
title: Recurring API extended Postman guide
sidebar_label: Postman guide
sidebar_position: 190
description: Extensive examples of the Recurring API using Postman.
pagination_next: null
pagination_prev: null
---
END_METADATA -->

# Recurring API extended Postman guide

This is a guide for the using the Recurring API with Postman, and is an extension of the
[Recurring API quick start guide](recurring-api-quick-start.md) which shows the
recommended flow in both Postman and curl.

## Setup

You must have already signed up as an organization with Vipps MobilePay and have
your test credentials from the merchant portal.

You will need the following values, as described in the
[Getting started guide](https://developer.vippsmobilepay.com/docs/getting-started):

* `client_id` - Client_id for a test sales unit.
* `client_secret` - Client_id for a test sales unit.
* `Ocp-Apim-Subscription-Key` - Subscription key for a test sales unit.
* `merchantSerialNumber` - The unique ID for a test sales unit.
* `mobileNumber` - The phone number for the test app profile you have received or registered.

Import the following files into Postman:

* [Recurring API Postman collection](/tools/recurring-v3-api-postman-collection.json)
* [Global Postman environment](https://github.com/vippsas/vipps-developer-docs/blob/main/static/internal-tools/global-postman-environment.json)

🔥 **To reduce risk of exposure, never store production keys in Postman or any similar tools.** 🔥

Update the *Current Value* field in your Postman environment with your **Merchant Test** keys.
Use *Current Value* field for added security, as these values are not synced to the cloud.

## Make API calls

For all the following, you will start by sending a request `Get Access Token`.
This provides you with access to the API.

The access token is valid for 1 hour in the test environment
and 24 hours in the production environment.
See the
[API reference](https://developer.vippsmobilepay.com/api/recurring)
for details about the calls.

### Create agreements

For details about the calls, see [Agreement endpoints][agreement-endpoints] in the Recurring API Specifications.

#### Create a minimal agreement

1. Send request `Get Access Token`. This provides you with access to the API.

2. Set `Idempotency-Key-Draft` value.

3. Send the `Draft Agreement - Legacy pricing` request.
   This demonstrates how to create a simple agreement using [`POST:/recurring/v3/agreements`][draft-agreement-endpoint] endpoint.

   Ctrl+click the link, it will take you to the
   [landing page](https://developer.vippsmobilepay.com/docs/knowledge-base/landing-page/).
   Enter your test phone number and complete the authorization in the Vipps or MobilePay app in your mobile test app.

   You should now have an active agreement. The variable `agreementId` is set in the environment.

4. To get the information about an agreement, send `Get Agreement`. This uses
   [`GET:/recurring/v3/agreements/{agreementId}`][fetch-agreement-endpoint].
   The variable `agreementId` is set by the `Draft Agreement` step.

   If you confirmed the agreement, the status should be ACTIVE in the response.
   If you didn't go through with the confirmation, the status will be PENDING for a
   [maximum of 10 minutes](https://developer.vippsmobilepay.com/docs/knowledge-base/timeouts),
   before it goes to `EXPIRED`.

#### Create an agreement with an initial charge

You can create more complex types of agreements by modifying the parameters in [`POST:/recurring/v3/agreements`][draft-agreement-endpoint] endpoint.

1. Send `Draft Agreement - Legacy pricing and direct capture initial charge`.  This demonstrates the addition of an initial charge.

   Ctrl+click on the link and complete the authorization.

   When you use this, a charge is automatically created for the initial payment and charged.
   The `agreementId` and `chargeId` are set in the environment.

2. Set `Idempotency-Key-Draft` value.

3. Send a request `Get Agreement` for information about this payment by using the
   [`GET:/recurring/v3/agreements/{agreementId}`][fetch-agreement-endpoint] endpoint.

#### Create an agreement with a reserve capture

When you use this, a charge is automatically created for the initial payment, and
you can capture it when you provide the product or service.

For example, if a customer purchased a mobile phone with a data subscription, you could set up a recurring
payment agreement where the price of the phone is registered as an initial payment with reserve capture.
Then, you can capture the payment when you ship the phone.

1. Send `Draft Agreement - Legacy pricing and reserve initial charge`.

   Ctrl+click on the link and complete the authorization.

2. Set `Idempotency-Key-Draft` value.

3. Send a request `Get Agreement` for information about this payment by using the
   [`GET:/recurring/v3/agreements/{agreementId}`][fetch-agreement-endpoint] endpoint.

4. The `agreementId` and `chargeId` are set in the environment.
   Take note of these values, because you will need them when you
   [capture the reserved charge](#capture-reserved-charge).

#### Getting access to user info

If you need to get access to some user information in addition to the recurring payment agreement, you can use the profile flow.

1. Send request `Draft Agreement - Profile sharing flow`.

   Provide the `scope` object in the [`POST:/recurring/v3/agreements`][draft-agreement-endpoint] call. This contains the information types that you want access to, separated by spaces (e.g., "name address email phoneNumber birthDate").

   Here, the `agreementId` and `landing_page_url` are retrieved from the response and set as variables.

2. Once you complete the session, a unique identifier `sub` can be retrieved in the agreement details.
   Send a request `Get Agreement` for information about this payment by using the
   [`GET:/recurring/v3/agreements/{agreementId}`][fetch-agreement-endpoint] endpoint.

   In this example, `sub` is retrieved from the response and set as a variable.

3. Send request `Get Userinfo`, from the *User Info* folder. This uses [`GET:/vipps-userinfo-api/userinfo/{sub}`][userinfo-endpoint] with the `sub` variable from the previous call.

#### Get a list of agreements

1. Send `List Agreements` to get a list of agreements by using the
   [`GET:/recurring/v3/agreements`][list-agreements-endpoint] endpoint.

   This includes a query `status=ACTIVE`, so it filters out other
   [agreement states](recurring-api-guide.md#agreement-states).

2. Change or remove the query to see agreements with other states.

#### Update an agreement

1. Set `agreementId` to the ID of an ACTIVE agreement.

2. Run `Get Agreement` to see the properties of the agreement.

3. Run `Update agreement` which modifies some properties by using the
   [`PATCH:/recurring/v3/agreements/{agreementId}`][update-agreement-patch-endpoint] endpoint.

4. Run `Get Agreement` to see the updated properties.

#### Stop an agreement

1. Set `agreementId` to the ID of an ACTIVE agreement.

2. Send the `Stop agreement` request, where the status is set to `STOPPED` in the body of [`PATCH:/recurring/v3/agreements/{agreementId}`][update-agreement-patch-endpoint].

3. Run `Get Agreement` to see that the status is `STOPPED``.

### Create charges

You will need at least one ACTIVE agreement for these examples.

For details about the calls, see [Charge endpoints][charge-endpoints] in the Recurring API Specifications.

#### Create a charge

Charges for initial payments are created automatically.
However, you need to create charge requests for the recurring payments.

A charge must be scheduled a minimum of two days before the payment will occur (it is a minimum one day in the test environment).
See [Direct Capture](recurring-api-guide.md#direct-capture) for more details about timing.

1. Set `agreementId` to the ID of an ACTIVE agreement.

2. Set `Idempotency-Key-Create` value.

3. Send `Create Charge - Direct capture`. This uses the
   [`POST:/recurring/v3/agreements/{agreementId}/charges`][create-charge-endpoint] endpoint.

   The `chargeId` variable is set for later use.

4. Send `Get Charge`.
   This uses [`POST:/recurring/v3/agreements/{agreementId}/charges/{chargeId}`][fetch-charge-endpoint]
   to get information about the charge. The status will be `PENDING` until the due date, when the payment is processed.

5. If you run `Get Charge` again on the due date, the status should be `CHARGED`.

#### Get a list of charges for an agreement

1. Set `agreementId` to the ID of an agreement.
   Note, you can get a list of all your agreements with the `Fetch Agreements` example.

2. Send `List Charges` which uses [`GET:/recurring/v3/agreements/{agreementId}/charges`][list-charges-endpoint].

   This includes a query `chargeStatus=DUE` and filters out other
   [charge states](recurring-api-guide.md#charge-states).

3. Change or remove the query to see charges with other states.

#### Cancel a charge

You can cancel an existing charge before the user is charged.

1. Create a new charge, that we can safely cancel, by sending
   `Create Charge - Reserve capture`.

   The `chargeId` variable is set to this charge.

2. Send `Cancel Charge` which uses
   [`DELETE:/recurring/v3/agreements/{agreementId}/charges/{chargeId}`][cancel-charge-endpoint].

   Send `Get Charge` to see the change of status.

#### Refund a charge

You can refund a charge that has already been charged.

1. Set `chargeId` to the ID of a charge with status of CHARGED.
   The easiest way to test this is to run `Draft Agreement - Reserve initial charge`.
   This creates a charge that is immediately processed, and
   it sets the `agreementId` and `chargeId` to the corresponding values.

2. Set `Idempotency-Key-Refund` value.

3. Send `Get Charge` to see the change of status.

4. Send `Refund Charge` which uses [`POST:/recurring/v3/agreements/{agreementId}/charges/{chargeId}/refund`][refund-charge-endpoint].

5. Send `Get Charge` to see that the charge is all or partially refunded.
   To fully refund, set the `amount` value to the amount remaining (`amount` - `amountRefunded`).

#### Capture reserved charge

When you create an agreement with a reserved charge, you will need to capture this charge.

1. In the environment, set `agreementId` and `chargeId` to refer to the agreement and charge.
   If you ran [Draft Agreement - Reserve initial charge](#create-an-agreement-with-a-reserve-capture),
   you can use the values set by the example.

2. Set `Idempotency-Key-Capture` value.

3. Send `Get Charge`, to see the status of this charge.

4. Send `Capture reserved charge` which uses [`POST:/recurring/v3/agreements/{agreementId}/charges/{chargeId}/capture`][capture-charge-endpoint].

5. Send `Get Charge` again, to see that the status is now "CHARGED".


[list-agreements-endpoint]: https://developer.vippsmobilepay.com/api/recurring#tag/Agreement-v3-endpoints/operation/ListAgreementsV3
[draft-agreement-endpoint]: https://developer.vippsmobilepay.com/api/recurring#tag/Agreement-v3-endpoints/operation/DraftAgreementV3
[agreement-endpoints]: https://developer.vippsmobilepay.com/api/recurring#tag/Agreement-v3-endpoints
[fetch-agreement-endpoint]: https://developer.vippsmobilepay.com/api/recurring#tag/Agreement-v3-endpoints/operation/FetchAgreementV3
[update-agreement-patch-endpoint]: https://developer.vippsmobilepay.com/api/recurring#tag/Agreement-v3-endpoints/operation/UpdateAgreementPatchV3
[force-accept-agreement-endpoint]: https://developer.vippsmobilepay.com/api/recurring#tag/Agreement-v3-endpoints/operation/acceptUsingPATCHV3
[charge-endpoints]: https://developer.vippsmobilepay.com/api/recurring#tag/Charge-v3-endpoints
[list-charges-endpoint]: https://developer.vippsmobilepay.com/api/recurring#tag/Charge-v3-endpoints/operation/ListChargesV3
[create-charge-endpoint]: https://developer.vippsmobilepay.com/api/recurring#tag/Charge-v3-endpoints/operation/CreateChargeV3
[fetch-charge-endpoint]: https://developer.vippsmobilepay.com/api/recurring#tag/Charge-v3-endpoints/operation/FetchChargeV3
[cancel-charge-endpoint]: https://developer.vippsmobilepay.com/api/recurring#tag/Charge-v3-endpoints/operation/CancelChargeV3
[capture-charge-endpoint]: https://developer.vippsmobilepay.com/api/recurring#tag/Charge-v3-endpoints/operation/CaptureChargeV3
[refund-charge-endpoint]: https://developer.vippsmobilepay.com/api/recurring#tag/Charge-v3-endpoints/operation/RefundChargeV3
[userinfo-endpoint]: https://developer.vippsmobilepay.com/api/userinfo#operation/getUserinfo
[access-token-endpoint]: https://developer.vippsmobilepay.com/api/access-token#tag/Authorization-Service/operation/fetchAuthorizationTokenUsingPost
[vipps-test-environment]: https://developer.vippsmobilepay.com/docs/test-environment
