# Postman

[Postman](https://www.getpostman.com/) is a common tool for working with REST APIs.
We offer a [Postman Collection](https://www.getpostman.com/collection) to make development easier.
See the [Postman documentation](https://www.getpostman.com/docs/) for more information about using Postman.

By following the steps below, you can make calls to all the
endpoints, and see the full `request` and `response` for each call.

We also have a short [getting started guide to Postman](https://github.com/vippsas/vipps-developers/blob/master/postman-guide.md).

## Setting up Postman

### Step 1: Get the Postman Collection

Import the collection by following the steps below:

1. Click `Import` in the upper left corner.
2. Import the [vipps-recurring-api-postman-collection.json](https://raw.githubusercontent.com/vippsas/vipps-recurring-api/master/tools/vipps-recurring-api-postman-collection.json) file.

### Step 2: Import the Postman Environment

1. Click `Import` in the upper-left corner.
2. Import the [vipps-recurring-api-postman-environment.json](https://raw.githubusercontent.com/vippsas/vipps-recurring-api/master/tools/vipps-recurring-api-postman-environment.json) file.

### Step 3: Setup Postman Environment

1. Click the down arrow, next to the "eye" icon in the top-right corner, and select the environment you have imported.
2. Click the "eye" icon and, in the dropdown window, click `Edit` in the top-right corner.
3. Fill in the `Current Value` for the following fields to get started. For the first three keys, go to *Vipps Portal* > *Utvikler* ->  *Test Keys*.
   - `client_id` - Merchant key is required for getting the access token.
   - `client_secret` - Merchant key is required for getting the access token.
   - `Ocp-Apim-Subscription-Key` - Merchant subscription key.
   - `merchantSerialNumber` - Merchant id.
   - `mobileNumber` - The mobile number for the test app profile you have received or registered.

### Step 4: Get access token

Send request `Get Access Token`. This provides you with access to the API.

The access token is valid for 1 hour in the test environment.

### Step 5: Run the Agreement Controller examples

See the [Recurring API Specifications](https://vippsas.github.io/vipps-recurring-api/#/) for details about the calls.

#### 1. Create a minimal agreement

1. Send the `Draft Agreement - Minimal` request.
   This demonstrates how to create a simple agreement using
   [`POST:/v2/agreements`]([draft-agreement-endpoint]).

   Ctrl+click the link and it will take you to the Vipps landing page.
   Enter your test phone number and complete the authorization in the Vipps app in your mobile test app.

1. Send `Get Agreement` for information about the agreement by using
   [`GET:/v2/agreements/{{agreementId}}`][fetch-agreement-endpoint].

   The `agreementId` variable was updated in the environment by the previous request.

   If you authorized the agreement, the status should be ACTIVE in the response.
   If you didn't go through with the authorization, the status will be PENDING for a
   [maximum of 10 minutes](vipps-recurring-api.md#timeouts),
   before it goes to EXPIRED.

#### 2. Create an agreement with an initial charge

You can create more complex types of agreements by modifying the parameters in [`POST:/v2/agreements`][draft-agreement-endpoint].

1. Send `Draft Agreement - Full`.  This demonstrates the addition of an initial charge and a campaign.

   Ctrl+click on the link and complete the authorization.

   When you use this, a charge is automatically created for the initial payment and charged.
   The `agreementId` and `chargeId` are set in the environment.

#### 3. Create an agreement with an initial charge and reserve capture

1. Send `Draft Agreement - Reserve Charge`.
   This demonstrates the addition of an initial charge with a transactionType: RESERVE_CAPTURE.

   Ctrl+click on the link and complete the authorization.

   The `agreementId` and `chargeId` are set in the environment.

   When you use this, a charge is automatically created for the initial payment and you can capture it when you provide the product or service.
   For example, if the customer purchases a mobile phone with subscription, you would set up the initial payment, which is the price of the phone,
   as RESERVE_CAPTURE. Then, when you ship the phone, you can send the capture request.


1. Send `Capture reserved charge` which uses [`POST:/v2/agreements/{{agreementId}}/charges/{{chargeId}}/capture`][capture-charge-endpoint].


#### 4. Getting access to user info

If you need to get access to some user information in addition to the recurring payment agreement, you can use the profile flow.

1. Send request `Draft Agreement - Profile flow`.

   Provide the `scope` object in the [`POST:/v2/agreements`][draft-agreement-endpoint] call. This contains the information types that you want access to, separated by spaces (e.g., "name address email phoneNumber birthDate").

   Here, the `agreementId` and `landing_page_url` are retrieved from the response and set as variables.

1. Once you complete the session, a unique identifier `sub` can be retrieved in the agreement details.
   Send request `Get Agreement` for information about this payment by using [`GET:/v2/agreements/{{agreementId}}`][fetch-agreement-endpoint].

   In this example, `sub` is retrieved from the response and set as a variable.

1. Send request `Get Userinfo`. This uses [`GET:/vipps-userinfo-api/userinfo/{sub}`][userinfo-endpoint] with the `sub` variable from the previous call.

See [User info](vipps-recurring-api.md#userinfo) for more information.


#### 5. Get a list of all ACTIVE agreements

1. Send `Fetch Agreements` to get a list of all the agreements by using
   [`GET:/v2/agreements`][list-agreements-endpoint].

1. Remove the query to see all types of agreements.


#### 6. Update an agreement

1. Set `agreementId` to the id of an ACTIVE agreement.

1. Run `Get Agreement` to see the properties of the agreement.

1. Open `Update agreement` and change some properties.

   For example, change the price, product name, product description
   and/or status in the body of
   [`PUT:/v2/agreements/{{agreementId}}`][update-agreement-endpoint].

1. Run `Get Agreement` to see the updated properties.


#### 7. Stop an agreement

1. Set `agreementId` to the id of an ACTIVE agreement.

1. Send the `Stop agreement` request, where the status is set to `STOPPED` in the body of [`PUT:/v2/agreements/{{agreementId}}`][update-agreement-endpoint].

1. Run `Get Agreement` to see the updated properties.

### Step 6: Run the Charge Controller examples

For these examples, you will need at least one ACTIVE agreement.

#### 1. Create a charge

Charges for initial payments are created automatically,
but you must create charge requests for the recurring payments.

Each specific charge on an agreement must be scheduled by the merchant, a
minimum of two days before the payment will occur (it is minimum one day in the test environment).
See [Direct Capture](vipps-recurring-api.md#direct-capture) for more details.

1. Set `agreementId` to the id of an ACTIVE agreement.

1. Send `Create Charge - Due tomorrow`. This creates a charge that is due tomorrow.

   The `chargeId` variable is set for later use.

1. Send `Get Charge`.
   This uses [`GET:/v2/agreements/{{agreementId}}/charges/{{chargeId}}`][fetch-charge-endpoint]
   to get information about the charge.

#### 2. Get a list of all the charges

For a list of all the charges for an agreement, send request [`GET:/v2/agreements/{{agreementId}}/charges`][list-charges-endpoint], where `agreementId` is the id for a specific agreement.

1. Set `agreementId` to the id of an ACTIVE agreement where you have done initial charges or created some charge requests.

1. Send `List Charges`.

#### 3. Cancel a charge

You can cancel an existing charge before the user is charged.

1. Set `chargeId` to the id of a charge with status of DUE.
   You can look for this by running `List Charges`.

1. Send `Cancel Charge` which uses
   [`DEL:/v2/agreements/{{agreementId}}/charges/{{chargeId}}`][cancel-charge-endpoint].

1. Send `Get Charge` to see the change of status.

#### 4. Refund a charge

You can refund a charge that has already been charged.

1. Set `chargeId` to the id of a charge with status of CHARGED.
   The easiest way to test this is to run `Draft Agreement - Full`. This creates a charge that is immediately processed.
   Alternatively, you can run `Create Charge - Due tomorrow` and then wait until tomorrow for it to be processed.

1. Send `Refund Charge` which uses
   [`post:{{base_url}}/recurring/v2/agreements/{{agreementId}}/charges/{{chargeId}}/refund`][refund-charge-endpoint].

1. Send `Get Charge` to see the change of status.

## Questions?

We're always happy to help with code or other questions you might have!
Please create an [issue](https://github.com/vippsas/vipps-ecom-api/issues),
a [pull request](https://github.com/vippsas/vipps-ecom-api/pulls),
or [contact us](https://github.com/vippsas/vipps-developers/blob/master/contact.md).

Sign up for our [Technical newsletter for developers](https://github.com/vippsas/vipps-developers/tree/master/newsletters).


[list-agreements-endpoint]: https://vippsas.github.io/vipps-recurring-api/#/Agreement%20v2%20endpoints/ListAgreements
[draft-agreement-endpoint]: https://vippsas.github.io/vipps-recurring-api/#/Agreement%20v2%20endpoints/DraftAgreement
[fetch-agreement-endpoint]: https://vippsas.github.io/vipps-recurring-api/#/Agreement%20v2%20endpoints/FetchAgreement
[update-agreement-endpoint]: https://vippsas.github.io/vipps-recurring-api/#/Agreement%20v2%20endpoints/UpdateAgreementPatch
[force-accept-agreement-endpoint]: https://vippsas.github.io/vipps-recurring-api/#/Agreement%20v2%20endpoints/acceptUsingPATCH
[list-charges-endpoint]: https://vippsas.github.io/vipps-recurring-api/#/Charge%20v2%20endpoints/ListCharges
[create-charge-endpoint]: https://vippsas.github.io/vipps-recurring-api/#/Charge%20v2%20endpoints/CreateCharge
[fetch-charge-endpoint]: https://vippsas.github.io/vipps-recurring-api/#/Charge%20v2%20endpoints/FetchCharge
[capture-charge-endpoint]: https://vippsas.github.io/vipps-recurring-api/#/Charge%20v2%20endpoints/CaptureCharge
[cancel-charge-endpoint]: https://vippsas.github.io/vipps-recurring-api/#/Charge%20v2%20endpoints/CancelCharge
[refund-charge-endpoint]: https://vippsas.github.io/vipps-recurring-api/#/Charge%20v2%20endpoints/RefundCharge
[userinfo-endpoint]: https://vippsas.github.io/vipps-recurring-api/#/Userinfo%20Endpoint/getUserinfo
[access-token-endpoint]: https://vippsas.github.io/vipps-recurring-api/#/Access%20Endpoints/getAccessToken