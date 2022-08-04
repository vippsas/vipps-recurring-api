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

### Step 4: Run the examples

See the [Recurring API Specifications](https://vippsas.github.io/vipps-recurring-api/#/) for details about the calls.

#### 1. Get access token

Send request `Get Access Token`. This provides you with access to the API.

The access token is valid for 1 hour in the test environment
and 24 hours in the production environment.

#### 2. Test working with an agreement

1. Send `Draft Agreement - Minimal`. This is to demonstrate a simple agreement by using
   [`POST:/v2/agreements`]([draft-agreement-endpoint]).

   The `agreementId` variable is now in the environment of this Postman example and can be used for subsequent calls relating to this agreement.

   Ctrl+click on the link that appears and it will take you to the Vipps landing page.
   Enter your test phone number and complete the agreement authorization in the Vipps app in your mobile test environment.

1. Send `Get Agreement` with `agreementId` for information about this agreement by using
   [`GET:/v2/agreements/{{agreementId}}`][fetch-agreement-endpoint].

1. Send `Fetch Agreements` to get a list of all the agreements by using
   [`GET:/v2/agreements`][list-agreements-endpoint].

1. Send `Update agreement`. This changes the price, product name, product description, and status according to values sent in the body of [`PUT:/v2/agreements/{{agreementId}}`][update-agreement-endpoint]. Send `Get Agreement` and `Fetch Agreements` to see the changes.

1. Open `Stop agreement`. The status is set to `STOPPED` in the body of [`PUT:/v2/agreements/{{agreementId}}`][update-agreement-endpoint].
   Send `Get Agreement` and `Fetch Agreements` to see the changes.


#### 3. Test creating a charge

1. Send `Draft Agreement - Minimal` and approve it.

   The `agreementId` variable is now in the environment.

1. Send `Create Charge - Due tomorrow`. This creates a charge that is due tomorrow.
   Note, that you can't create the charge for the same day as you send the request.

   The `chargeId` variable is set for later use.

1. Send `Get Charge` with `chargeId`. This uses [`GET:/v2/agreements/{{agreementId}}/charges/{{chargeId}}`][fetch-charge-endpoint]
   to get information about the charge.

1. Send `List Charges` which uses [`GET:/v2/agreements/{{agreementId}}/charges`][list-charges-endpoint] to provide a list of all the charges
   for an agreement.


#### 4. Create an agreement with initial charges

You can create more complex types of agreements by modifying the parameters in [`POST:/v2/agreements`][draft-agreement-endpoint].

1. Send `Draft Agreement - Full`.  This demonstrates the addition of an initial charge and a campaign.

   Ctrl+click on the link and complete the authorization.

   When you use this, a charge is automatically created for the initial payment.
   The `agreementId` and `chargeId` are set in the environment.

#### 5. Test creating agreements with initial charges and reserve capture

1. Send `Draft Agreement - Reserved Charge`.  This demonstrates the addition of an initial charge with a transactionType: `RESERVE_CAPTURE`.

   Ctrl+click on the link  and complete the authorization.

   When you use this, a charge is automatically created for the initial payment and you can capture it immediately.
   The `agreementId` and `chargeId` are set in the environment.

1. Send `Capture reserved charge` which uses [`POST:/v2/agreements/{{agreementId}}/charges/{{chargeId}}/capture`][capture-charge-endpoint].



#### X. Getting access to user info

1. Send request `Draft Agreement - Profile flow`. Provide the `scope` object in the [`POST:/v2/agreements`][draft-agreement-endpoint] call. This contains the information types that you want access to, separated by spaces (e.g., "name address email phoneNumber birthDate").

   Here, the `agreementId` and `landing_page_url` are retrieved from the response and set as variables.

1. Once you complete the session, a unique identifier `sub` can be retrieved in the agreement details.
   Send request `Get Agreement` for information about this payment by using [`GET:/v2/agreements/{{agreementId}}`][fetch-agreement-endpoint].

   In this example, `sub` is retrieved from the response and set as a variable.

1. Send request `Get Userinfo`. This uses [`GET:/vipps-userinfo-api/userinfo/{sub}`][userinfo-endpoint] with the `sub` variable from the previous call.

See [Userinfo](vipps-recurring-api.md#userinfo) for more information.

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