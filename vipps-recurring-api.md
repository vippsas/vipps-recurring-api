# Vipps Recurring API

The Vipps Recurring API delivers recurring payment functionality for a merchant
to create a payment agreement with a customer for fixed interval payments.
When the agreement is accepted by the end user the merchant can send charges
that will be automatically processed on the due date.

**IMPORTANT:** The Vipps Recurring API is available for existing customers that
have "Vipps på Nett" and a direct integration with the
[Vipps eCom API](https://github.com/vippsas/vipps-recurring-api)
and have completed some additional KYC checks required by Finanstilsynet.
Vipps is required to perform some extra compliance checks before
activating the Vipps Recurring API, please order Vipps Faste betalinger on
[portal.vipps.no](https://portal.vipps.no)
to get access to the Recurring API in production.

See: [How it works](vipps-login-recurring-howitworks.md).

These Swagger/OpenAPI representations may be useful to get a quick overview:
* [Swagger](https://vippsas.github.io/vipps-recurring-api/)
* [ReDoc](https://vippsas.github.io/vipps-recurring-api/redoc.html)
* [Shins](https://vippsas.github.io/vipps-recurring-api/shins/index.html)

See: Vipps Recurring [GitHub repository](https://github.com/vippsas/vipps-recurring-api/),
with [Postman collection](tools/),
[Integration checklist](vipps-recurring-api-checklist.md),
[FAQ](vipps-recurring-api-faq.md).

API version: 1.0.0.

Document version 2.3.19.

## Table of Contents

- [Terminology](#terminology)
- [Flow diagram](#flow-diagram)
- [Call by call guide](#call-by-call-guide)
  - [Direct capture](#direct-capture)
  - [Reserve capture](#reserve-capture)
  - [Vipps screenshots](#vipps-screenshots)
- [API endpoints](#api-endpoints)
- [Authentication](#authentication)
- [Optional Vipps HTTP headers](#optional-vipps-http-headers)
- [orderId recommendations](#orderid-recommendations)
- [Agreements](#agreements)
  - [Create an agreement](#create-an-agreement)
  - [Accept an agreement](#accept-an-agreement)
  - [Intervals](#intervals)
  - [Initial charge](#initial-charge)
  - [Campaigns](#campaigns)
  - [Retrieve an agreement](#retrieve-an-agreement)
- [Charges](#charges)
  - [Create a charge](#create-a-charge)
  - [Amount changes](#amount-changes)
  - [Charge descriptions](#charge-descriptions)
  - [Charge times](#charge-times)
  - [Charge retries](#charge-retries)
  - [Retrieve a charge](#retrieve-a-charge)
- [Manage charges and agreements](#manage-charges-and-agreements)
  - [Agreement states](#agreement-states)
  - [Update an agreement](#update-an-agreement)
  - [Pause an agreement](#pause-an-agreement)
  - [Stop an agreement](#stop-an-agreement)
  - [Charge states](#charge-states)
  - [Charge failure reasons](#charge-failure-reasons)
- [Userinfo](#userinfo)
  - [Scope](#scope)
  - [Userinfo call by call guide](#userinfo-call-by-call-guide)
  - [Example calls](#example-calls)
  - [Userinfo call](#userinfo-call)
  - [Consent](#consent)
- [Recurring agreements with variable amount](#recurring-agreements-with-variable-amount)
  - [How it works](#how-it-works)
    - [Create agreement](#create-agreement)
    - [Get agreement](#get-agreement)
    - [Change suggestedMaxAmount](#change-suggestedmaxamount)
    - [Create charge](#create-charge)
    - [Charge amount higher than the users max amount](#charge-amount-higher-than-the-users-max-amount)
- [Skip landing page](#skip-landing-page)
- [HTTP responses](#http-responses)
- [Rate limiting](#rate-limiting)
- [Partner keys](#partner-keys)
- [Polling guidelines](#polling-guidelines)
- [Timeouts](#timeouts)
  - [Using a phone](#using-a-phone)
  - [Using a laptop/desktop](#using-a-laptopdesktop)
- [Authentication and authorization](#authentication-and-authorization)
- [Testing](#testing)
- [Recommendations regarding handling redirects](#recommendations-regarding-handling-redirects)
- [When to use campaigns or initial charge](#when-to-use-campaigns-or-initial-charge)
  - [Normal agreement](#normal-agreement-flow)
  - [Initial charge](#initial-charge-flow)
  - [Campaign](#campaign)
  - [Initial charge and campaign](#initial-charge-and-campaign)
- [Questions?](#questions)

## Terminology

| Term        |  Description                                    |
| ----------- | ----------------------------------------------- |
| Agreement   | A payment subscription with a set of parameters that a customer agrees to  |
| Charge      | A single payment within an agreement |
| Idempotency | The property of endpoints to be called multiple times without changing the result after the initial request. |

## Flow diagram

This diagram shows a simplified payment flow:

![Recurring agreement flow](/images/Recurring-createagreement.svg)

## Call by call guide

There are two happy-flows based on how the sale unit is set up:
One for "direct capture" and one for "reserve capture".
This is specified with the `transactionType`, and for "direct capture"
the sale unit must be configured for this by Vipps.
See the eCom FAQ for the difference:
[What is the difference between "Reserve Capture" and "Direct Capture"?](https://github.com/vippsas/vipps-ecom-api/blob/master/vipps-ecom-api-faq.md#what-is-the-difference-between-reserve-capture-and-direct-capture)

**Note:** Vipps will *only* perform a payment transaction on an agreement that
the merchant has created a charge for with
[`POST:/recurring/v2/agreements/{agreementId}/charges`](https://vippsas.github.io/vipps-recurring-api/#/Charge%20Endpoints/createCharge).
You can also [Manage charges and agreements](#manage-charges-and-agreements).

### Direct capture

For a `"transactionType": "DIRECT_CAPTURE"` setup, the normal flow would be:

1. Create a (draft) agreement: [`POST:/recurring/v2/agreements`][draft-agreement-endpoint].
   The user can now confirm the agreement in Vipps (the app). See [Create a new agreement](#create-an-agreement).
2. The user approves the agreement in Vipps:
   This will result in a capture of the initial charge (if one was defined in the first step).
3. Retrieve the (hopefully approved) agreement:
   [`GET:/recurring/v2/agreements/{agreementId}`](https://vippsas.github.io/vipps-recurring-api/#/Agreement%20Endpoints/getAgreement).
   See [Retrieve an agreement](#retrieve-an-agreement).
   **Note:** At this point the agreement will be `ACTIVE` if the user completed step 2.
4. For all future charges, you must create a charge:
   [`POST:/recurring/v2/agreements/{agreementId}/charges`](https://vippsas.github.io/vipps-recurring-api/#/Charge%20Endpoints/createCharge).
   See [Create a charge](#create-a-charge).
   Based on the `dueDate` set in the request, our batch job will try to process the charge on that day.
   If for some reason, a charge fails to be processed,
   we will retry up to twice a day for the amount of days provided by the `retryDays` value.
   We recommend at least 2 days retry.

### Reserve capture

For a `"transactionType": "RESERVE_CAPTURE"` setup, the normal flow would be:

1. Create a (draft) agreement: [`POST:/recurring/v2/agreements`][draft-agreement-endpoint].
   The user can now confirm the agreement in Vipps (the app). See [Create a new agreement](#create-an-agreement).
2. The user approves the agreement in Vipps:
   This will result in a capture of the initial charge (if one was defined in the first step).
3. Retrieve the (hopefully approved) agreement:
   [`GET:/recurring/v2/agreements/{agreementId}`](https://vippsas.github.io/vipps-recurring-api/#/Agreement%20Endpoints/getAgreement).
   See [Retrieve an agreement](#retrieve-an-agreement).
   **Note:** At this point the agreement will be `ACTIVE` if the user completed step 2.
4. If there is a product that is shipped to the customer, the initial charge should be captured at this point.
   Capture the charge:
   [`POST:/recurring/v2/agreements/{agreementId}/charges/{chargeId}/capture`](https://vippsas.github.io/vipps-recurring-api/#/Charge%20Endpoints/captureCharge)
   If there is no product being shipped, or a need to provide access at a later point - the merchant should change the
   merchant sale unit setup to use `DIRECT CAPTURE` instead.
5. For all future charges, you must create a charge:
   [`POST:/recurring/v2/agreements/{agreementId}/charges`](https://vippsas.github.io/vipps-recurring-api/#/Charge%20Endpoints/createCharge).
   See [Create a charge](#create-a-charge).
   Based on the `dueDate` set in the request, our batch job will try to process the charge on that day.
   If for some reason, a charge fails to be processed,
   we will retry up to twice a day for the amount of days provided by the `retryDays` value.
   We recommend at least 2 days retry.

### Vipps screenshots

![Vipps screenshots](images/RecurringOverview.png)

## API endpoints

| Operation           | Description         | Endpoint          |
| -------------------- | ------------------- | ----------------- |
| [Create an agreement](#create-an-agreement)  | Create a new, draft agreement.  | [`POST:/recurring/v2/agreements`][draft-agreement-endpoint]  |
| [Retrieve an agreement](#retrieve-an-agreement)  | Retrieve the details of an agreement.  |  [`GET:/recurring/v2/agreements/{agreementId}`](https://vippsas.github.io/vipps-recurring-api/#/Agreement%20Endpoints/getAgreement) |
| [Update an agreement]()   |  Update an agreement with new details. |  [`PATCH:/recurring/v2/agreements/{agreementId}`](https://vippsas.github.io/vipps-recurring-api/#/Agreement%20Endpoints/updateAgreement) |
| [Stop an agreement](#stop-an-agreement) | Update the status to `STOPPED`.  | [`PATCH:/recurring/v2/agreements/{agreementId}`](https://vippsas.github.io/vipps-recurring-api/#/Agreement%20Endpoints/updateAgreement)  |
| List all charges  | Get all charges for an agreement.  | [`GET:/recurring/v2/agreements/{agreementId}/charges`](https://vippsas.github.io/vipps-recurring-api/#/Charge%20Endpoints/listCharges)  |
| [Create a charge](#create-a-charge)   |  Create a new charge for an agreement. |  [`POST:/recurring/v2/agreements/{agreementId}/charges`](https://vippsas.github.io/vipps-recurring-api/#/Charge%20Endpoints/createCharge) |
| Retrieve a charge  | Retrieve all details of a charge.  | [`GET:/recurring/v2/agreements/{agreementId}/charges/{chargeId}`](https://vippsas.github.io/vipps-recurring-api/#/Charge%20Endpoints/getCharge)  |
| Capture a charge  | Each charge must first be created, then captured.  | [`POST:/recurring/v2/agreements/{agreementId}/charges/{chargeId}/capture`](https://vippsas.github.io/vipps-recurring-api/#/Charge%20Endpoints/captureCharge)  |
| Cancel a charge | Cancel an existing charge before the user is charged. | [`DELETE:/recurring/v2/agreements/{agreementId}/charges/{chargeId}`](https://vippsas.github.io/vipps-recurring-api/#/Charge%20Endpoints/cancelCharge) |
| Refund a charge    | Refund a charge that has been performed.   | [`POST:/recurring/v2/agreements/{agreementId}/charges/{chargeId}/refund`](https://vippsas.github.io/vipps-recurring-api/#/Charge%20Endpoints/refundCharge)  |

See [Authentication and authorization](#authentication-and-authorization).

See the Postman collection available on https://github.com/vippsas/vipps-recurring-api
for en easy way to test the API.

## Authentication

All Vipps API calls are authenticated and authorized with an access token
(JWT bearer token) and an API subscription key:

| Header Name | Header Value | Description |
| ----------- | ------------ | ----------- |
| `Authorization` | `Bearer <JWT access token>` | Type: Authorization token. This obtained as described in [Getting started](https://github.com/vippsas/vipps-developers/blob/master/vipps-getting-started.md): [Get an access token](https://github.com/vippsas/vipps-developers/blob/master/vipps-getting-started.md#get-an-access-token) |
| `Ocp-Apim-Subscription-Key` | Base 64 encoded string | The subscription key for this API. This is available on [portal.vipps.no](https://portal.vipps.no). |

For more information about how to obtain an access token and all details around this, please see:
[Quick overview of how to make an API call](https://github.com/vippsas/vipps-developers/blob/master/vipps-getting-started.md#quick-overview-of-how-to-make-an-api-call)
in the
[Getting started guide](https://github.com/vippsas/vipps-developers/blob/master/vipps-getting-started.md).

## Optional Vipps HTTP headers

We recommend using the following _optional_ HTTP headers for all requests to the
Vipps Recurring API. These headers provide useful metadata about the merchant's system,
which help Vipps improve our services, and also help in investigating problems.

| Header                        | Description                                  | Example value        |
| ----------------------------- | -------------------------------------------- | -------------------- |
| `Merchant-Serial-Number`      | The merchant serial number                   | `123456`             |
| `Vipps-System-Name`           | The name of the ecommerce solution           | `woocommerce`        |
| `Vipps-System-Version`        | The version number of the ecommerce solution | `5.4`                |
| `Vipps-System-Plugin-Name`    | The name of the ecommerce plugin             | `vipps-woocommerce`  |
| `Vipps-System-Plugin-Version` | The version number of the ecommerce plugin   | `1.4.1`              |

These headers are required for plugins and partners and sent by the recent versions of
[the official Vipps plugins](https://github.com/vippsas/vipps-developers#plugins)
and we recommend all customers with direct integration with the API to also do so.

## orderId recommendations

A `orderId` must be unique.

If you ever have a problem that requires us to search in our logs, we need
`orderId`s that are "unique enough" to actually find them. An `orderId` that
is just a number may not be possible to find.

While the minimum length for `orderId` _technically_ is just one character,
we strongly recommend using at least 6 characters, and a combination of numbers
and characters.

The maximum length of an `orderId` is 50, and can contain alphanumeric characters and dashes:
a-z, A-Z, 0-9, -. Example: `2c2a838c-5a88-4b3a-ab9f-e884b92b9bec`.

We _strongly_ recommend to use `orderId` format that makes it easy to
search for them in logs. This means that `abc123def456` is a better
format than `123456`.

Leading zeros should be avoided, as some applications (like Excel)
tend to remove them, and this may cause misunderstandings.

With multiple sale units, prefixing the `orderId` with the MSN
for each sale unit is recommended: If the MSN is `654321`, the
`orderId`s could start at `654321000000000001` and increment by 1
for each order, or some similar, unique and readable pattern.

## Agreements

An agreement is between the Vipps user and the merchant.
Think of it as a subscription.
An agreement has payments, called charges.
See [Charges](#charges).

### Create an agreement

Create an agreement:
[`POST:/recurring/v2/agreements`][draft-agreement-endpoint].

This code illustrates how to create an agreement:

[`POST:/recurring/v2/agreements`][draft-agreement-endpoint]

```json
{
  "currency": "NOK",
  "customerPhoneNumber":"90000000",
  "interval": "MONTH",
  "intervalCount": 1,
  "isApp": false,
  "merchantRedirectUrl": "https://example.com/confirmation",
  "merchantAgreementUrl": "https://example.com/my-customer-agreement",
  "price": 49900,
  "productDescription": "Access to all games of English top football",
  "productName": "Premier League subscription"
}
```

**Note:** To create agreements with support for variable amounts on charges, see [Recurring agreements with variable amount](#Recurring-agreements-with-variable-amount).

The `merchantAgreementUrl` is a link to a "My page", where the customer
can manage the agreement: Change, pause, cancel, etc.
Vipps does not offer any form of agreement management, as this may be
quite complex operations, like changing subscription types,
temporary address change, etc.  
The URL is opened in the standard web browser.  
The integrator ***must*** implement such functionality for the customer to manage the agreement in their system.

**Please note:** If the user closes Vipps before the redirect is done,
the `merchantRedirectUrl` will not be used. It is therefore important that you
actively check the payment with
[`GET:/recurring/v2/agreements/{agreementId}](https://vippsas.github.io/vipps-recurring-api/#/Agreement%20Endpoints/getAgreement).

The `merchantAgreementUrl` is just a normal link to a page where the customer
can log in and manage the agreement.
We do not have any specific requirements for the security of the page, but
we strongly recommend to use
[Vipps Logg Inn](https://www.vipps.no/produkter-og-tjenester/bedrift/logg-inn-med-vipps/logg-inn-med-vipps/)
so the user does not have to use a username and password, but is logged
in automatically through Vipps. See the
[API documentation](https://github.com/vippsas/vipps-login-api)
for more details.

The request parameters have the following size limits
(see [`POST:/recurring/v2/agreements`][draft-agreement-endpoint] for more details):

* `productName`: Max length 45 characters
* `productDescription`: Max length 100 characters
* `price`: Greater than 100, meaning 1 NOK.

Agreements may be initiated with or without an [initial charge](#initial-charge),

The agreement price, and the amount for the initial charge, is given in øre,
the centesimal subdivision of the Norwegian kroner (NOK).
There are 100 øre in 1 krone.

| # | Agreement      | Description                                                                          |
|:--|:-----------|:-------------------------------------------------------------------------------------|
| 1 | `Agreement starting now`  | Agreement with an `initialcharge` that uses `DIRECT_CAPTURE` will only be `active` if the initial charge is processed successfully |
| 2 | `Agreement starting in future`  | Agreement without an `initialcharge`, or with `initialcharge` that uses `RESERVE_CAPTURE` can be approved but no payment will happen until the first charge is provided |

The response contains an `agreementResource`, a `vippsConfirmationUrl` and an `agreementId`.
This `agreementResource` is a complete URL for performing a
[`GET:/recurring/v2/agreements/{agreementId}`](https://vippsas.github.io/vipps-recurring-api/#/Agreement%20Endpoints/getAgreement)
request.

The `vippsConfirmationUrl` should be used to redirect the
user to the Vipps landing page in a desktop flow (with `https://`),
or to Vipps in a mobile flow (with `vipps://`), where the
user can then approve the agreement.

### Accept an agreement

[`POST:/recurring/v2/agreements`][draft-agreement-endpoint] will return the following JSON structure.

```json
{
  "vippsConfirmationUrl": "https://api.vipps.no/dwo-api-application/v1/deeplink/vippsgateway?v=2/token=eyJraWQiOiJqd3RrZXkiLCJhbGciOiJSUzI1NiJ9.eyJzdWIiOiJmMDE0MmIxYy02YjI",
  "agreementResource": "https://api.vipps.no-recurring/v2/agreements/agr_TGSuPyV",
  "agreementId": "agr_TGSuPyV"
}
```

The `vippsConfirmationUrl` should be used to redirect the user to the Vipps landing
page. The user can then confirm their identity, and receive a prompt to accept the
agreement within Vipps.

The `isApp` property can be used to receive a deeplink URL, which in a mobile context,
can be used to perform an app-switch, which removes the landing page step. This will only
work if the user has Vipps installed on the same device as they are initiating
the agreement from.

### Intervals

Intervals are defined with an interval type `YEAR`, `MONTH`, `WEEK`, or `DAY` and frequency as a count.

Example for a bi-weekly subscription:
```json
{
  "interval": "WEEK",
  "intervalCount": 2
}
```

Example for a quarterly subscription
```json
{
  "interval": "MONTH",
  "intervalCount": 3
}
```

Examples for a yearly subscription
```json
{
  "interval": "YEAR",
  "intervalCount": 1
}
```
OR
```json
{
  "interval": "MONTH",
  "intervalCount": 12
}
```

Example for a subscription every 100th day:
```json
{
  "interval": "DAY",
  "intervalCount": 100
}
```

### Initial charge

**Please note:** Use
[Campaigns](#campaigns)
if the subscription is cheaper in the beginning than the normal price later.
If you use `initialcharge` for campaigns, users will be confused by how it appears in Vipps,
as it looks like the full price period starts immediately.

Initial charge will be performed if the `initialcharge` is provided when
creating an agreement. If there is no initial charge: Don't sent `initialcharge`
when creating the new agreement.

Unlike regular (or `RECURRING`) charges, there is no price limit on an `initialCharge`.
This allows for products to be bundled with agreements as one transaction
(for example a phone). The user will be clearly informed when an `initialCharge`
is included in the agreement they are accepting.

See [Charge Titles](#charge-descriptions) for explanation of how the charge description
is shown to the user.

The initial charge has two forms of transaction, `DIRECT_CAPTURE` and `RESERVE_CAPTURE`.
See:
[What is the difference between "Reserve Capture" and "Direct Capture"?](https://github.com/vippsas/vipps-ecom-api/blob/master/vipps-ecom-api-faq.md#what-is-the-difference-between-reserve-capture-and-direct-capture)
in the eCom FAQ.

`DIRECT_CAPTURE` processes the payment immediately, while `RESERVE_CAPTURE`
reserves the payment for capturing at a later date. `RESERVE_CAPTURE` must be
used when selling physical goods bundled with an agreement - such as a phone
when subscribing to an agreement.

This example shows the same agreement as above, with an `initialCharge`
of 499 NOK:

```json
{
  "currency": "NOK",
  "customerPhoneNumber": "90000000",
  "initialCharge": {
     "amount": 49900,
     "currency": "NOK",
     "description": "Premier League subscription",
     "transactionType": "DIRECT_CAPTURE"
  },
  "interval": "MONTH",
  "intervalCount": 1,
  "isApp": false,
  "merchantRedirectUrl": "https://example.com/confirmation",
  "merchantAgreementUrl": "https://example.com/my-customer-agreement",
  "price": 49900,
  "productDescription": "Access to all games of English top football",
  "productName": "Premier League subscription"
}
```

Change the `transactionType` field to `RESERVE_CAPTURE` to reserve the initial charge.

```json
{
  "initialCharge": {
    "transactionType": "RESERVE_CAPTURE",
    "amount": 19900,
    "currency": "NOK",
    "description": "Phone"
  }
}
```

A reserved charge can be captured with
[`POST:/recurring/v2/agreements/{agreementId}/charges/{chargeId}/capture`](https://vippsas.github.io/vipps-recurring-api/#/Charge%20Endpoints/captureCharge)
when the product is shipped.

### Campaigns

A campaign in recurring is a period where the price is lower than usual, and
this is communicated to the customer with the original price shown for comparison.
Campaigns can not be used in combination with variable amount, see more about variable amount [here](#Recurring-agreements-with-variable-amount).

![Campaign example](images/CampaignExample.PNG)

In order to start a campaign the campaign field has to be added either to the agreement draft
[`POST:/recurring/v2/agreements`][draft-agreement-endpoint]
for a campaign in the start of an agreement or update an agreement
[`PATCH:/recurring/v2/agreements/{agreementId}`](https://vippsas.github.io/vipps-recurring-api/#/Agreement%20Endpoints/updateAgreement)
for an ongoing agreement. When adding a campaign
while drafting a new agreement the start date is ignored and the current
date-time is used. All dates must be in date-time format as according to
[RFC-3999](https://www.ietf.org/rfc/rfc3339.txt).

```json
{
  "campaign": {
    "start": "2019-05-01T00:00:00Z",
    "end": "2019-06-01T00:00:00Z",
    "campaignPrice": 49900
  }
}
```

| Field         | Description                                 |
| ------------------- | ------------------------------------------- |
| `start`            | Start date of campaign offer, if you are creating a agreement this is set to default now, and not an available variable  |
| `end`            | End date of campaign offer, can not be in the past |
| `campaignPrice`       | The price that will be shown for comparison   |

### Retrieve an agreement

A newly created agreement will be in status `PENDING` for 5 minutes before it expires.
If the customer approves the agreement, and the initialCharge (if provided) is successfully
processed, the agreement status will change to `ACTIVE`.

The approved agreement is retrieved from
[`GET:/recurring/v2/agreements/{agreementId}`](https://vippsas.github.io/vipps-recurring-api/#/Agreement%20Endpoints/getAgreement)
with `"status":"ACTIVE"` when the customer has approved the agreement.

See [Agreement states](#agreement-states).

This is an example response from a call to
[`GET:/recurring/v2/agreements/{agreementId}`](https://vippsas.github.io/vipps-recurring-api/#/Agreement%20Endpoints/getAgreement):

```json
{
  "id": "agr_5kSeqzFAMkfBbc",
  "start": "2018-08-22T13:00:00Z",
  "stop": null,
  "status": "ACTIVE",
  "productName": "Premier League subscription",
  "price": 49900,
  "productDescription": "Access to all games of English top football",
  "interval": "MONTH",
  "intervalCount": 1,
  "currency": "NOK",
  "campaign": null
}
```

## Charges

An [agreement](#agreements) has payments, called charges.

### Create a charge

*Recurring has functionality to charge a variable amount each interval, more information about recurring agreements with variable amount can be found [here](#Recurring-agreements-with-variable-amount).*

Charge the customer for each period with
[`POST:/recurring/v2/agreements/{agreementId}/charges`](https://vippsas.github.io/vipps-recurring-api/#/Charge%20Endpoints/createCharge).

Each specific charge on an agreement must be scheduled by the merchant, a
minimum of two days before the payment will occur (it is minimum one day in the test environment).

Example: If the charge is _created_ on the 25th, the earliest the charge can be
_made_ is the 27th (25+2). This is so that the user can be informed about the
upcoming charge. The user is only shown one charge per agreement, in order to
not overwhelm the user when doing daily or weekly charges.

Create a charge for a given agreement. `due` will define for which date
the charge will be performed. This date has to be at a minimum two days in the
future (it is minimum one day in the test environment), and all charges `due` in
30 days or less are visible for users in Vipps.

An optional _and recommended_ `orderId` field can be set in the request.
If used this will be the id used to identify the charge throughout its payment
history, including in settlement files.

This `orderId` must be unique across all Recurring and eCom
transactions for the given `merchantSerialNumber`.
See [orderId recommendations](#orderid-recommendations).

If the field is not given a unique id will be generated in the form `chr-xxxxxxx` (where each x is an alphanumeric character).

### Amount changes

The `amount` of a charge is flexible and does not have to match the
`price` of the agreement.

A limit is in place however, which is 5 times the agreement `price`.
For example, in the agreement
[above](#retrieve-an-agreement)
a limit of 2495 NOK (499 x 5) would be in place. If this limit becomes a
hindrance the agreement `price` can be [updated](#update-an-agreement).

**Note:** Although it is _technically_ possible to increase the price 10
times, we **strongly** recommend to be as user-friendly as possible, and
to make sure the user understands any changes and get updated information.

### Charge descriptions

When charges are shown to users in Vipps, they will have a title, and a
description. The title of a charge is derived directly from
`{agreement.ProductName}` whereas the description is set per charge, i.e.
`{charge.description}`. For example, a charge on an agreement with product
name *Premier League subscription* with description *October* would look like
the following screenshot:

![Charge description example](images/charge_descriptions_example.png)

When the charge is processed, the payment will show up in the users's payment
history. In the payment history a charge from Vipps recurring payment will have
a description with follow format `{agreement.ProductName} - {charge.description}`.

[`POST:/recurring/v2/agreements/{agreementId}/charges`](https://vippsas.github.io/vipps-recurring-api/#/Charge%20Endpoints/createCharge)

```json
{
  "amount": 49900,
  "currency": "NOK",
  "description": "October",
  "due": "2018-09-01",
  "retryDays": 5
}
```

**Note:** `description` cannot be longer than 45 characters.

### Charge times

Charge _attempts_ are primarily made two times during the day: 07:00 and 15:00 UTC.
The processing of charges typically takes around one hour, however this varies and we do not guarantee any time.
This is the same both for our production and test environment.
Subsequent attempts are made according to the `retryDays` specified.

**Note:** Payments _might_ get processed any time during the day (07:00 UTC - 23:59 UTC) due to special circumstances requiring it.

**Note:** Since payments _can_ be processed any time (07:00UTC - 23:59 UTC) it is advisable to fetch the charge at/after 00:00 UTC the day after the last retry day to be sure you get the last status.

### Charge retries

Vipps will retry the charge for the number of days specified in `retryDays`. The maximum number of retryDays is currently set to 14 days

So if `retryDays=2` that would mean a maximum of 6 retries:
Two on the initial day, and two each for the subsequent days.

If `retryDays=0` it will try two times on the initial day.

Be aware that if you check the status of the charge within the retry period, it
might have status `FAILED`, also after the first attempt on the first and only day.

If a charge fails, you will get information about the reason:
[Charge failure reasons](https://github.com/vippsas/vipps-recurring-api/blob/master/vipps-recurring-api.md#charge-failure-reasons).

### Retrieve a charge

A charge can be retrieved with
[`GET:/recurring/v2/agreements/{agreementId}/charges/{chargeId}`](https://vippsas.github.io/vipps-recurring-api/#/Charge%20Endpoints/getCharge).

Example response:

```json
{
  "amount": 39900,
  "amountRefunded": 39900,
  "description": "Premier League subscription: September",
  "due": "2019-06-01T00:00:00Z",
  "id": "chg_WCVbcAbRCmu2zk",
  "status": "PENDING",
  "transactionId": "5001419121",
  "type": "RECURRING",
  "failureReason": "insufficient_funds",
  "failureDescription": "Payment was declined by the payer bank due to lack of funds"
}
```

## Manage charges and agreements

It is the merchant's responsibility to manage and update charges and agreements,
and to use the API to make sure everything is in sync.

### Agreement states

| # | State      | Description                                                                          |
|:--|:-----------|:-------------------------------------------------------------------------------------|
| 1 | `PENDING`  | Agreement has been created, but not approved by the user in Vipps yet |
| 2 | `ACTIVE` | The agreement has been confirmed by the end user in Vipps and can receive charges |
| 3 | `STOPPED`  | Agreement has been stopped, either by the merchant by [`PATCH:/recurring/v2/agreements/{agreementId}`](https://vippsas.github.io/vipps-recurring-api/#/Agreement%20Endpoints/updateAgreement), or by the user by cancelling or rejecting the agreement. |
| 4 | `EXPIRED` | The user did not accept, or failed to accept (due to processing an `initialCharge`), the agreement in Vipps |

### Update an agreement

A merchant can update an agreement by calling
[`PATCH:/recurring/v2/agreements/{agreementId}`](https://vippsas.github.io/vipps-recurring-api/#/Agreement%20Endpoints/updateAgreement).
The following properties are available for updating:

```json
{
  "productName": "A new name",
  "productDescription": "A new description",
  "price": 25000,
  "status": "ACTIVE",
  "campaign": {
    "start": "2019-10-01T00:00:00Z",
    "end": "2019-12-01T00:00:00Z",
    "campaignPrice": 10000
  },
  "suggestedMaxAmount": 300000
}
```

**Please note:** As a `PATCH` operation all parameters are optional. However
when setting an agreement status to `STOPPED` no other changes are allowed.
Attempts at changing other properties while stopping an agreement will result
in a `400 Bad Request` response.

### Pause an agreement

If there should be a pause in an agreement, like a temporary stop of a
subscription: Simply do not create any charges during the pause.

We recommended not to set the agreement status to `STOPPED`, but to update
the `productDescription` field of the agreement so the user can see that the
subscription is paused in Vipps.

### Stop an agreement

When a user notifies the merchant that they want to cancel a subscription or
service, the merchant must ensure that the status of the recurring agreement is
set to `STOPPED` at a suitable time.

We recommend that the recurring agreement remains `ACTIVE` for as long as the
user has access to the service.    
For example; if the user cancels their subscription, but they are still able to
use the service until the end of the billing cycle, the agreement should only be
set to `STOPPED` at the end of the billing cycle. In this case we also recommend
updating the `productDescription` field of the agreement so that the user can see
that the subscription is cancelled or due to be cancelled at a given time.

Since `STOPPED` agreements cannot be reactivated, a benefit of waiting until
the "end of service" before setting the agreement status to `STOPPED` is that
the merchant will be able to reactivate the user's subscription without having
to set up a new agreement.

### Charge states

![Recurring charge states](images/Recurring-chargestates-new.svg)

The illustration above is a simplification. This table has all the details for
the charge states returned by
[`GET:/recurring/v2/agreements/{agreementId}/charges/{chargeId}`](https://vippsas.github.io/vipps-recurring-api/#/Charge%20Endpoints/getCharge):

| # | State      | Description                                                                          |
|:--|:-----------|:-------------------------------------------------------------------------------------|
| 1 | `PENDING`  | The charge has been created, but is not yet visible to the user in Vipps. |
| 2 | `DUE`      | The charge is now visible in Vipps and will be processed on the due date. |
| 3 | `CHARGED`  | The charge has been completed. |
| 4 | `FAILED`   | The charge has failed for some reason, i.e. expired card, insufficient funds, etc. Read the [Charge failure reasons](https://github.com/vippsas/vipps-recurring-api/blob/master/vipps-recurring-api.md#charge-failure-reasons) section for more details. |
| 5 | `REFUNDED` | The charge has been refunded. The timeframe for issuing a refund is 365 days from the date of capture. |
| 6 | `PARTIALLY_REFUNDED`| A part of the captured amount has been refunded. |
| 7 | `RESERVED` | An initial charge with `transactionType` set to `RESERVE_CAPTURE` changes state to `CHARGED` when captured successfully. |
| 8 | `CANCELLED` | The charge has been cancelled. |
| 9 | `PROCESSING` | The charge is currently being processed by Vipps. Normal processing takes less than 1 second, but in some cases they can stay in this status for several minutes |

### Example charge flows

Scenario: The user does not have funds and `retryDays = 0`,
`PENDING` -> `DUE` -> `PROCESSING` -> `DUE` -> `PROCESSING` -> `FAILED`

Scenario: The user does not have funds on the first attempt, but the second attempt is successful:
`PENDING` -> `DUE` -> `PROCESSING` -> `DUE` -> `PROCESSING` -> `CHARGED`

**Please note:** Since charges are polled by the merchant, it is possible that
the charge status appears to "skip" a transition, e.g. moving directly from
`PROCESSING` to `CHARGED`, or even from `PROCESSING` to `REFUNDED`
depending on your systems. `PROCESSING` is not normally seen, as a charge usually
only has this status for a very brief period of time, but merchants must
still make sure they can handle this status.

### Charge failure reasons

When fetching a charge through the API, you can find two fields in the response
body to identify why the charge failed `failureReason` and `failureDescription`

**Important:** This is an experimental feature. The design has not been finalized
and it only applies to Vipps Recurring API. You may use `failureReason` and
`failureDescription` when the fields are available in the response, but do
not depend on them, and be aware that they may only contain `None` if there
are no further details available for the failure.

An example from a response:

```json
{
  "status": "FAILED",
  "type": "RECURRING",
  "failureReason": "insufficient_funds",
  "failureDescription": "Payment was declined by the payer bank due to lack of funds"
}
```

Here is a list of possible values for `failureReason`, their respective descriptions and possible actions that the user/merchant could take.

| Reason | Description | Action |
| ---- | ----------- | ------ |
| insufficient_funds | Payment was declined by the payer bank due to lack of funds. | User must either add funds to the card to cover the difference between the amount to be paid. Alternatively they can change to another, or add a new, payment source that is adequately funded to complete the transaction. |
| invalid_card | The user tried to pay using a card that has either expired or is disabled by the issuer. | User must change, or add a new, payment source on the agreement in Vipps. |
| verification_required | Payment declined because the issuing bank requires verification. | Ask the user to change, or add a new, payment source on their agreement in Vipps. Alternatively removing and then adding the card might solve the issue. |
| invalid_payment_source | The provided payment source is disabled or does not exist. | User must change payment source for the agreement. |
| charge_amount_too_high | Amount is higher than the users specified max amount | The user have a lower `maxAmount` on the variableAmount agreement than the amount of the charge. The user must update their `maxAmount` on the agreement for the charge to be processed.
| internal_error | Internal Error / Something went wrong | The error could not be identified as one of the above. Try to create the charge again, changing or adding payment sources on the agreement, or contact Vipps for more information. |

## Userinfo

Vipps offers the possibility for merchants to ask for the user's profile information as part of the payment flow.
This is done through Vipps Userinfo which
You can learn more at the [OIDC Standard](https://openid.net/specs/openid-connect-core-1_0.html#UserInfo).

To enable the posibility to fetch profile information for a user the merchant can add a `scope`
parameter to the draft agreement call:
[`POST:/recurring/v2/agreements`][draft-agreement-endpoint].

If the enduser has not already consented to sharing information from Vipps to the merchant the user will be asked for such consent before activating the agreement. Once the agreement has been accepted the merchant can get the profile information from our Userinfo endpoint.

A users consent to share information with a merchant applies across our services. Thus, if the merchant implements Vipps login in addition to profile information as part of the agreement flow, the merchant can also use Vipps to log the user in without the need for additional consents.

### Scope

| Scope      | Description                                    | User consent required  |
| ------------| -----------------------------------------------|-------- |
| `address`     | List containing the user's addresses. Will always contain the home address, but can also include work and other.    |   yes   |
| `birthDate`   | Birth date (BankID verified)                               |   yes   |
| `email`       | Email address (verified), the flag "email_verified : true" in the response indicates whether the email address is verified                                   |   yes   |
| `name`        | First, middle and given name (verified with National Population Register)              |   yes   |
| `phoneNumber` | Phone number (verified - the number used when creating the Vipps account)                          |   yes   |
| `nin`        | Norwegian national identity number (verified with BankID). **NB:** Merchants need to apply for access to NIN. See: [Who can get access to NIN and how?](https://github.com/vippsas/vipps-login-api/blob/master/vipps-login-api-faq.md#who-can-get-access-to-nin-and-how) |   yes      |
| `accountNumbers` | User bank account numbers. **NB:** Merchants need to apply for access to accountNumbers. See: [Who can get access to account numbers and how?](https://github.com/vippsas/vipps-login-api/blob/master/vipps-login-api-faq.md#who-can-get-access-to-accountnumbers-and-how) |   yes      |

See the API specification for the formats and other details for each scope.

**Please note:** If the e-mail address that is delivered has the flag `email_verified : false`
this address should not be used to link the user to an existing account without
further authentication. Such authentication could be to prompt the user to
login to the original account or confirm the account linking by having a
confirmation link sent to the email address.

### Userinfo call by call guide

Scenario: You want to complete a payment and get the name and phone number of
a customer.

1. Retrieve the access token:
   [`POST:/recurring/v2/accesstoken/get`](https://vippsas.github.io/vipps-recurring-api/#/Access%20Endpoints/getAccessToken).
2. Add the scope field to the draft agreement request body and include the scope you wish to get
   access to (valid scope) before calling [`POST:/recurring/v2/agreements`][draft-agreement-endpoint].
3. The user consents to the information sharing and accepts the agreement in Vipps.
4. Retrieve the `sub` by calling
   [`GET:/recurring/v2/agreements/{agreementId}`](https://vippsas.github.io/vipps-recurring-api/#/Agreement%20Endpoints/getAgreement)
5. Using the sub from step 4, call
   [`GET:/vipps-userinfo-api/userinfo/{sub}`](https://vippsas.github.io/vipps-recurring-api/#/Userinfo%20Endpoint/getUserinfo)
   to retrieve the user's information.

**Important note:** The API call to
[`GET:/vipps-userinfo-api/userinfo/{sub}`](https://vippsas.github.io/vipps-recurring-api/#/Userinfo%20Endpoint/getUserinfo)
must _not_ include the subscription key (the `Ocp-Apim-Subscription-Key` header) used for the Recurring API.
This is because userinfo is part of Vipps Login and is therefore _not_ under the same subscription,
and will result in a `HTTP Unauthorized 401` error.

### Example calls

To request this scope add the scope to the initial call to
[`POST:​/v2​/agreements`][draft-agreement-endpoint]

Example of request with scope:

```json
{
  "currency": "NOK",
  "customerPhoneNumber":"90000000",
  "interval": "MONTH",
  "intervalCount": 1,
  "isApp": false,
  "merchantRedirectUrl": "https://example.com/confirmation",
  "merchantAgreementUrl": "https://example.com/my-customer-agreement",
  "price": 49900,
  "productDescription": "Access to all games of English top football",
  "productName": "Premier League subscription",
  "scope": "address name email birthDate phoneNumber"
}
```

The user then consents and pays in Vipps.

**Please note:** This operation has an all or nothing approach, a user must
complete a valid agreement and consent to all values in order to complete the
session. If a user chooses to reject the terms the agreement will not be
processed. Unless the whole flow is completed, this will be handled as a regular failed agreement by the recurring APIs.

Once the user completes the session a unique identifier `sub` can be retrieved in the agreement details [`GET:/recurring/v2/agreements/{agreementId}`](https://vippsas.github.io/vipps-recurring-api/#/Agreement%20Endpoints/getAgreement) endpoint alongside the full URL to Userinfo.

Example `sub` and `userinfoUrl` format:

```json
{
  "sub": "c06c4afe-d9e1-4c5d-939a-177d752a0944",
  "userinfoUrl": "https://api.vipps.no/vipps-userinfo-api/userinfo/c06c4afe-d9e1-4c5d-939a-177d752a0944"
}
```

This `sub` is a link between the merchant and the user and can used to retrieve
the user's details from Vipps userinfo: [`GET:/vipps-userinfo-api/userinfo/{sub}`](https://vippsas.github.io/vipps-recurring-api/#/Userinfo%20Endpoint/getUserinfo)

**Please note:** It is recommended to get the user's information directly after
completing the transaction. There is however a _time limit of 168 hours_
(one week) to retrieve the consented profile data from the `/userinfo` endpoint to
better support merchants that depend on manual steps/checks in their process of
fetching the profile data. The merchant will get the information that is in the
user profile at the time when they actually fetch the information. This means
that the information might have changed from the time the user completed the
transaction and the fetching of the profile data.

### Userinfo call

This endpoint returns the payload with the information that the user has consented to share.

Call [`GET:/vipps-userinfo-api/userinfo/{sub}`](https://vippsas.github.io/vipps-recurring-api/#/Userinfo%20Endpoint/getUserinfo) with the `sub` that was retrieved earlier. See below on how to construct the call.

**Request**

*Headers*

| Header            | Description                            |
| ----------------- | -------------------------------------  |
| Authorization     | "Bearer {Access Token}"                |

The access token is received on a successful request to the token endpoint described in [Authentication](#authentication).

**Important note:** Subscription key used for the Recurring API must _not_ be included. This is because userinfo is part of Vipps Login and is therefore _not_ under the same subscription, and will result in a `HTTP Unauthorized 401` error.

**Example response:**

```json
{
    "sub": "c06c4afe-d9e1-4c5d-939a-177d752a0944",
    "birthdate": "1815-12-10",
    "email": "user@example.com",
    "email_verified": true,
    "nin": "10121550047",
    "name": "Ada Lovelace",
    "given_name": "Ada",
    "family_name": "Lovelace",
    "sid": "7d78a726-af92-499e-b857-de263ef9a969",
    "phone_number": "4712345678",
    "address": {
        "street_address": "Suburbia 23",
        "postal_code": "2101",
        "region": "OSLO",
        "country": "NO",
        "formatted": "Suburbia 23\\n2101 OSLO\\nNO",
        "address_type": "home"
    },
    "other_addresses": [
        {
            "street_address": "Fancy Office Street 2",
            "postal_code": "0218",
            "region": "OSLO",
            "country": "NO",
            "formatted": "Fancy Office Street 2\\n0218 OSLO\\nNO",
            "address_type": "work"
        },
        {
            "street_address": "Summer House Lane 14",
            "postal_code": "1452",
            "region": "OSLO",
            "country": "NO",
            "formatted": "Summer House Lane 14\\n1452 OSLO\\nNO",
            "address_type": "other"
        }
    ],
    "accounts": [
        {
            "account_name": "My savings",
            "account_number": "12064590675",
            "bank_name": "My bank"
        }
    ]
}
```

![Userinfo sequence](images/userinfo-direct.png)

### Consent

A user's consent to share information with a merchant applies across all Vipps
services. Thus, if the merchant implements Vipps Login in addition to profile
information as part of the agreement flow, the merchant can also use Vipps to
log the user in without the need for additional consent.

The user is presented with a consent card that must be accepted before
approving the agreement in Vipps. The following screens shows examples
of consent cards for Android(left) and iOS(right):

![Consent card](images/share-user-info.png)

**Please note:** This operation has an "all or nothing" approach, so a user must accept the agreement and consent to _all_ values in order to complete the
session. If a user chooses to reject the terms the agreement will not be
activated. Unless the whole flow is completed, this will be handled as a
failed agreement by the Recurring API.

## Recurring agreements with variable amount
Recurring with variable amounts offer merchants a way to charge users a different amount each interval, based on the users specified max amount.

Instead of setting a price when drafting a new agreement, the new `suggestedMaxAmount` field is set to what the maximum price could be each interval. `suggestedMaxAmount` is then presented to the user when accepting an agreement, as a suggestion that indicates the maxmium price that could potentially be charged within each interval.
The user chooses a max amount themselves when accepting the agreement, but we recomended the user to choose the same amount as `suggestedMaxAmount`. The max amount can at any time be changed by the user. What the user has picked as their max amount will be available in the `GET agreement` response. Its recommended that when you set the `suggestedMaxAmount`, that you set a realistic amount - as setting it to unrealistic amounts might scare of the user when they accept the agreement.

### How it works

#### Create agreement

Create an agreement and specify that it's with `variableAmount` and set a `suggestedMaxAmount` (in øre).

Create agreement request:

```json
{
  "variableAmount": {
    "suggestedMaxAmount": 200000
  },
  "currency": "NOK",
  "interval": "MONTH",
  "intervalCount": 1,
  "isApp": false,
  "merchantRedirectUrl": "https://example.com/confirmation",
  "merchantAgreementUrl": "https://example.com/my-customer-agreement",
  "customerPhoneNumber": "90000000",
  "productDescription": "Access to subscription",
  "productName": "Power company A"
}
```

**Note:** There is no need to supply the agreement with a `price` field, this will be ignored since the user picks the allowed max amount themselves.

**Restrictions when using variable amount:**

- There is currently a limit of 5 000 NOK for the `suggestedMaxAmount`.
- `Campaign` can not be used when the agreement has `variableAmount`.

The user will be presented with the variable agreement in the app. Here they can change the max amount they allow to be charged each interval.

Accepting agreement in Vipps
![variable_amount_accept](images/variable_amount_accept.png)

Variable amount and initial charge can be combined
![variable_amount_accept_initial](images/variable_amount_accept_initial.png)

#### Get agreement

Retrieving the agreement shows the `maxAmount` that was picked by the user.

GET agreement response:

```json
{
    "id": "agr_Yv2zYk3",
    "start": "2021-06-18T19:56:22Z",
    "stop": null,
    "status": "ACTIVE",
    "productName": "Power company A",
    "price": 0,
    "productDescription": "Access to subscription",
    "interval": "MONTH",
    "intervalCount": 1,
    "currency": "NOK",
    "campaign": null,
    "sub": null,
    "userinfoUrl": null,
    "tags": [],
    "variableAmount": {
        "suggestedMaxAmount": 200000,
        "maxAmount": 180000
    }
}
```

#### Change suggestedMaxAmount

It's possible to change the suggestedMaxAmount on the agreement by calling the update agreement endpoint with the PATCH request below.

```json
{
    "suggestedMaxAmount": 300000
}
```

**Note:** The user will not be allerted by this change by Vipps.

#### Create charge

There are changes in how the interval and amount calculation works for agreements with `variable amount`. The amount of the charge/charges in the interval can not be higher than either the `suggestedMaxAmount` or `maxAmount` field, based on whichever is highest.

Changes in how interval works:

**Yearly:**

Can be charged once a year, regardless of the day in the year.

Example:

- First charge can be 02.06.2022
- Second could be any date in 2023, for example 01.01.2023

**Monthly:**

Can be charged once a calendar month, regardless of the day in the month.

Example:

- First charge can be 03.03.2022
- Second could be any day the next month, for example 04.20.2022

**Weekly:**

Can be charged once a week, regardless of the day in the week.

Example:

- First charge can be on a Wednesday
- Second charge could be on a Monday the next week

**Daily:**

Once a day, same as without variable amount.

**Note:** In the examples above the `intervalCount` is 1. This can be changed as described in the [Intervals](#intervals) section.

#### Charge amount higher than the users max amount

If the created charge is above the users `max amount`, the charge will be set to `DUE` with a `failureReason` as shown below. If the user does not update their maxAmount to the same or a higher amount than the charge, it will fail when `dueDate` + `retryDays` is reached.

GET charge response where amount is higher than the users max amount:

```json
{
    "id": "chr-ZZt75qs",
    "status": "DUE",
    "due": "2021-06-19T00:01:00Z",
    "amount": 190000,
    "amountRefunded": 0,
    "transactionId": null,
    "description": "Monthly payment",
    "type": "RECURRING",
    "failureReason": "charge_amount_too_high",
    "failureDescription": "Amount is higher than the users specified max amount"
}
```

The user will aslo see a failure description on the charge in the app and a push notification will be sent if enabled.

Display of ChargeFailure and changing maxAmount in Vipps
![variable_amount_charge](images/variable_amount_charge.png)

## Skip landing page

_This functionality is only available for special cases._

Skipping the landing page is only reserved for physical points of sale and vending machines, when there is no display available.

This feature has to be specially enabled by Vipps for eligible sale units: The sale units must be whitelisted by Vipps.

If the `skipLandingPage` property is set to `true` in the [`POST:/recurring/v2/agreements`][draft-agreement-endpoint] call, it will cause a push notification to be sent to the given phone number immediately, without loading the landing page.

If the sale unit is not whitelisted, the request will fail and an error message will be returned.

If you want to check if a sale unit is allowed to use `skipLandingPage`:

1. Draft an agreement with `"skipLandingPage": true`.
2. Check the response code and message. The API will return an error if attempting to use `skipLandingPage` without being whitelisted.

If you need to skip the landing page for a different reason: contact your Key Account Manager. If you do not have a KAM:
Please log in on [portal.vipps.no](https://portal.vipps.no), find the right sale unit and click the email link under the
"i" information bubble. Include a detailed description of why it is not possible to display the landing page.

**Please note:** When using `skipLandingPage`, the user is not sent to a URL after complation of the payment. The "result page" is just
the confirmation in Vipps. The `fallback` URL sent in the API request can therefore be the merchant's main URL, like `https://example.com`, etc.

## HTTP responses

This API returns the following HTTP statuses in the responses:

| HTTP status                 | Description                                 |
| --------------------------- | ------------------------------------------- |
| `200 OK`                    | Request successful                          |
| `201 Created`               | Request successful, resource created        |
| `400 Bad Request`           | Invalid request, see the error for details  |
| `401 Unauthorized`          | Invalid credentials                         |
| `403 Forbidden`             | Authentication ok, but credentials lacks authorization  |
| `404 Not Found`             | The resource was not found                  |
| `409 Conflict`              | Unsuccessful due to conflicting resource    |
| `422 Unprocessable Entity`  | Vipps could not process                     |
| `429 Too Many Requests`     | Look at [table below to view current rate limits](#rate-limiting)   |
| `500 Server Error`          | An internal Vipps problem.                  |

All error responses contains an `error` object in the body, with details of the
problem.

## Rate limiting

We have added rate-limiting to our API (`HTTP 429 Too Many Requests`) to prevent
fraudulent and wrongful behaviour, and increase the stability and security of
our API. The limits should not affect normal behaviour, but please contact us
if you notice any unexpected behaviour.

The "Key" column specifies what we consider to be the unique identifier, and
what we "use to count". The limits are of course not _total_ limits.

| API                                                                                                      | Limit          | Key used                                          | Explanation |
|----------------------------------------------------------------------------------------------------------|----------------|---------------------------------------------------| --- |
| [CreateCharge](https://vippsas.github.io/vipps-recurring-api/#/Charge%20Endpoints/createCharge)         | 2 per minute   | agreementId + chargeId (based on idempotency key) | Two calls per minute per unique agreementId and chargeId |
| [CancelCharge](https://vippsas.github.io/vipps-recurring-api/#/Charge%20Endpoints/cancelCharge)         | 5 per minute   | agreementId + chargeId                            | Five calls per minute per unique agreementId and chargeId |
| [CaptureCharge](https://vippsas.github.io/vipps-recurring-api/#/Charge%20Endpoints/captureCharge)       | 5 per minute   | agreementId + chargeId                            | Five calls per minute per unique agreementId and chargeId |
| [RefundCharge](https://vippsas.github.io/vipps-recurring-api/#/Charge%20Endpoints/refundCharge)         | 5 per minute   | agreementId + chargeId                            | Five calls per minute per unique agreementId and chargeId |
| [ListAgreements](https://vippsas.github.io/vipps-recurring-api/#/Agreement%20Endpoints/listAgreements)  | 5 per minute   | (per merchant)                                    | Five calls per minute per merchant |
| [UpdateAgreement](https://vippsas.github.io/vipps-recurring-api/#/Agreement%20Endpoints/updateAgreement)| 5 per minute   | agreementId                                       | Five calls per minute per unique agreementId |
| [FetchCharge](https://vippsas.github.io/vipps-recurring-api/#/Charge%20Endpoints/getCharge)             | 10 per minute  | agreementId + chargeId                            | Ten calls per minute per unique agreementId and chargeId |
| [ListCharges](https://vippsas.github.io/vipps-recurring-api/#/Charge%20Endpoints/listCharges)           | 10 per minute  | agreementId                                       | Ten calls per minute per unique agreementId |
| [FetchAgreement](https://vippsas.github.io/vipps-recurring-api/#/Agreement%20Endpoints/getAgreement)    | 120 per minute | agreementId                                       | 120 calls per minute per unique agreementId |
| [DraftAgreement][draft-agreement-endpoint]  | 300 per minute | (per merchant)                                    | 300 calls per minute per merchant |

**Please note:** The "Key" column is important. The above means that we allow two
CreateCharge calls per minute per unique agreementId and chargeId. This is to prevent
too many CreateCharge calls for the same charge. The overall limit for number of
different payments is far higher than 2.

## Partner keys

In addition to the normal [Authentication](#authentication) we offer _partner keys_,
which let a partner make API calls on behalf of a merchant.

If you are a Vipps partner managing agreements on behalf of Vipps merchants you
can use your own API credentials to authenticate, and then send
the `Merchant-Serial-Number` header to identify which of your merchants you
are acting on behalf of. The `Merchant-Serial-Number` must be sent in the header
of all API requests.

By including the [Optional HTTP Headers](#optional-vipps-http-headers) you will make
it easier to investigate problems, if anything unexpected happens. Partners may
re-use the values of the `Vipps-System-Name` and `Vipps-System-Plugin-Name` in
the plugins headers if having different values do not make sense.

Here's an example of headers (please refer to the
[OpenAPI/Swagger specification](https://vippsas.github.io/vipps-recurring-api/)
for all the details):

```http
Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1Ni <snip>
Ocp-Apim-Subscription-Key: 0f14ebcab0ec4b29ae0cb90d91b4a84a
Merchant-Serial-Number: 123456
Vipps-System-Name: Acme Enterprises Ecommerce DeLuxe
Vipps-System-Version: 3.1.2
Vipps-System-Plugin-Name: Point Of Sale Excellence
Vipps-System-Plugin-Version: 4.5.6
Content-Type: application/json
```

**Please note:** The Merchant Serial Number (MSN) is a unique id for the sale
unit. This is a required parameter if you are a Vipps partner making API requests
on behalf of a merchant. The partner must use the _merchant's_ MSN, not the
partner's MSN. This parameter is optional, and recommended, for regular Vipps
merchants making API calls for themselves.

## Polling guidelines

General guidelines for When to start polling with
[`GET:/recurring/v2/agreements/{agreementId}`](https://vippsas.github.io/vipps-recurring-api/#/Agreement%20Endpoints/getAgreement):

1. Start after 5 seconds
2. Check every 2 seconds

These are reasonable values, but different merchants have different use cases,
and values should be adapted to the specific case.

See [Timeouts](#timeouts) for details about timeouts.

## Timeouts

### Using a phone

Both the deeplink URL, which causes the app-switch to Vipps, and the landing
page displayed in browsers, is valid for 5 minutes.

If the user does not act on the app-switch (such as not attempting to log into
Vipps) within 5 minutes, the payment times out.

After the app-switch to Vipps, the user has another 5 minutes to complete the
payment in Vipps.

This means that the user has a total of 10 minutes to complete the payment.

### Using a laptop/desktop

If the user is using a laptop/desktop device, and the user must confirm or
enter the phone number on the landing page within 5 minutes.
If the user does not do so, the payment times out.

After the user has clicked "OK" on the landing page, the user
has an additional 5 minutes to complete the payment in Vipps.

This means that the user has a total of 10 minutes to complete the payment.

## Authentication and authorization

All Vipps API requests must include an `Authorization` header with
a JSON Web Token (JWT), which we call the _access token_.
The access token is obtained by calling
[`POST:/accesstoken/get`](https://vippsas.github.io/vipps-ecom-api/#/Authorization_Service/fetchAuthorizationTokenUsingPost)
and passing the `client_id`, `client_secret` and `Ocp-Apim-Subscription-Key`.

See [Get an access token](https://github.com/vippsas/vipps-developers/blob/master/vipps-getting-started.md#get-an-access-token)
in the
[Getting started guide](https://github.com/vippsas/vipps-developers/blob/master/vipps-getting-started.md)
for more information.

# Testing

To facilitate automated testing in [The Vipps Test Environment (MT)][vipps-test-environment], the Vipps Recurring API provides a
"force accept" endpoint to avoid manual agreement acceptance in the Vipps app: [`POST:/recurring/v2/agreements/{agreementId}/accept`](https://vippsas.github.io/vipps-recurring-api/#/Agreement%20Endpoints/forceAcceptAgreement).

The "force approve" endpoint allows developers to approve a payment through the Vipps Recurring API without the use of Vipps.
This is useful for automated testing. The endpoint is only available in our test environment.

## Recommendations regarding handling redirects

Since Vipps is a mobile entity the amount of control Vipps have over the redirect back to the merchant after the purchase is completed is limited. A merchant must not assume that Vipps will redirect to the exact same session and for example rely entirely on cookies in order to handle the redirect event. For example the redirect could happen to another browser.

Examples of some, but not all, factors outside of Vipps control.
- Configurations set by the OS itself, for example the default browser.
- User configurations of browsers.
- Users closing app immediately upon purchase.

Therefore, Vipps recommends having a stateless approach in the site that is supposed to be the end session. An example would a polling based result handling from a value in the redirect url.

Example for demonstration purposes that should be handled.

- User starts is in web session in a Chrome Browser.
- A Vipps purchase is started, a redirect URL is defined by the Merchant.
- The user completes the purchase.
- The Vipps app redirects the user.
- The OS defaults to a Safari Browser for the redirect.
- The merchant handles the redirect without the customer noticing any discrepancies from the browser switch.

## When to use campaigns or initial charge

Vipps recurring payments is a fairly flexible service, that allows you as a merchant to tailor the user experience in Vipps to your needs by utilising the normal agreements, initial charges, campaigns, or a combination of those.

This can be a bit confusing when deciding on which implementation to go for.
In short our advice is to implement support for all our flows, and also implement features in your own systems for moving between the flows depending on the use case.

First a short description on the flows.

### Normal agreement flow

![flow_Normal_agreement](images/flow-Normal-agreement.png)

In the normal agreement, the user gets presented with the agreement, agrees to that, and gets sent to a confirmation screen.
On the agreement we present start date, interval, the price of the agreements, as well as `productName` and `productDescription`, the latter of which can be used to describe the agreement to the user and can be defined by the merchant.

This is the preferred flow whenever there is no campaigns or similar present.

### Initial charge flow

![flow_Initial_charge](images/flow-Initial-charge.png)

When an initial charge is present, the flow in Vipps will change. First the user gets presented with an overview over both the agreement and the initial charge. The user then proceed to confirm the agreement, and finally they will have to go through the actual payment of the initial charge.

Here we also show `productName` and `productDescription` on the agreement, as well as `description` on the initial charge. All of which are defined by the merchant.

Initial charges are designed to be used whenever there is an additional cost in setting up the agreement. This could be bundling of a mobilephone together with a mobile subscription, or a TV setup-box when becoming a customer at a cable company. We do not recommend this flow to be used purely for campaigns, as it could be confusing to the user.

As an example: If you have a campaign of 10 NOK for a digital media subscription for 3 months, and the normal price is 299,- monthly, the user would see both the charge of 10 NOK, as well as having to confirm the agreement for 299,- monthly, which can lead the user to believe that both will be payed upon entering the agreement. If used for campaigns, be sure to have good descriptions in `productName` and `productDescription` on the agreement, as well as `description` on the initial charge.

### Campaign

![flow_Campaign](images/flow-Campaign.png)

When setting a campaign, this follows the normal agreement flow - with some changes. Instead of showing the ordinary price of the agreement, the campaign price will override this, and the ordinary price will be shown below together with information about when the change from the campaign price to the ordinary price will happen. Here as well you have options of setting `productName` and `productDescription` to even further explain to the user.

This is the preferred flow whenever you have a type of campaign where the subscription has a certain price for a certain interval or time, before it switches over to ordinary price.

**Note:** Campaign is not supported for `variableAmount` agreements.

### Initial charge and campaign

![flow_initial_charge_campaign](images/flow-Initial-charge-and-campaign.png)

In addition to campaigns and initial charges being available as individual flows, they can also be combined. In this case the user would see first a summary of both the agreement, including the campaign as described in the sections on campaigns, as well as the initial charge. Again, all fields described in previous flows are available for the merchant to display information to the user.

Ideally this flow is intended for when you have a combination of an additional cost when setting up the agreement, presented as the initial charge, as well as having a limited time offer on the actual subscription.

## Questions?

We're always happy to help with code or other questions you might have!
Please create an [issue](https://github.com/vippsas/vipps-recurring-api/issues),
a [pull request](https://github.com/vippsas/vipps-recurring-api/pulls),
or [contact us](https://github.com/vippsas/vipps-developers/blob/master/contact.md).

Sign up for our [Technical newsletter for developers](https://github.com/vippsas/vipps-developers/tree/master/newsletters).

[draft-agreement-endpoint]: https://vippsas.github.io/vipps-recurring-api/#/Agreement%20Endpoints/draftAgreement
[vipps-test-environment]: https://github.com/vippsas/vipps-developers/blob/master/vipps-test-environment.md
