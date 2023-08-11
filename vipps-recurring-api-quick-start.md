<!-- START_METADATA
---
title: Quick start for the Recurring API
sidebar_label: Quick start
sidebar_position: 20
description: Quick steps for getting started with the Recurring API.
toc_min_heading_level: 2
toc_max_heading_level: 5
pagination_next: null
pagination_prev: null
---

import ApiSchema from '@theme/ApiSchema';
import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';

END_METADATA -->

# Quick start

Use the Recurring API to create different types of agreements, get information about agreements, and update and stop agreements.
After creating the agreements, you still need to send the charge requests.
So, you use the Recurring API to create charges for an agreement, get lists of charges for an agreement, cancel or refund charges, or capture reserved charges.

## Before you begin

This document covers the quick steps for getting started with the Recurring API.
You must have already signed up as an organization with Vipps MobilePay and have
your test credentials from the merchant portal, as described in the
[Getting started guide](https://developer.vippsmobilepay.com/docs/getting-started).

**Important:** The examples use standard example values that you must change to
use *your* values. This includes API keys, HTTP headers, reference, etc.

## Your first agreement

### Step 1 - Setup

<Tabs
defaultValue="curl"
groupId="sdk-choice"
values={[
{label: 'curl', value: 'curl'},
{label: 'Postman', value: 'postman'},
]}>
<TabItem value="postman">

**Please note:** Postman is discontinuing their offline version. Use only your test keys and delete them after testing. Ensure that your company allows for cloud use before continuing.

If you wish to use Postman, import the following files:

* [Recurring API Postman collection](/tools/vipps-recurring-v3-api-postman-collection.json)
* [Global Postman environment](https://github.com/vippsas/vipps-developers/blob/master/tools/vipps-api-global-postman-environment.json)

In Postman, tweak the environment with your own values (see
[API keys](https://developer.vippsmobilepay.com/docs/common-topics/api-keys/)):

* `client_id` - Merchant key required for getting the access token.
* `client_secret` - Merchant key required for getting the access token.
* `Ocp-Apim-Subscription-Key` - The subscription key for making API requests.
* `merchantSerialNumber` - The unique ID for your sales unit.
* `mobileNumber` - The phone number for your
   [test user](https://developer.vippsmobilepay.com/docs/test-environment#test-users).

</TabItem>
<TabItem value="curl">

No setup needed :)

</TabItem>
</Tabs>

### Step 2 - Authentication

For all the following, you will need an `access_token` from the
[Access token API](https://developer.vippsmobilepay.com/docs/APIs/access-token-api):
[`POST:/accesstoken/get`][access-token-endpoint].
This provides you with access to the API.

<Tabs
defaultValue="curl"
groupId="sdk-choice"
values={[
{label: 'curl', value: 'curl'},
{label: 'Postman', value: 'postman'},
]}>
<TabItem value="postman">

```bash
Send request Get Access Token
```

</TabItem>
<TabItem value="curl">

```bash
curl https://apitest.vipps.no/accessToken/get \
-H "client_id: YOUR-CLIENT-ID" \
-H "client_secret: YOUR-CLIENT-SECRET" \
-H "Ocp-Apim-Subscription-Key: YOUR-SUBSCRIPTION-KEY" \
-H "Merchant-Serial-Number: 123456" \
-H "Vipps-System-Name: acme" \
-H "Vipps-System-Version: 3.1.2" \
-H "Vipps-System-Plugin-Name: acme-webshop" \
-H "Vipps-System-Plugin-Version: 4.5.6" \
-X POST \
--data ''
```

</TabItem>
</Tabs>

The property `access_token` should be used for all other API requests in the `Authorization` header as the Bearer token.

### Step 3 - Create a minimal agreement

Create an agreement with: [`POST:/agreements`][draft-agreement-endpoint].
When your test mobile number is provided in `phoneNumber`, it will be pre-filled in the form.

Note that `orderId` must be unique for each payment you create.

<Tabs
defaultValue="curl"
groupId="sdk-choice"
values={[
{label: 'curl', value: 'curl'},
{label: 'Postman', value: 'postman'},
]}>
<TabItem value="postman">

Set the `Idempotency-Key-Draft` value in your Postman environment.

```bash
Send request Draft Agreement - Legacy pricing
```

</TabItem>
<TabItem value="curl">

```bash
curl https://apitest.vipps.no/recurring/v3/agreements/ \
-H 'Content-Type: application/json' \
-H "Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1Ni <truncated>" \
-H "Ocp-Apim-Subscription-Key: 0f14ebcab0ec4b29ae0cb90d91b4a84a" \
-H "Merchant-Serial-Number: 123456" \
-H "Vipps-System-Name: acme" \
-H "Vipps-System-Version: 3.1.2" \
-H "Vipps-System-Plugin-Name: acme-webshop" \
-H "Vipps-System-Plugin-Version: 4.5.6" \
-H "Idempotency-key: UNIQUE-IDEMPOTENCY-KEY" \
-X POST \
-d '{
   "interval": {
      "unit" : "WEEK",
      "count": 2
   },
   "pricing": {
      "amount": 1000,
      "currency": "NOK"
   },
   "merchantRedirectUrl": "https://example.com/redirect-url",
   "merchantAgreementUrl": "https://example.com/agreement-url",
   "phoneNumber": "91234567",
   "productName": "Postman test product"
}'
```

</TabItem>
</Tabs>

### Step 4 - Authorize the agreement

Open the `vippsConfirmationUrl` link that is returned in the previous step. It will take you to the
[landing page](https://developer.vippsmobilepay.com/docs/common-topics/landing-page/).
The phone number of your test user should already be filled in, so you only have to click *Next*.

You will be presented with the agreement in the app, where you can authorize and be directed to the specified `merchantRedirectUrl` under a "best effort" policy.

:::note
We cannot guarantee the user will be redirected back to the same browser or session, or that they will at all be redirected back. User interaction can be unpredictable, and the user may choose to fully close the app or browser.
:::

You should now have an active agreement.
Take note of the value included in `agreementId`, as you will need this to access the agreement later.

### Step 5 - Fetch the agreement

To receive the result of the user action, you may poll the status of the agreement with:
[`GET:/agreements/{agreementId}`][fetch-agreement-endpoint].

<Tabs
defaultValue="curl"
groupId="sdk-choice"
values={[
{label: 'curl', value: 'curl'},
{label: 'Postman', value: 'postman'},
]}>
<TabItem value="postman">

```bash
Send request Get Agreement
```

</TabItem>
<TabItem value="curl">

```bash
curl https://apitest.vipps.no/recurring/v3/agreements/UNIQUE-AGREEMENT-ID \
-H "Content-Type: application/json" \
-H "Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1Ni <truncated>" \
-H "Ocp-Apim-Subscription-Key: 0f14ebcab0ec4b29ae0cb90d91b4a84a" \
-H "Merchant-Serial-Number: 123456" \
-H "Vipps-System-Name: acme" \
-H "Vipps-System-Version: 3.1.2" \
-H "Vipps-System-Plugin-Name: acme-webshop" \
-H "Vipps-System-Plugin-Version: 4.5.6" \
-X GET
```

</TabItem>
</Tabs>

If you confirmed the agreement, the status should be ACTIVE in the response.

### Step 6 - Create a charge for the agreement

Create a charge with:
[`POST:/agreements/{agreementId}/charges`][create-charge-endpoint].

Set a unique `orderId` that can be used to access the charge later.
Also, set a unique `Idempotency-Key` value to ensure the charge is not created more than once.

<Tabs
defaultValue="curl"
groupId="sdk-choice"
values={[
{label: 'curl', value: 'curl'},
{label: 'Postman', value: 'postman'},
]}>
<TabItem value="postman">

Ensure that `agreementId` is set to the ID of an ACTIVE agreement.

Set `Idempotency-Key-Create` value.

```bash
Send request Create Charge - Direct capture
```


</TabItem>
<TabItem value="curl">

```bash
curl https://apitest.vipps.no/recurring/v3/agreements/UNIQUE-AGREEMENT-ID/charges \
-H "Content-Type: application/json" \
-H "Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1Ni <truncated>" \
-H "Ocp-Apim-Subscription-Key: 0f14ebcab0ec4b29ae0cb90d91b4a84a" \
-H "Merchant-Serial-Number: 123456" \
-H "Idempotency-Key: 49ca711a-acee-4d01-993b-9487112e1def" \
-H "Vipps-System-Name: acme" \
-H "Vipps-System-Version: 3.1.2" \
-H "Vipps-System-Plugin-Name: acme-webshop" \
-H "Vipps-System-Plugin-Version: 4.5.6" \
-X POST \
-d '{
  "amount": 1000,
  "description": "This payment was charged with Postman.",
  "due": "2023-08-08",
  "retryDays": 0,
  "transactionType": "DIRECT_CAPTURE",
  "orderId": "UNIQUE-ORDERID"
}'
```

</TabItem>
</Tabs>

The status will be `PENDING` until the payment is processed.

A charge must be scheduled a minimum of two days before the payment will occur (it is a minimum one day in the test environment).
See [Direct Capture](vipps-recurring-api.md#direct-capture) for more details about timing.

### Step 7 - Fetch the charge

To get the status of the charge, use:
[`POST:/agreements/{agreementId}/charges/{chargeId}`][fetch-charge-endpoint].

The value for `chargeId` must match what you provided for `orderId`.

<Tabs
defaultValue="curl"
groupId="sdk-choice"
values={[
{label: 'curl', value: 'curl'},
{label: 'Postman', value: 'postman'},
]}>
<TabItem value="postman">

```bash
Send request Get Charge
```

</TabItem>
<TabItem value="curl">

```bash
curl https://apitest.vipps.no/recurring/v3/agreements/UNIQUE-AGREEMENT-ID/charges/UNIQUE-ORDERID \
-H "Content-Type: application/json" \
-H "Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1Ni <truncated>" \
-H "Ocp-Apim-Subscription-Key: 0f14ebcab0ec4b29ae0cb90d91b4a84a" \
-H "Merchant-Serial-Number: 123456" \
-H "Vipps-System-Name: acme" \
-H "Vipps-System-Version: 3.1.2" \
-H "Vipps-System-Plugin-Name: acme-webshop" \
-H "Vipps-System-Plugin-Version: 4.5.6" \
-X GET
```

</TabItem>
</Tabs>

### (Optional) Step 8 - Cancel the charge

To cancel an unpaid charge, use:
[`DEL:/agreements/{agreementId}/charges/{chargeId}`][cancel-charge-endpoint].

<Tabs
defaultValue="curl"
groupId="sdk-choice"
values={[
{label: 'curl', value: 'curl'},
{label: 'Postman', value: 'postman'},
]}>
<TabItem value="postman">

```bash
Send request Cancel Charge
```

</TabItem>
<TabItem value="curl">

```bash
curl https://apitest.vipps.no/recurring/v3/agreements/UNIQUE-AGREEMENT-ID/charges/UNIQUE-CHARGE-ID \
-H "Content-Type: application/json" \
-H "Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1Ni <truncated>" \
-H "Ocp-Apim-Subscription-Key: 0f14ebcab0ec4b29ae0cb90d91b4a84a" \
-H "Merchant-Serial-Number: 123456" \
-H "Idempotency-Key: 86109711-52b5-4d4e-9c0a-8cccf1ff9309" \
-H "Vipps-System-Name: acme" \
-H "Vipps-System-Version: 3.1.2" \
-H "Vipps-System-Plugin-Name: acme-webshop" \
-H "Vipps-System-Plugin-Version: 4.5.6" \
-X DELETE
```

</TabItem>
</Tabs>

### (Optional) Step 9 - Refund a charge

To refund a charge, use:
[`POST:/agreements/{agreementId}/charges/{chargeId}/refund`][refund-charge-endpoint].

<Tabs
defaultValue="curl"
groupId="sdk-choice"
values={[
{label: 'curl', value: 'curl'},
{label: 'Postman', value: 'postman'},
]}>
<TabItem value="postman">

Update the `chargeId` to a charge that has been processed.

```bash
Send request Refund Charge
```

</TabItem>
<TabItem value="curl">

```bash
curl https://apitest.vipps.no/recurring/v3/agreements/UNIQUE-AGREEMENT-ID/charges/UNIQUE-CHARGE-ID \
-H "Content-Type: application/json" \
-H "Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1Ni <truncated>" \
-H "Ocp-Apim-Subscription-Key: 0f14ebcab0ec4b29ae0cb90d91b4a84a" \
-H "Merchant-Serial-Number: 123456" \
-H "Idempotency-Key: 5ad7ec7a-cc6e-44c3-807b-812eb011480a" \
-H "Vipps-System-Name: acme" \
-H "Vipps-System-Version: 3.1.2" \
-H "Vipps-System-Plugin-Name: acme-webshop" \
-H "Vipps-System-Plugin-Version: 4.5.6" \
-X POST \
-d '{
  "amount": 1000,
  "description":"This charge was refunded with Postman"
}'
```

</TabItem>
</Tabs>

### (Optional) Step 10 - Stop an agreement

To stop an agreement,
send [`PATCH:/agreements/{agreementId}`][update-agreement-patch-endpoint]
with `"status": "STOPPED"`.

<Tabs
defaultValue="curl"
groupId="sdk-choice"
values={[
{label: 'curl', value: 'curl'},
{label: 'Postman', value: 'postman'},
]}>
<TabItem value="postman">

Set `agreementId` to the ID of an ACTIVE agreement

```bash
Send request Stop agreement
```

</TabItem>
<TabItem value="curl">

```bash
curl https://apitest.vipps.no/recurring/v3/agreements/UNIQUE-AGREEMENT-ID  \
-H "Content-Type: application/json" \
-H "Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1Ni <truncated>" \
-H "Ocp-Apim-Subscription-Key: 0f14ebcab0ec4b29ae0cb90d91b4a84a" \
-H "Merchant-Serial-Number: 123456" \
-H "Idempotency-Key: 5ad7ec7a-cc6e-44c3-807b-812eb011480a" \
-H "Vipps-System-Name: acme" \
-H "Vipps-System-Version: 3.1.2" \
-H "Vipps-System-Plugin-Name: acme-webshop" \
-H "Vipps-System-Plugin-Version: 4.5.6" \
-X PATCH \
-d '{
  "status": "STOPPED"
}'
```

</TabItem>
</Tabs>

## Next steps

See the [Recurring API guide](./vipps-recurring-api.md) to read about the concepts and details.

For more examples, see the step-by-step instructions in the
[Recurring API Postman guide](vipps-recurring-api-postman-guide.md).


[access-token-endpoint]: https://developer.vippsmobilepay.com/api/access-token#tag/Authorization-Service/operation/fetchAuthorizationTokenUsingPost
[draft-agreement-endpoint]: https://developer.vippsmobilepay.com/api/recurring#tag/Agreement-v3-endpoints/operation/DraftAgreementV3
[agreement-endpoints]: https://developer.vippsmobilepay.com/api/recurring#tag/Agreement-v3-endpoints
[fetch-agreement-endpoint]: https://developer.vippsmobilepay.com/api/recurring#tag/Agreement-v3-endpoints/operation/FetchAgreementV3
[update-agreement-patch-endpoint]: https://developer.vippsmobilepay.com/api/recurring#tag/Agreement-v3-endpoints/operation/UpdateAgreementPatchV3
[create-charge-endpoint]: https://developer.vippsmobilepay.com/api/recurring#tag/Charge-v3-endpoints/operation/CreateChargeV3
[fetch-charge-endpoint]: https://developer.vippsmobilepay.com/api/recurring#tag/Charge-v3-endpoints/operation/FetchChargeV3
[cancel-charge-endpoint]: https://developer.vippsmobilepay.com/api/recurring#tag/Charge-v3-endpoints/operation/CancelChargeV3
[refund-charge-endpoint]: https://developer.vippsmobilepay.com/api/recurring#tag/Charge-v3-endpoints/operation/RefundChargeV3
