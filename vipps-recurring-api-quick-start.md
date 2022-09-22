<!-- START_METADATA
---
title: Postman
sidebar_position: 40
---
END_METADATA -->

# Quick start

<!-- START_TOC -->

## Table of Contents

* [Postman](#postman)
  * [Step 1: Get the Postman collection](#step-1-get-the-postman-collection)
  * [Step 2: Import the Postman environment](#step-2-import-the-postman-environment)
  * [Step 3: Set up Postman environment](#step-3-setup-postman-environment)
* [Make API calls](#make-api-calls)
  * [Create agreements](#create-agreements)
  * [Create charges](#create-charges)
* [Questions?](#questions)

<!-- END_TOC -->

Document version 1.0.1.

## Postman

[Postman](https://www.getpostman.com/) is a common tool for working with REST APIs.
We offer a [Postman Collection](https://www.getpostman.com/collection) to make development easier.
See the [Postman documentation](https://www.getpostman.com/docs/) for more information about using Postman.

By following the steps below, you can make calls to all the
endpoints, and see the full `request` and `response` for each call.

### Step 1: Get the Postman collection

Import the collection by following the steps below:

1. Click `Import` in the upper left corner.
2. Import the [vipps-recurring-api-postman-collection.json](./tools/vipps-recurring-api-postman-collection.json) file.

### Step 2: Import the Postman environment

1. Click `Import` in the upper-left corner.
2. Import the [vipps-recurring-api-postman-environment.json](./tools/vipps-recurring-api-postman-environment.json) file.

### Step 3: Setup Postman environment

1. Click the down arrow, next to the "eye" icon in the top-right corner, and select the environment you have imported.
2. Click the "eye" icon and, in the dropdown window, click `Edit` in the top-right corner.
3. Fill in the `Current Value` for the following fields to get started. For the first three keys, go to *Vipps Portal* > *Utvikler* ->  *Test Keys*.
   - `client_id` - Merchant key is required for getting the access token.
   - `client_secret` - Merchant key is required for getting the access token.
   - `Ocp-Apim-Subscription-Key` - Merchant subscription key.
   - `merchantSerialNumber` - Merchant id.
   - `customerPhoneNumber` - The mobile number for the test app profile you have received or registered.

## Make API calls

For all of the following, you will start by sending request `Get Access Token`.
This provides you with access to the API.

   The access token is valid for 1 hour in the test environment
   and 24 hours in the production environment.

See the
[API reference](https://vippsas.github.io/vipps-developer-docs/api/recurring)
for details about the calls.

### Create agreements

For details about the calls, see [Agreement v2 endpoints][agreement-endpoints] in the Recurring API Specifications.

#### 1. Create a minimal agreement

1. Send the `Draft Agreement - Minimal` request.
   This demonstrates how to create a simple agreement using
   [`POST:/v2/agreements`][draft-agreement-endpoint].

   Ctrl+click the link, it will take you to the Vipps landing page.
   Enter your test phone number and complete the authorization in the Vipps app in your mobile test app.

   You should now have an active agreement. The variable `agreementId` is set in the environment.

2. To get the information about an agreement, send `Get Agreement`. This uses
   [`GET:/v2/agreements/{{agreementId}}`][fetch-agreement-endpoint].
   The variable `agreementId` is set by the `Draft Agreement` step.

   If you authorized the agreement, the status should be ACTIVE in the response.
   If you didn't go through with the authorization, the status will be PENDING for a
   [maximum of 10 minutes](vipps-recurring-api.md#timeouts),
   before it goes to `EXPIRED`.

#### 2. Create an agreement with an initial charge

You can create more complex types of agreements by modifying the parameters in [`POST:/v2/agreements`][draft-agreement-endpoint].

1. Send `Draft Agreement - Full`.  This demonstrates the addition of an initial charge and a campaign.

   Ctrl+click on the link and complete the authorization.

   When you use this, a charge is automatically created for the initial payment and charged.
   The `agreementId` and `chargeId` are set in the environment.

2. Send request `Get Agreement` for information about this payment by using
   [`GET:/v2/agreements/{{agreementId}}`][fetch-agreement-endpoint].

#### 3. Create an agreement with a reserve capture

When you use this, a charge is automatically created for the initial payment, and
you can capture it when you provide the product or service.

For example, if a customer purchased a mobile phone with a data subscription, you could set up a recurring
payment agreement where the price of the phone is registered an initial payment with reserve capture.
Then, you can capture the payment when you ship the phone.

1. Send `Draft Agreement - Reserve Charge`.

   Ctrl+click on the link and complete the authorization.

2. Send request `Get Agreement` for information about this payment by using
   [`GET:/v2/agreements/{{agreementId}}`][fetch-agreement-endpoint].

3. The `agreementId` and `chargeId` are set in the environment.
   Take a note of these values, because you will need them when you
   [capture the reserved charge](#5-capture-reserved-charge).

#### 4. Getting access to user info

If you need to get access to some user information in addition to the recurring payment agreement, you can use the profile flow.

1. Send request `Draft Agreement - Profile flow`.

   Provide the `scope` object in the [`POST:/v2/agreements`][draft-agreement-endpoint] call. This contains the information types that you want access to, separated by spaces (e.g., "name address email phoneNumber birthDate").

   Here, the `agreementId` and `landing_page_url` are retrieved from the response and set as variables.

2. Once you complete the session, a unique identifier `sub` can be retrieved in the agreement details.
   Send request `Get Agreement` for information about this payment by using
   [`GET:/v2/agreements/{{agreementId}}`][fetch-agreement-endpoint].

   In this example, `sub` is retrieved from the response and set as a variable.

3. Send request `Get Userinfo`, from the *User Info* folder. This uses [`GET:/vipps-userinfo-api/userinfo/{sub}`][userinfo-endpoint] with the `sub` variable from the previous call.

See [User info](vipps-recurring-api.md#userinfo) for more information.

#### 5. Get a list of agreements

1. Send `Fetch Agreements` to get a list of agreements by using
   [`GET:/v2/agreements`][list-agreements-endpoint].

   This includes a query `status=ACTIVE`, so it filters out other
   [agreement states](vipps-recurring-api.md#agreement-states).

2. Change or remove the query to see agreements with other states.

#### 6. Update an agreement

1. Set `agreementId` to the id of an ACTIVE agreement.

2. Run `Get Agreement` to see the properties of the agreement.

3. Run `Update agreement` which modifies some properties by using
   [`PUT:/v2/agreements/{{agreementId}}`][update-agreement-put-endpoint].

4. Run `Get Agreement` to see the updated properties.

#### 7. Stop an agreement

1. Set `agreementId` to the id of an ACTIVE agreement.

2. Send the `Stop agreement` request, where the status is set to `STOPPED` in the body of [`PUT:/v2/agreements/{{agreementId}}`][update-agreement-put-endpoint].

3. Run `Get Agreement` to see that the status is not "STOPPED".

### Create charges

You will need at least one ACTIVE agreement for these examples.

For details about the calls, see [Charge v2 endpoints][charge-endpoints] in the Recurring API Specifications.

#### 1. Create a charge

Although charges for initial payments are created automatically,
you must create charge requests for the recurring payments.

A charge must be scheduled a minimum of two days before the payment will occur (it is minimum one day in the test environment).
See [Direct Capture](vipps-recurring-api.md#direct-capture) for more details about timing.

1. Set `agreementId` to the id of an ACTIVE agreement.

2. Send `Create Charge - Due tomorrow`. This uses
   [`POST:/v2/agreements/{{agreementId}}/charges`][create-charge-endpoint]
   with "due" set to tomorrow's date.

   The `chargeId` variable is set for later use.

3. Send `Get Charge`.
   This uses [`GET:/v2/agreements/{{agreementId}}/charges/{{chargeId}}`][fetch-charge-endpoint]
   to get information about the charge. The status will be "PENDING" for a while before it goes to "DUE".

4. If you run `Get Charge` again tomorrow, you should see that the status changes to "CHARGED".

#### 2. Get a list of charges for an agreement

1. Set `agreementId` to the id of an agreement.
   Note, you can get a list of all your agreements with the `Fetch Agreements` example.

2. Send `List Charges` which uses [`GET:/v2/agreements/{{agreementId}}/charges`][list-charges-endpoint].

   This includes a query `chargeStatus=DUE` and filters out other
   [charge states](vipps-recurring-api.md#charge-states).

3. Change or remove the query to see charges with other states.

#### 3. Cancel a charge

You can cancel an existing charge before the user is charged.

1. Create a new charge, that we can safely cancel, by sending
   `Create Charge - Due tomorrow`.

   The `chargeId` variable is set to this charge.

2. Send `Cancel Charge` which uses
   [`DEL:/v2/agreements/{{agreementId}}/charges/{{chargeId}}`][cancel-charge-endpoint].

   Send `Get Charge` to see the change of status.

#### 4. Refund a charge

You can refund a charge that has already been charged.

1. Set `chargeId` to the id of a charge with status of CHARGED.
   The easiest way to test this is to run `Draft Agreement - Full`.
   This creates a charge that is immediately processed, and
   it sets the `agreementId` and `chargeId` to the corresponding values.

2. Send `Get Charge` to see the change of status.

3. Send `Refund Charge` which uses
   [`post:{{base_url}}/recurring/v2/agreements/{{agreementId}}/charges/{{chargeId}}/refund`][refund-charge-endpoint].

4. Send `Get Charge` to see that the charge is all or partially refunded.
   To fully refund, set the amount value to the amount remaining (amount - amountRefunded).

#### 5. Capture reserved charge

When you create an agreement with a reserved charge, you will need to capture this charge.

1. In the environment, set `agreementId` and `chargeId` to refer to the agreement and charge.
   If you ran [Create an agreement with a reserve capture](#3-create-an-agreement-with-a-reserve-capture),
   you can use the values set by the example.

2. Send `Get Charge`, to see the status of this charge.

3. Send `Capture reserved charge` which uses [`POST:/v2/agreements/{{agreementId}}/charges/{{chargeId}}/capture`][capture-charge-endpoint].

4. Send `Get Charge` again, to see that the status is now "CHARGED".

## Questions?

We're always happy to help with code or other questions you might have!
Please create an [issue](https://github.com/vippsas/vipps-ecom-api/issues),
a [pull request](https://github.com/vippsas/vipps-ecom-api/pulls),
or [contact us](https://github.com/vippsas/vipps-developers/blob/master/contact.md).

Sign up for our [Technical newsletter for developers](https://github.com/vippsas/vipps-developers/tree/master/newsletters).

[list-agreements-endpoint]: https://vippsas.github.io/vipps-developer-docs/api/recurring#tag/Agreement-v2-endpoints/operation/ListAgreements
[draft-agreement-endpoint]: https://vippsas.github.io/vipps-developer-docs/api/recurring#tag/Agreement-v2-endpoints/operation/DraftAgreement
[agreement-endpoints]: https://vippsas.github.io/vipps-developer-docs/api/recurring#tag/Agreement-v2-endpoints
[fetch-agreement-endpoint]: https://vippsas.github.io/vipps-developer-docs/api/recurring#tag/Agreement-v2-endpoints/operation/FetchAgreement
[update-agreement-patch-endpoint]: https://vippsas.github.io/vipps-developer-docs/api/recurring#tag/Agreement-v2-endpoints/operation/UpdateAgreementPatch
[update-agreement-put-endpoint]: https://vippsas.github.io/vipps-developer-docs/api/recurring#tag/Agreement-v2-endpoints/operation/UpdateAgreementPut
[force-accept-agreement-endpoint]: https://vippsas.github.io/vipps-developer-docs/api/recurring#tag/Agreement-v2-endpoints/operation/acceptUsingPATCH
[charge-endpoints]: https://vippsas.github.io/vipps-developer-docs/api/recurring#tag/Charge-v2-endpoints
[list-charges-endpoint]: https://vippsas.github.io/vipps-developer-docs/api/recurring#tag/Charge-v2-endpoints/operation/ListCharges
[create-charge-endpoint]: https://vippsas.github.io/vipps-developer-docs/api/recurring#tag/Charge-v2-endpoints/operation/CreateCharge
[fetch-charge-endpoint]: https://vippsas.github.io/vipps-developer-docs/api/recurring#tag/Charge-v2-endpoints/operation/FetchCharge
[cancel-charge-endpoint]: https://vippsas.github.io/vipps-developer-docs/api/recurring#tag/Charge-v2-endpoints/operation/CancelCharge
[capture-charge-endpoint]: https://vippsas.github.io/vipps-developer-docs/api/recurring#tag/Charge-v2-endpoints/operation/CaptureCharge
[refund-charge-endpoint]: https://vippsas.github.io/vipps-developer-docs/api/recurring#tag/Charge-v2-endpoints/operation/RefundCharge
[userinfo-endpoint]: https://vippsas.github.io/vipps-developer-docs/api/recurring#tag/Userinfo-Endpoint/operation/getUserinfo
[access-token-endpoint]: https://vippsas.github.io/vipps-developer-docs/api/recurring#tag/Authorization-Service/operation/getAccessToken
[vipps-test-environment]: https://github.com/vippsas/vipps-developers/blob/master/vipps-test-environment.md
