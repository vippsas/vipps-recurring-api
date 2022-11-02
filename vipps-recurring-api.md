<!-- START_METADATA
---
title: API Guide
sidebar_position: 30
---
END_METADATA -->

# Vipps Recurring API

<!-- START_COMMENT -->

ℹ️ Please use the new documentation:
[Vipps Technical Documentation](https://vippsas.github.io/vipps-developer-docs/).

<!-- END_COMMENT -->

The Vipps Recurring API delivers recurring payment functionality for a merchant
to create a payment agreement with a customer for fixed interval payments.
When the agreement is accepted by the end user, the merchant can send charges
that will be automatically processed on the due date.

The overall flow is:

* The merchant creates a draft agreement and proposes it to the customer via Vipps.
* The customer approves the agreement in Vipps.
* The merchant sends a charge request to Vipps at least two days before due date
* If the agreement is active, Vipps authorizes the charge.
* The customer can find a full overview in Vipps, including a link to the merchant's website.

To get access to the Recurring API in production, order Vipps "Faste Betalinger" (_recurring payments_) on
[portal.vipps.no](https://portal.vipps.no).

**IMPORTANT:** Before activating recurring payments for you,
Vipps must perform some extra Know Your Customer (KYC) checks, as required by [Finanstilsynet](https://www.finanstilsynet.no).
You will also need to set up a direct agreement for use of "Vipps på Nett" ([Vipps eCom API](https://github.com/vippsas/vipps-ecom-api)).

As a merchant, you can check if you have access to recurring payments through [portal.vipps.no](https://portal.vipps.no). Partners can check if a merchant has access to recurring payments through the [Vipps Partner API](https://github.com/vippsas/vipps-partner-api/blob/main/vipps-partner-api.md#get-information-about-a-sale-unit-based-on-msn).

See a detailed example of [how it works](vipps-login-recurring-howitworks.md).

See also:

* [API Reference](https://vippsas.github.io/vipps-recurring-api/)
* [Quick start guide](vipps-recurring-api-quick-start.md)
* [Integration checklist](vipps-recurring-api-checklist.md)
* [FAQ](vipps-recurring-api-faq.md)

API version: 2.0.0.

Document version 2.6.1.

<!-- START_TOC -->

## Table of Contents

- [Vipps Recurring API](#vipps-recurring-api)
  - [Table of Contents](#table-of-contents)
  - [Terminology](#terminology)
  - [Flow diagram](#flow-diagram)
  - [Call by call guide](#call-by-call-guide)
    - [Direct capture](#direct-capture)
    - [Reserve capture](#reserve-capture)
  - [API endpoints](#api-endpoints)
  - [Authentication and authorization](#authentication-and-authorization)
  - [Vipps HTTP headers](#vipps-http-headers)
  - [Idempotency Key header (V3 API Coming Soon)](#idempotency-key-header-v3-api-coming-soon)
  - [Continuation-Token header (V3 API Coming Soon)](#continuation-token-header-v3-api-coming-soon)
  - [orderId recommendations](#orderid-recommendations)
  - [Agreements](#agreements)
    - [Create an agreement](#create-an-agreement)
    - [Accept an agreement](#accept-an-agreement)
    - [Intervals](#intervals)
      - [Example in the V2 api:](#example-in-the-v2-api)
    - [Initial charge](#initial-charge)
    - [Campaigns](#campaigns)
      - [Campaigns in V2 API](#campaigns-in-v2-api)
      - [(Coming soon - WORK IN PROGRESS) Campaigns in V3 API](#coming-soon---work-in-progress-campaigns-in-v3-api)
        - [Price campaign](#price-campaign)
        - [Period campaign](#period-campaign)
        - [Event campaign](#event-campaign)
        - [Full flex campaign](#full-flex-campaign)
        - [Product description guidelines for agreements with campaigns.](#product-description-guidelines-for-agreements-with-campaigns)
    - [Retrieve an agreement](#retrieve-an-agreement)
  - [Charges](#charges)
    - [Create a charge](#create-a-charge)
    - [Capture a charge](#capture-a-charge)
    - [Partial capture (Coming soon)](#partial-capture-coming-soon)
    - [Amount changes](#amount-changes)
    - [Charge descriptions](#charge-descriptions)
    - [Cancel a charge](#cancel-a-charge)
    - [Charge times](#charge-times)
    - [Charge retries](#charge-retries)
    - [Retrieve a charge](#retrieve-a-charge)
    - [List charges](#list-charges)
  - [Manage charges and agreements](#manage-charges-and-agreements)
    - [Agreement states](#agreement-states)
    - [Update an agreement](#update-an-agreement)
    - [Pause an agreement](#pause-an-agreement)
    - [Stop an agreement](#stop-an-agreement)
    - [Charge states](#charge-states)
    - [Example charge flows](#example-charge-flows)
  - [Charge failure reasons](#charge-failure-reasons)
    - [Deprecated failureReasons](#deprecated-failurereasons)
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
      - [Charge amount higher than the user's max amount](#charge-amount-higher-than-the-users-max-amount)
  - [Skip landing page](#skip-landing-page)
  - [HTTP responses](#http-responses)
  - [Rate limiting](#rate-limiting)
  - [Partner keys](#partner-keys)
  - [Polling guidelines](#polling-guidelines)
  - [Timeouts](#timeouts)
  - [Testing](#testing)
  - [Recommendations regarding handling redirects](#recommendations-regarding-handling-redirects)
  - [When to use campaigns or initial charge](#when-to-use-campaigns-or-initial-charge)
    - [Normal agreement flow](#normal-agreement-flow)
    - [Initial charge flow](#initial-charge-flow)
    - [Campaign](#campaign)
    - [Initial charge and campaign](#initial-charge-and-campaign)
  - [Questions?](#questions)

<!-- END_TOC -->

## Terminology

| Term        | Description                                                                                                  |
|-------------|--------------------------------------------------------------------------------------------------------------|
| Agreement   | A payment subscription with a set of parameters that a customer agrees to.                                   |
| Charge      | A single payment within an agreement.                                                                        |
| Idempotency | The property of endpoints to be called multiple times without changing the result after the initial request. |

## Flow diagram

This diagram shows a simplified payment flow:

![Recurring agreement flow](images/Recurring-createagreement.svg)

## Call by call guide

There are two happy-flows based on how the sale unit is set up:
One for "direct capture" and one for "reserve capture".
This is specified with the `transactionType`, and for "direct capture"
the sale unit must be configured for this by Vipps.
See the Vipps FAQ for the difference:
[What is the difference between "Reserve Capture" and "Direct Capture"?](https://github.com/vippsas/vipps-developers/blob/master/faqs/reserve-and-capture-faq.md#what-is-the-difference-between-reserve-capture-and-direct-capture)

**Note:** Vipps will *only* perform a payment transaction on an agreement that
the merchant has created a charge for with the [`POST:/agreements/{agreementId}/charges`][create-charge-endpoint] endpoint.
You can also [manage charges and agreements](#manage-charges-and-agreements).

### Direct capture

For a `"transactionType": "DIRECT_CAPTURE"` setup, the normal flow would be:

1. Create a (draft) agreement using the [`POST:/agreements`][draft-agreement-endpoint-v2] endpoint.
   The user can now confirm the agreement in Vipps (the app). See [Create a new agreement](#create-an-agreement).
2. The user approves the agreement in Vipps:
   This will result in a capture(or reserve) of the initial charge (if one was defined in the first step).
   See [Initial charge](#initial-charge).
3. Retrieve the agreement by calling the [`GET:/agreements/{agreementId}`][fetch-agreement-endpoint] endpoint.
   See [Retrieve an agreement](#retrieve-an-agreement).
   **Note:** At this point the agreement will be `ACTIVE` if the user completed step 2.
4. All future charges can be created by using the [`POST:/agreements/{agreementId}/charges`][create-charge-endpoint] endpoint.
   For direct capture you must set `"transactionType": "DIRECT_CAPTURE"`.
   See [Create a charge](#create-a-charge).
   Based on the `due` set in the request, we will try to process the charge on that day.
   If for some reason, a charge fails to be processed,
   we will retry for the number of days specified by the `retryDays` value.
   We recommend at least two days retry.

### Reserve capture

**Note:** Reserve capture on recurring charges is available in the recurring API v3 (Coming soon - WORK IN PROGRESS)
In the API V2, reserve capture is only available on initial charges.

For a `"transactionType": "RESERVE_CAPTURE"` setup, the normal flow would be:

1. Create a (draft) agreement using the [`POST:/agreements`][draft-agreement-endpoint-v2] endpoint.
   The user can now confirm the agreement in Vipps (the app). See [Create a new agreement](#create-an-agreement).
2. The user approves the agreement in Vipps:
   This will result in a capture(or reserve) of the initial charge (if one was defined in the first step).
   See [Initial charge](#initial-charge).
3. Retrieve the agreement by calling the [`GET:/agreements/{agreementId}`][fetch-agreement-endpoint] endpoint.
   See [Retrieve an agreement](#retrieve-an-agreement).
   **Note:** At this point the agreement will be `ACTIVE` if the user completed step 2.
4. All future charges can be created by using the [`POST:/agreements/{agreementId}/charges`][create-charge-endpoint] endpoint.
   For reserve capture you must set `"transactionType": "RESERVE_CAPTURE"`.
   See [Create a charge](#create-a-charge).
   Based on the `due` set in the request, we will try to process the charge on that day.
   If the charge is processed successfully, the status will be `RESERVED`.
   If for some reason, a charge fails to be processed,
   we will retry for the number of days specified by the `retryDays` value.
   We recommend at least 2 days retry.
5. If there is a product that is shipped to the customer, the charge should be captured at this point.
   Capture the charge by calling the [`POST:/agreements/{agreementId}/charges/{chargeId}/capture`][capture-charge-endpoint] endpoint.


## API endpoints

| Operation                                       | Description                                           | Endpoint                                                                               |
|-------------------------------------------------|-------------------------------------------------------|----------------------------------------------------------------------------------------|
| List agreements                                 | List all agreements for a merchant.                   | [`GET:/agreements`][list-agreements-endpoint]                                          |
| [Create an agreement](#create-an-agreement)     | Create a new, draft agreement.                        | [`POST:/agreements`][draft-agreement-endpoint-v2]                                      |
| [Retrieve an agreement](#retrieve-an-agreement) | Retrieve the details of an agreement.                 | [`GET:/agreements/{agreementId}`][fetch-agreement-endpoint]                            |
| [Update an agreement](#update-an-agreement)     | Update an agreement with new details.                 | [`PATCH:/agreements/{agreementId}`][update-agreement-patch-endpoint]                   |
| [Stop an agreement](#stop-an-agreement)         | Update the status to `STOPPED`.                       | [`PATCH:/agreements/{agreementId}`][update-agreement-patch-endpoint]                   |
| [List charges](#list-charges)                   | Get all charges for an agreement.                     | [`GET:/agreements/{agreementId}/charges`][list-charges-endpoint]                       |
| [Create a charge](#create-a-charge)             | Create a new charge for an agreement.                 | [`POST:/agreements/{agreementId}/charges`][create-charge-endpoint]                     |
| [Retrieve a charge](#retrieve-a-charge)         | Retrieve all details of a charge.                     | [`GET:/agreements/{agreementId}/charges/{chargeId}`][fetch-charge-endpoint]            |
| [Capture a charge](#capture-a-charge)           | Each charge must first be created, then captured.     | [`POST:/agreements/{agreementId}/charges/{chargeId}/capture`][capture-charge-endpoint] |
| [Cancel a charge](#cancel-a-charge)             | Cancel an existing charge before the user is charged. | [`DELETE:/agreements/{agreementId}/charges/{chargeId}`][cancel-charge-endpoint]        |
| Refund a charge                                 | Refund a charge that has been performed.              | [`POST:/agreements/{agreementId}/charges/{chargeId}/refund`][refund-charge-endpoint]   |

See [Authentication and authorization](#authentication-and-authorization).

See the [Quick start guide](vipps-recurring-api-quick-start.md) for en easy way to test the API.

## Authentication and authorization

All Vipps API calls are authenticated with an access token and an API subscription key.
See
[Get an access token](https://github.com/vippsas/vipps-developers/blob/master/vipps-getting-started.md#get-an-access-token)
in the Getting started guide, for details.

## Vipps HTTP headers

We recommend using the standard Vipps HTTP headers for all requests.

See [Vipps HTTP headers](https://github.com/vippsas/vipps-developers/blob/master/common-topics/http-headers.md)
in the Getting started guide, for details.

## Idempotency Key header (V3 API Coming Soon)

The `Idempotency-Key` header must be set in each `POST` or `PATCH` request.
This way, if a request fails for any reason, it can be retried with the same `Idempotency-Key`.
Also, in the case of retries, it will prevent duplicating operations.

**Important:** If the response is a client-error (4xx), you will continue to get the same error as long as you use the same idempotency-key, as the requested operation is not retried.

**Important:** If you reuse an idempotency-key on a different request, you will get a 409 CONFLICT.

See the
[Idempotency header](https://github.com/vippsas/vipps-developers/blob/master/common-topics/http-headers.md#idempotency)
for more details.

## Continuation-Token header (V3 API Coming Soon)

The `Continuation-Token` header is introduced on endpoints that returns multiple items to allow pagination. When returned from the API, it indicates that there are more items to be received. In order to receive the next page, repeat the request adding the received token in the `Continuation-Token`-header.

## orderId recommendations

An optional _and recommended_ `orderId` field can be set in the [`POST:/agreements/{agreementId}/charges`][create-charge-endpoint] request.

```json
{
  "amount": 49900,
  "currency": "NOK",
  "description": "Premier League subscription",
  "due": "2030-12-31",
  "retryDays": 5,
  "orderId": "acmeshop123order123abc"
}
```

The `orderId` replaces the `chargeId`.

**Important:** If the `orderId` is provided:

* The value of the `orderId` is used for all instances of `chargeId`
* The `orderId` (and `chargeId`) is sed for all subsequent identification
  of the charge.
* The `orderId` is used in the settlement files.

This `orderId` must be unique across all Recurring and eCom
transactions for the given `merchantSerialNumber`.

If the field is _not_ provided, Vipps will automatically create a unique id
prefixed with `chr-`: `chr-xxxxxxx`(where each x is an alphanumeric character).

If you ever have a problem that requires us to search in our logs, we will need
`orderId` values that are "unique enough" to actually find them. An `orderId` that
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
`orderId` values could start at `654321000000000001` and increment by 1
for each order, or some similar, unique and readable pattern.

## Agreements

An agreement is between the Vipps user and the merchant.
Think of it as a subscription.
An agreement has payments, called [charges](#charges).

### Create an agreement

This is an example of a request body for the [`POST:/agreements`][draft-agreement-endpoint-v2] call:

```json
{
  "currency": "NOK",
  "customerPhoneNumber":"90000000",
  "interval": "MONTH",
  "intervalCount": 1,
  "merchantRedirectUrl": "https://example.com/confirmation",
  "merchantAgreementUrl": "https://example.com/my-customer-agreement",
  "price": 49900,
  "productDescription": "Access to all games of English top football",
  "productName": "Premier League subscription"
}
```

**Note:** To create agreements with support for variable amounts on charges, see
[Recurring agreements with variable amount](#recurring-agreements-with-variable-amount).

The `merchantAgreementUrl` is a link to the customer's account page on your website, where they
can manage the agreement (e.g., change, pause, cancel the agreement).
Vipps does not offer any form of agreement management, as this may include
quite complex operations (e.g., changing subscription types,
temporary address change).
The URL is opened in the standard web browser.  
The integrator ***must*** implement such functionality for the customer to manage the agreement in their system.

**Please note:** If the user closes Vipps before the redirect is done,
the `merchantRedirectUrl` will not be used. It is therefore important that you
actively check the payment with the
[`GET:/agreements/{agreementId}`][fetch-agreement-endpoint] endpoint.

The `merchantAgreementUrl` is just a normal link to a page where the customer
can log in and manage the agreement.
Vipps does not have any specific requirements for the security of the page, other than using https, but
strongly recommend using
[Vipps Login](https://www.vipps.no/produkter-og-tjenester/bedrift/logg-inn-med-vipps/logg-inn-med-vipps/)
so the user does not need a username or password, but is logged
in automatically through Vipps. See the
[Login API documentation](https://github.com/vippsas/vipps-login-api)
for more details.

The request parameters have the following size limits
(see the [`POST:/agreements`][draft-agreement-endpoint-v2] endpoint for more details):

* `productName`: Max length 45 characters
* `productDescription`: Max length 100 characters
* `price`: Greater than 100, meaning 1 NOK.

Agreements may be initiated with or without an [initial charge](#initial-charge).

The agreement price and the amount for the initial charge, is given in øre,
the centesimal subdivision of the Norwegian kroner (NOK).
There are 100 øre in 1 krone.

| # | Agreement      | Description                                                                          |
|:--|:-----------|:-------------------------------------------------------------------------------------|
| 1 | `Agreement starting now`  | Agreement with an `initialcharge` that uses `DIRECT_CAPTURE` will only be `active` if the initial charge is processed successfully |
| 2 | `Agreement starting in future`  | Agreement without an `initialcharge`, or with `initialcharge` that uses `RESERVE_CAPTURE`, can be approved but no payment will happen until the first charge is provided |

The response contains an `agreementResource`, a `vippsConfirmationUrl` and an `agreementId`.
This `agreementResource` is a complete URL for performing a
[`GET:/agreements/{agreementId}`][fetch-agreement-endpoint]
request.

The `vippsConfirmationUrl` should be used to redirect the
user to the Vipps landing page in a desktop flow (with `https://`),
or to Vipps in a mobile flow (with `vipps://`), where the
user can then approve the agreement.

### Accept an agreement

The [`POST:/agreements`][draft-agreement-endpoint-v2] endpoint will return the following JSON structure.

```json
{
  "vippsConfirmationUrl": "https://api.vipps.no/dwo-api-application/v1/deeplink/vippsgateway?v=2/token=eyJraWQiOiJqd3RrZXkiLCJhbGciOiJSUzI1NiJ9.eyJzdWIiOiJmMDE0MmIxYy02YjI",
  "agreementResource": "https://api.vipps.no-recurring/v2/agreements/agr_TGSuPyV",
  "agreementId": "agr_TGSuPyV"
}
```

The `vippsConfirmationUrl` should be used to redirect the user to the Vipps landing
page. The user can then confirm their identity and receive a prompt to accept the
agreement within Vipps.

If the payment is initiated in a native app, it is possible to explicitly force
a `vipps://` URL by sending the `isApp` parameter in the initiate call:

* `"isApp": false`: The URL is `https://`, which handles
  everything automatically for you.
  The phone's operating system will know, through "universal linking", that
  the `https://api.vipps.no` URL should open the Vipps app, and not the default
  web browser.
  **Please note:** In some cases, this requires the user to approve that
  Vipps is opened, but this is usually only the first time.
* `"isApp": true`: The URL is for a deeplink, for forced app-switch to Vipps, with `vipps://`.
  **Please note:** In our test environment (MT), the scheme is `vippsMT://`

If the user does not have Vipps installed:

* `"isApp":false`: The Vipps landing page will be shown,
   and the user can enter a phone number and pay on a device with Vipps installed.
* `"isApp": true`: The user will get an error message saying that the link can
  not be opened.


### Intervals

Intervals are defined with an interval type `YEAR`, `MONTH`, `WEEK`, or `DAY` and frequency as a count. The count can be any number between 1 and 31.

#### Example in the V2 api:

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

Example for a subscription every 30th day:
```json
{
  "interval": {
    "unit": "DAY",
    "count": 30
  }
}
```

Example in the V3 api (Coming soon - WORK IN PROGRESS):

Example for a bi-weekly subscription:
```json
{
  "interval": {
    "unit": "WEEK",
    "count": 2
  }
}
```

Example for a quarterly subscription
```json
{
  "interval": {
    "unit": "MONTH",
    "count": 3
  }
}
```

Examples for a yearly subscription
```json
{
  "interval": {
    "unit": "YEAR",
    "count": 1
  }
}
```
OR
```json
{
  "interval": {
    "unit": "MONTH",
    "count": 12
  }
}
```

Example for a subscription every 30th day:
```json
{
  "interval": {
    "unit": "DAY",
    "count": 30
  }
}
```

**Please note:** It is not possible to change intervals. If the user has
accepted a yearly interval, the agreement cannot be changed to a monthly
agreement. This requires a new agreement and a new consent from the user.
It _is_ possible to make a monthly agreement and charge some months only.
The general rule: Be as customer friendly and easy to understand
as possible.

### Initial charge

**Please note:** If the subscription is cheaper in the beginning than the normal price later, use
[campaigns](#campaigns) in combination with initial charge.
If you use `initialcharge` alone for campaigns, users will be confused by how it appears in Vipps,
as it looks like the full price period starts immediately.

Initial charge will be performed if the `initialcharge` is provided when
creating an agreement. If there is no initial charge, don't send `initialcharge`
when creating the new agreement.

Unlike regular (or `RECURRING`) charges, there is no price limit on an `initialCharge`.
This allows for products to be bundled with agreements as one transaction
(for example, a phone). The user will be clearly informed when an `initialCharge`
is included in the agreement they are accepting.

See [Charge Titles](#charge-descriptions) for explanation of how the charge description
is shown to the user.

The initial charge has two forms of transaction, `DIRECT_CAPTURE` and `RESERVE_CAPTURE`.

`DIRECT_CAPTURE` processes the payment immediately, while `RESERVE_CAPTURE`
reserves the payment for capturing at a later date. See:
[What is the difference between "Reserve Capture" and "Direct Capture"?](https://github.com/vippsas/vipps-developers/blob/master/faqs/reserve-and-capture-faq.md#what-is-the-difference-between-reserve-capture-and-direct-capture)
in the Vipps FAQ for more details.

`RESERVE_CAPTURE` must be
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

A reserved charge can be captured with the
[`POST:/agreements/{agreementId}/charges/{chargeId}/capture`][capture-charge-endpoint] endpoint
when the product is shipped.

### Campaigns

A campaign in recurring is a period where the price is lower than usual, and
this is communicated to the customer with the original price shown for comparison.
Campaigns cannot be used in combination with [variable amount](#recurring-agreements-with-variable-amount).

#### Campaigns in V2 API

![screen-legacy-campaign](images/campaigns/screens/legacy-campaign.png)

In order to start a campaign, the campaign field must be added either to the
[`POST:/agreements`][draft-agreement-endpoint-v2] request
for a campaign in the start of an agreement, or to the
[`PATCH:/agreements/{agreementId}`][update-agreement-patch-endpoint] request
for an ongoing agreement. When adding a campaign
while drafting a new agreement, the start date is ignored and the current
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

| Field           | Description                                                                                                             |
|-----------------|-------------------------------------------------------------------------------------------------------------------------|
| `start`         | Start date of campaign offer, if you are creating a agreement this is set to default now, and not an available variable |
| `end`           | End date of campaign offer, can not be in the past                                                                      |
| `campaignPrice` | The price that will be shown for comparison                                                                             |

#### (Coming soon - WORK IN PROGRESS) Campaigns in V3 API
In V3, we introduce 4 different campaign types: price campaign, period campaign, event campaign, and full flex campaign.
See more about the different campaign types in the table below.

| Campaign types       | Description                                                                                             | Example                                                                 |
|----------------------|---------------------------------------------------------------------------------------------------------|-------------------------------------------------------------------------|
| `price campaign`     | Different interval price until specified date. Same interval as agreement.                              | 1kr every week until 2022-12-25T00:00:00Z and then 50kr every week      |
| `period campaign`    | A set price for a given duration. A duration is defined by a number of periods (DAY, WEEK, MONTH, YEAR) | 10 weeks for 1kr and then 349kr every month                             |
| `event campaign`     | A set price until a given event date with a text describing the event                                   | 1kr until Christmas and then 349kr every month                          |
| `full flex campaign` | Different price and interval until a given date                                                         | 100kr every month until 2023-01-01T00:00:00Z and then 1000kr every year |

In order to start a campaign, the `campaign` field has to be added to the agreement draft body in the
[`draft agreement`][draft-agreement-endpoint-v3] call.

##### Price campaign

![price-campaign-explanation](images/campaigns/price-campaign.png)  

```json
{
  "campaign": {
    "type": "PRICE_CAMPAIGN",
    "end": "2022-12-25T00:00:00Z",
    "price": 100
  }
}
```
| Field   | Description                                                                 |
|---------|-----------------------------------------------------------------------------|
| `type`  | The type of the campaign                                                    |
| `price` | The price that the customer will pay for each interval during the campaign  |
| `end`   | The end date of the campaign                                                |

![screen-price-campaign](images/campaigns/screens/price-campaign.png)

##### Period campaign

![period-campaign-explanation](images/campaigns/period-campaign.png)

```json
{
 "campaign": {
    "type": "PERIOD_CAMPAIGN",
    "price": 100,
    "period": {
      "unit": "WEEK",
      "count": 10
    }
  }
}
```
| Field    | Description                                                                                                             |
|----------|-------------------------------------------------------------------------------------------------------------------------|
| `type`   | The type of the campaign                                                                                                |
| `price`  | The price that the customer will pay for the period of the campaign                                                     |
| `period` | The period where the campaign price is applied. Consist of a Unit and a Count, example; { "unit": "MONTH", "count": 1 } |

![screen-price-campaign](images/campaigns/screens/period-campaign.png)

##### Event campaign

![event-campaign](images/campaigns/event-campaign.png)

```json
{
"campaign": {
    "type": "EVENT_CAMPAIGN",
    "price": 100,
    "eventDate": "2022-09-01T00:00:00Z",
    "eventText": "To the end of august"
  }
}
```
| Field       | Description                                               |
|-------------|-----------------------------------------------------------|
| `type`      | The type of the campaign                                  |
| `price`     | The price that the customer will pay until the event date |
| `eventDate` | The date of the event marking the end of the campaign     |
| `eventText` | The event text to display to the end user                 |

**Note:** We recommend to start the event text with lowercase for better user experience. See example below.

![screen-event-campaign](images/campaigns/screens/event-campaign.png)

##### Full flex campaign

**Note:** Contact Vipps before creating a draft agreement with a full flex campaign.
See [contact us](https://vippsas.github.io/vipps-developer-docs/docs/vipps-developers/contact).

![full-flex-campaign](images/campaigns/full-flex-campaign.png)

```json
{
 "campaign": {
    "type": "FULL_FLEX_CAMPAIGN",
    "price": 10000,
    "end": "2023-01-01T00:00:00Z",
   "interval": {
     "unit": "MONTH",
     "count": 1
   }
  }
}
```
| Field      | Description                                                                                                                       |
|------------|-----------------------------------------------------------------------------------------------------------------------------------|
| `type`     | The type of the campaign                                                                                                          |
| `price`    | The price that the customer will pay for each interval during the campaign                                                        |
| `end`      | The end date of the campaign                                                                                                      |
| `interval` | The payment interval where the campaign price is applied. Consist of a Unit and a Count, example; { "unit": "MONTH", "count": 1 } |

![screen-full-flex-campaign](images/campaigns/screens/full-flex-campaign.png)

##### Product description guidelines for agreements with campaigns.
We do not recommend you to use `Product Description` for agreements with a campaign.
We see that the user experience is not optimal when a lot of text is "squeezed" in the purple bubble displaying an agreement.
`Product description` will be, at a point in the future, phased out in order to improve user experience.

### Retrieve an agreement

A newly created agreement will be in status `PENDING` for 10 minutes before it expires.
If the customer approves the agreement, and the initialCharge (if provided) is successfully
processed, the agreement status will change to `ACTIVE`.

The approved agreement is retrieved from the
[`GET:/agreements/{agreementId}`][fetch-agreement-endpoint] endpoint
with `"status":"ACTIVE"` when the customer has approved the agreement.

See [Agreement states](#agreement-states).

This is an example response from a call to the
[`GET:/agreements/{agreementId}`][fetch-agreement-endpoint] endpoint:

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

_Recurring has functionality to charge a variable amount each interval. See:
[Recurring agreements with variable amount](#recurring-agreements-with-variable-amount)._

Each specific charge on an agreement must be scheduled by the merchant, a
minimum of two days before the payment will occur (it is minimum one day in the test environment).

Charge the customer for each period with the
[`POST:/agreements/{agreementId}/charges`][create-charge-endpoint] endpoint.
`due` will define for which date the charge will be performed.
This date has to be at a minimum two days in the
future (it is minimum one day in the test environment), and all charges `due` in
30 days or less are visible for users in Vipps.

Example: If the charge is _created_ on the 25th, the earliest the charge can be
_due_ is the 27th (25+2). This is so that the user can be informed about the
upcoming charge. The user is only shown one charge per agreement, in order to
not overwhelm the user when doing daily or weekly charges.

A recurring charge has two forms of transaction, `DIRECT_CAPTURE` and `RESERVE_CAPTURE`.
**Note:** `RESERVE_CAPTURE` transaction type is only available in the V3 api (Coming soon - WORK IN PROGRESS)

`DIRECT_CAPTURE` processes the payment immediately, while `RESERVE_CAPTURE`
reserves the payment for capturing at a later date. See:
[What is the difference between "Reserve Capture" and "Direct Capture"?](https://github.com/vippsas/vipps-developers/blob/master/faqs/reserve-and-capture-faq.md#what-is-the-difference-between-reserve-capture-and-direct-capture)
in the Vipps FAQ for more details.

`RESERVE_CAPTURE` must be used when selling physical goods or a need to provide access at a later point.

The advantage to using reserve capture is that you can release the reservation immediately:

- For a reserved payment, the merchant can make a /cancel call to immediately release the reservation and make it available in the customer's account.
- For a captured payment, the merchant must make a /refund call. It then takes a few days before the amount is available in the customer's account.

See the [`POST:/agreements/{agreementId}/charges`][create-charge-endpoint] endpoint definition for examples.

Also see check [orderId recommendations](#orderid-recommendations) before creating charges.

### Capture a charge

Capture payment allows the merchant to capture the reserved amount of a charge.
The API allows for both a full amount capture and a partial amount capture (partial amount only available in API V3)(Coming soon)

The amount to capture cannot be higher than the reserved amount.
According to Norwegian regulations, capture cannot be done before the goods have been shipped.
The `description` text is mandatory and is displayed to the end user in the Vipps app.

Capture is done with the [`POST:/agreements/{agreementId}/charges/{chargeId}/capture`][capture-charge-endpoint] endpoint.

**Please note:** It is important to check the response of the /capture call. The capture is only successful when the response is `HTTP 204 No Content`.

Capture can be made up to 180 days after reservation. Attempting to capture an older payment will result in `HTTP 400 Bad Request`.

### Partial capture (Coming soon)

Partial capture may be used in cases where a partial order is shipped or for other reasons.
Partial capture can be called as many times as required while remaining reserved amount is available.

If one or more partial capture have been made, any remaining reserved amount will be automatically released after a few days.
See [For how long is a payment reserved](https://github.com/vippsas/vipps-developers/blob/master/faqs/reserve-and-capture-faq.md#for-how-long-is-a-payment-reserved)
in the Vipps FAQ for more details.

If you cancel a charge that is `PARTIALLY_CAPTURED`, the remaining funds on the charge will be released back to the customer.

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
`{agreement.productName}` whereas the description is set per charge, i.e.
`{charge.description}`. For example, a charge on an agreement with product
name "Premier League subscription" with description "October" would look like
the following screenshot:

![Charge description example](images/charge_descriptions_example.png)

When the charge is completed (the money has been moved), the payment will
show up in the users' payment history. In the payment history a charge from
Vipps recurring payment will have a description with follow format
`{agreement.ProductName} - {charge.description}`.

This is an example of a request body for the [`POST:/agreements/{agreementId}/charges`][create-charge-endpoint] call:

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

### Cancel a charge

You can cancel charges that are in the `pending`, `due` or `reserved` state.
If you cancel a charge that is `PARTIALLY_CAPTURED`, the remaining funds on the charge will be released back to the customer.

**Note:** If you cancel an agreement, there is no need to cancel the charges that belong to the agreement. This will be done automatically by Vipps.

A charge can be cancelled with the [`DELETE:/agreements/{agreementId}/charges/{chargeId}`][cancel-charge-endpoint] endpoint.

### Charge times

Charge _attempts_ are primarily made two times during the day: 07:00 and 15:00 UTC.
Vipps may do extra attempts and/or change this without notice.
The processing of charges typically takes around one hour, however this varies, and we do not guarantee any time.
This is the same both for our production and test environment.
Subsequent attempts are made according to the `retryDays` specified.

**Note:** Payments _might_ get processed any time during the day
(07:00 UTC - 23:59 UTC) due to special circumstances requiring it.

**Note:** Since payments _can_ be processed any time (07:00UTC - 23:59 UTC) it is advisable to fetch the charge at/after 00:00 UTC the day after the last retry day to be sure you get the last status.

When a charge has reached its `due` date, the status of the charge will be
`DUE` until the charge is successful, for as long as the merchant has
specified with `retryDays`.

**Important:** Vipps does not "leak" the customers' information about insufficient funds,
blocked cards, etc. Users are informed about all such problems in Vipps,
which is the only place they can be corrected. The merchant's customer service
should always ask the user to check in Vipps if a charge has failed.

### Charge retries

Vipps will retry the charge for the number of days specified in `retryDays`.
The maximum number of `retryDays` is 14.

The status of a charge will be `DUE` while Vipps is taking care of business,
from the `due` date until the charge has succeeded, or until the
`retryDays` have passed without a successful charge.
The final status will be `CHARGED` or `FAILED`.
See the API documentation for more details.

See: [Charge states](#charge-states).

**IMPORTANT:** Vipps does not provide details about each charge attempt to the merchant,
but helps the user to correct any problems in Vipps.
This results in a _very_ high success rate for charges.

### Retrieve a charge

A charge can be retrieved with the [`GET:/agreements/{agreementId}/charges/{chargeId}`][fetch-charge-endpoint] endpoint.

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
  "failureReason": "user_action_required",
  "failureDescription": "User action required"
}
```

**Please note:** `failureReason` and `failureDescription` are experimental, and
will soon be replaced by an event log. Subscribe to the technical newsletter
to get updates: [Technical newsletter for developers](https://vippsas.github.io/vipps-developer-docs/docs/vipps-developers/newsletters).

See more about [charge failure reason](#charge-failure-reasons).

See: [Charge states](#charge-states).

### List charges

All charges, including the optional initial charge, for an agreement can be retrieved with the
[`GET:/agreements/{agreementId}/charges`][list-charges-endpoint] endpoint.

## Manage charges and agreements

It is the merchant's responsibility to manage and update charges and agreements,
and to use the API to make sure everything is in sync.

### Agreement states

| # | State      | Description                                                                          |
|:--|:-----------|:-------------------------------------------------------------------------------------|
| 1 | `PENDING`  | Agreement has been created, but not approved by the user in Vipps yet |
| 2 | `ACTIVE` | The agreement has been confirmed by the end user in Vipps and can receive charges |
| 3 | `STOPPED`  | Agreement has been stopped, either by the merchant by the [`PATCH:/agreements/{agreementId}`][update-agreement-patch-endpoint] endpoint, or by the user by cancelling or rejecting the agreement. |
| 4 | `EXPIRED` | The user did not accept, or failed to accept (due to processing an `initialCharge`), the agreement in Vipps |

### Update an agreement

A merchant can update an agreement by calling the
[`PATCH:/agreements/{agreementId}`][update-agreement-patch-endpoint] endpoint.
The following properties are available for updating:

```json
{
  "productName": "A new name",
  "productDescription": "A new description",
  "merchantAgreementUrl": "https://www.example.com/vipps-subscriptions/1234/",
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

**Please note:** As a `PATCH` operation all parameters are optional. However,
when setting an agreement status to `STOPPED` no other changes are allowed.
Attempts at changing other properties while stopping an agreement will result
in a `400 Bad Request` response.

### Pause an agreement

If there should be a pause in an agreement, like a temporary stop of a
subscription: Simply do not create any charges during the pause.

We recommended not to set the agreement status to `STOPPED`. `STOPPED` agreements cannot be reactivated.

### Stop an agreement

When a user notifies the merchant that they want to cancel a subscription or
service, the merchant must ensure that the status of the recurring agreement is
set to `STOPPED` at a suitable time.

Stopping an agreement results in cancellation of any charges that are DUE/PENDING at the time of stopping it,
and it will not be possible to create new charges for a stopped agreement.

We recommend that the recurring agreement remains `ACTIVE` for as long as the
user has access to the service.    
For example; if the user cancels their subscription, but they are still able to
use the service until the end of the billing cycle, the agreement should only be
set to `STOPPED` at the end of the billing cycle.

Since `STOPPED` agreements cannot be reactivated, a benefit of waiting until
the "end of service" before setting the agreement status to `STOPPED` is that
the merchant will be able to reactivate the user's subscription without having
to set up a new agreement.

### Charge states

This table has all the details for the charge states returned by the
[`GET:/agreements/{agreementId}/charges/{chargeId}`][fetch-charge-endpoint] endpoint:

| State                | Description                                                                                                                                   |
|:---------------------|:----------------------------------------------------------------------------------------------------------------------------------------------|
| `PENDING`            | The charge has been created, but _may_ not yet be visible in Vipps. **Please note:** All charges due in 30 days or less are visible in Vipps. |
| `DUE`                | The charge is visible in Vipps and will be processed on the `due` date for `retryDays`.                                                       |
| `PROCESSING`         | The charge status is unknown but should be processed.                                                                                         |
| `CHARGED`            | The charge has been successfully completed.                                                                                                   |
| `FAILED`             | The charge has failed because of an expired card, insufficient funds, etc. Vipps does not provide the details to the merchant.                |
| `REFUNDED`           | The charge has been refunded. Refunds are allowed up to 365 days after the capture date.                                                      |
| `PARTIALLY_REFUNDED` | A part of the captured amount has been refunded.                                                                                              |
| `RESERVED`           | An initial charge with `transactionType` set to `RESERVE_CAPTURE` changes state to `CHARGED` when captured successfully.                      |
| `CANCELLED`          | The charge has been cancelled by the merchant.                                                                                                |

**IMPORTANT:** Vipps does not provide details about each charge attempt to the merchant,
but helps the user to correct any problems in Vipps.
This results in a _very_ high success rate for charges.

### Example charge flows

Scenario: Everything goes as it should: The user has money, and the charge is successful on the `due` date:
* `PENDING` -> `DUE` (just for the one due day)-> `CHARGED`

Scenario: The user does not have funds and `retryDays = 0`:
* `PENDING` -> `DUE` -> `FAILED`

Scenario: The user does not have funds on the `due` date, `retryDays = 10`, and has funds on the fifth day:
* `PENDING` -> `DUE` (for five days) -> `CHARGED`

**Please note:** Since charges are polled by the merchant, it is possible that
the charge status appears to "skip" a transition, e.g. moving directly from
`PENDING` to `CHARGED`, or even from `PENDING` to `REFUNDED`
depending on your systems.

## Charge failure reasons

**Please note:** `failureReason` and `failureDescription` are experimental, and
will soon be replaced by an event log. Subscribe to the technical newsletter
to get updates: [Technical newsletter for developers](https://vippsas.github.io/vipps-developer-docs/docs/vipps-developers/newsletters).

When fetching a charge through the API, you can find two fields in the response
body to identify why the charge failed `failureReason` and `failureDescription`.

An example from a response:

```json
{
  "status": "FAILED",
  "type": "RECURRING",
  "failureReason": "user_action_required",
  "failureDescription": "User action required"
}
```

Here is a list of possible values for `failureReason`, their respective descriptions and possible actions that the user/merchant could take.

| Reason                 | Description                                                                                                                                                                                                                                 | Action                                                                                                                                                                                   |
|------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| user_action_required   | Payment failed. Could be lack of funds, card is blocked for ecommerce, card is expired. If you want to send an email or similar to the user, you should encourage them to open Vipps and check the payment there to see why it is not paid. | User will get notified in Vipps and need to take action. This could be to add funds to the card or change the card on the agreement.                                                     |
| charge_amount_too_high | Amount is higher than the user's specified max amount                                                                                                                                                                                       | The user have a lower `maxAmount` on the variableAmount agreement than the amount of the charge. The user must update their `maxAmount` on the agreement for the charge to be processed. |
| non_technical_error    | Payment failed. Could be that the user has deleted their Vipps profile.                                                                                                                                                                     | The user needs to take action in Vipps.                                                                                                                                                  |
| technical_error        | Payment failed due to a technical error in Recurring or a downstream service                                                                                                                                                                | As long as the charge is not in status `FAILED` we are retrying to payment. Contact Vipps for more information if this failure show up on a `FAILED` charge.                             |

### Deprecated failureReasons

The following `failureReasons` are no longer exposed on charges:

| Reason                 | Description                                                                              | Action                                                                                                                                                                                                                      |
|------------------------|------------------------------------------------------------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| insufficient_funds     | Payment was declined by the payer bank due to lack of funds.                             | User must either add funds to the card to cover the difference between the amount to be paid. Alternatively they can change to another, or add a new, payment source that is adequately funded to complete the transaction. |
| invalid_card           | The user tried to pay using a card that has either expired or is disabled by the issuer. | User must change, or add a new, payment source on the agreement in Vipps.                                                                                                                                                   |
| verification_required  | Payment declined because the issuing bank requires verification.                         | Ask the user to change, or add a new, payment source on their agreement in Vipps. Alternatively removing and then adding the card might solve the issue.                                                                    |
| invalid_payment_source | The provided payment source is disabled or does not exist.                               | User must change payment source for the agreement.                                                                                                                                                                          |
| internal_error         | Internal Error / Something went wrong                                                    | The error could not be identified as one of the above. Try to create the charge again, changing or adding payment sources on the agreement, or contact Vipps for more information.                                          |


The user gets more information in Vipps regarding why the Charge did not get charged. If they contact you about failing charges, you should refer them to Vipps. As long as the charge has `retryDays` left, we will continue to try and process the charge and notify the user.

## Userinfo

Vipps offers the possibility for merchants to ask for the user's profile information as part of the payment flow.
This is done through Vipps Userinfo which
You can learn more at the [OIDC Standard](https://openid.net/specs/openid-connect-core-1_0.html#UserInfo).

To enable the possibility to fetch profile information for a user the merchant can add a `scope`
parameter to the [`POST:/agreements`][draft-agreement-endpoint-v2] call.

If the user has not already consented to sharing information from Vipps to the
merchant the user will be asked for such consent before activating the agreement.
Once the agreement has been accepted the merchant can get the profile information
from our Userinfo endpoint.

A users consent to share information with a merchant applies across our services. Thus, if the merchant implements Vipps login in addition to profile information as part of the agreement flow, the merchant can also use Vipps to log the user in without the need for additional consents.

### Scope

| scope            | Description                                                                                                                                                                                                                                                                | User consent required |
|------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|-----------------------|
| `address`        | A list containing the user's addresses. The list always contains the home address from the National Population Register, and can also include work address and other addresses added by the user in Vipps.                                                                 | yes                   |
| `birthDate`      | Birth date. Verified with BankID.                                                                                                                                                                                                                                          | yes                   |
| `email`          | Email address. The flag `email_verified : true` (or `false`) in the response indicates whether the email address is verified.                                                                                                                                              | yes                   |
| `name`           | First, middle and given name. Verified with the National Population Register.                                                                                                                                                                                              | yes                   |
| `phoneNumber`    | Phone number. Verified when creating the Vipps account.                                                                                                                                                                                                                    | yes                   |
| `nin`            | Norwegian national identity number. Verified with BankID. **NB:** Merchants need to apply for access to NIN. See: [Who can get access to NIN and how?](https://github.com/vippsas/vipps-login-api/blob/master/vipps-login-api-faq.md#who-can-get-access-to-nin-and-how)    | yes                   |
| `accountNumbers` | User bank account numbers. **NB:** Merchants need to apply for access to accountNumbers. See: [Who can get access to account numbers and how?](https://github.com/vippsas/vipps-login-api/blob/master/vipps-login-api-faq.md#who-can-get-access-to-accountnumbers-and-how) | yes                   |  

See the API specification for the formats and other details for each scope.

**Please note:** If the e-mail address that is delivered has the flag `email_verified : false`
this address should not be used to link the user to an existing account without
further authentication. Such authentication could be to prompt the user to
log in to the original account or confirm the account linking by having a
confirmation link sent to the email address.

### Userinfo call by call guide

Scenario: You want to complete a payment and get the name and phone number of
a customer.

1. Retrieve the access token by calling the [`POST:/accesstoken/get`][access-token-endpoint] endpoint.
2. Add the scope field to the draft agreement request body and include the scope you wish to get
   access to (valid scope) before calling the [`POST:/agreements`][draft-agreement-endpoint-v2] endpoint.
3. The user consents to the information sharing and accepts the agreement in Vipps.
4. Retrieve the `sub` by calling the [`GET:/agreements/{agreementId}`][fetch-agreement-endpoint] endpoint.
5. Using the sub from step 4, call the [`GET:/vipps-userinfo-api/userinfo/{sub}`][userinfo-endpoint] endpoint to retrieve the user's information.

**Important note:** The API call to the [`GET:/vipps-userinfo-api/userinfo/{sub}`][userinfo-endpoint] endpoint
must _not_ include the subscription key (the `Ocp-Apim-Subscription-Key` header) used for the Recurring API.
This is because userinfo is part of Vipps Login and is therefore _not_ under the same subscription,
and will result in a `HTTP Unauthorized 401` error.

### Example calls

To request this scope add the scope to the initial [`POST:/agreements`][draft-agreement-endpoint-v2] call

Example of request with scope:

```json
{
  "currency": "NOK",
  "customerPhoneNumber":"90000000",
  "interval": "MONTH",
  "intervalCount": 1,
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

Once the user completes the session a unique identifier `sub` can be retrieved with the [`GET:/agreements/{agreementId}`][fetch-agreement-endpoint] endpoint alongside the full URL to Userinfo.

Example `sub` and `userinfoUrl` format:

```json
{
  "sub": "c06c4afe-d9e1-4c5d-939a-177d752a0944",
  "userinfoUrl": "https://api.vipps.no/vipps-userinfo-api/userinfo/c06c4afe-d9e1-4c5d-939a-177d752a0944"
}
```

This `sub` is a link between the merchant and the user and can be used to retrieve
the user's details from the Vipps [`GET:/vipps-userinfo-api/userinfo/{sub}`][userinfo-endpoint] endpoint

The `sub` is based on the user's national identity number ("fødselsnummer"
in Norway), and does not change (except in very special cases).

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

Call the Vipps [`GET:/vipps-userinfo-api/userinfo/{sub}`][userinfo-endpoint] endpoint with the `sub` that was retrieved earlier. See below on how to construct the call.

**Request**

*Headers*

| Header        | Description             |
|---------------|-------------------------|
| Authorization | "Bearer {Access Token}" |

The access token is received on a successful request to the token endpoint described in
[Get an access token](https://github.com/vippsas/vipps-developers/blob/master/vipps-getting-started.md#get-an-access-token)
in the Getting started guide.

**Important note:** Subscription key used for the Recurring API must _not_ be
included. This is because userinfo is part of Vipps Login and is therefore
_not_ under the same subscription, and will result in a `HTTP Unauthorized 401` error.

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

**Please note:** This operation has an "all or nothing" approach, so a user
must accept the agreement and consent to _all_ values in order to complete the
session. If a user chooses to reject the terms the agreement will not be
activated. Unless the whole flow is completed, this will be handled as a
failed agreement by the Recurring API.

## Recurring agreements with variable amount

Recurring with variable amounts offer merchants a way to charge users a different
amount each interval, as long as the amount is lower than the user's specified max amount.

Instead of setting a price when drafting a new agreement, the new
`suggestedMaxAmount` field is set to what the maximum price could be each interval.
`suggestedMaxAmount` is then presented to the user when accepting an agreement,
as a suggestion that indicates the maximum price that could potentially be charged
within each interval.

The user chooses a max amount themselves when accepting the agreement, but we
recommended the user to choose the same amount as `suggestedMaxAmount`. The max
amount can at any time be changed by the user. What the user has picked as their
max amount will be available in the `GET agreement` response. Its recommended
that when you set the `suggestedMaxAmount`, that you set a realistic amount -
as setting it to unrealistic amounts might scare off the user when they accept
the agreement.

### How it works

#### Create agreement

Create an agreement and specify that it's with `variableAmount` and set a
`suggestedMaxAmount` (in øre).

Create agreement request:

```json
{
  "variableAmount": {
    "suggestedMaxAmount": 200000
  },
  "currency": "NOK",
  "interval": "MONTH",
  "intervalCount": 1,
  "merchantRedirectUrl": "https://example.com/confirmation",
  "merchantAgreementUrl": "https://example.com/my-customer-agreement",
  "customerPhoneNumber": "90000000",
  "productDescription": "Access to subscription",
  "productName": "Power company A"
}
```

**Please note:** There is no need to supply the agreement with a `price` field,
this will be ignored since the user picks the allowed max amount themselves.

**Restrictions when using variable amount:**

- There is currently a limit of **20 000 NOK** for the `suggestedMaxAmount`.
- `Campaign` can not be used when the agreement has `variableAmount`.

The user will be presented with the variable agreement in Vipps,
where they can change the max amount they allow to be charged each interval.

Accepting agreement in Vipps:
![variable_amount_accept](images/variable_amount_accept.png)

Variable amount and initial charge can be combined:
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

**Note:** The user will not be alerted by this change by Vipps.

#### Create charge

There are changes in how the interval and amount calculation works for agreements
with `variable amount`. The amount of the charge/charges in the interval can not
be higher than either the `suggestedMaxAmount` or `maxAmount` field, depending on
which is highest. The user will be notified if a charge is created with an amount higher than their specified max amount, and they are encouraged to alter the max amount to a higher amount.

Changes in how intervals and charge rules work:

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

**Please note:** In the examples above the `intervalCount` is 1.
This can be changed as described in the [Intervals](#intervals) section.

#### Charge amount higher than the user's max amount

If the created charge is above the user's `maxAmount`, the charge will be set
to `DUE`. If the user does not update their `maxAmount` to the same or a higher
amount than the charge, it will fail when `due` + `retryDays` is reached, and
the status will be `FAILED`.

GET charge response where amount is higher than the user's max amount:

```json
{
    "id": "chr-ZZt75qs",
    "status": "DUE",
    "due": "2021-06-19T00:01:00Z",
    "amount": 190000,
    "amountRefunded": 0,
    "transactionId": null,
    "description": "Monthly payment",
    "type": "RECURRING"
}
```

The user will also see a failure description on the charge in the app and a
push notification will be sent .

Display of charge failure due to a charge being higher than the `maxAmount` in Vipps:
![variable_amount_charge](images/variable_amount_charge.png)

## Skip landing page

_This functionality is only available for special cases._

Skipping the landing page is only reserved for physical points of sale and vending machines, when there is no display available.

This feature has to be specially enabled by Vipps for eligible sale units: The sale units must be whitelisted by Vipps.

If the `skipLandingPage` property is set to `true` in the
[`POST:/agreements`][draft-agreement-endpoint-v2]
call, it will cause a push notification to be sent to the given phone number
immediately, without loading the landing page.

If the sale unit is not whitelisted, the request will fail and an error message will be returned.

If you want to check if a sale unit is allowed to use `skipLandingPage`:

1. Draft an agreement with `"skipLandingPage": true`.
2. Check the response code and message.
   The API will return an error if attempting to use `skipLandingPage` without being whitelisted.

If you need to skip the landing page for a different reason: contact your
Key Account Manager. If you do not have a KAM: Please log in on
[portal.vipps.no](https://portal.vipps.no),
find the right sale unit and click the email link under the "i" information
bubble. Include a detailed description of why it is not possible to display
the landing page.

**Please note:** When using `skipLandingPage`, the user is not sent to a URL
after completion of the payment. The "result page" is just the confirmation in
Vipps. The `fallback` URL sent in the API request can therefore be the
merchant's main URL, like `https://example.com`, etc.

## HTTP responses

This API returns the following HTTP statuses in the responses:

| HTTP status                | Description                                                       |
|----------------------------|-------------------------------------------------------------------|
| `200 OK`                   | Request successful                                                |
| `201 Created`              | Request successful, resource created                              |
| `400 Bad Request`          | Invalid request, see the error for details                        |
| `401 Unauthorized`         | Invalid credentials                                               |
| `403 Forbidden`            | Authentication ok, but credentials lacks authorization            |
| `404 Not Found`            | The resource was not found                                        |
| `409 Conflict`             | Unsuccessful due to conflicting resource                          |
| `422 Unprocessable Entity` | Vipps could not process                                           |
| `429 Too Many Requests`    | Look at [table below to view current rate limits](#rate-limiting) |
| `500 Server Error`         | An internal Vipps problem.                                        |

All error responses contains an `error` object in the body, with details of the
problem.

## Rate limiting

We have added rate-limiting to our API (`HTTP 429 Too Many Requests`) to prevent
fraudulent and wrongful behaviour, and increase the stability and security of
our API. The limits should not affect normal behaviour, but please 
[contact us](https://vippsas.github.io/vipps-developer-docs/docs/vipps-developers/contact)
if you notice any unexpected behaviour.

The "Key" column specifies what we consider to be the unique identifier, and
what we "use to count". The limits are of course not _total_ limits.

| API                                                | Limit          | Key used                                          | Explanation                                               |
|----------------------------------------------------|----------------|---------------------------------------------------|-----------------------------------------------------------|
| [CreateCharge][create-charge-endpoint]             | 2 per minute   | agreementId + chargeId (based on idempotency key) | Two calls per minute per unique agreementId and chargeId  |
| [CancelCharge][cancel-charge-endpoint]             | 5 per minute   | agreementId + chargeId                            | Five calls per minute per unique agreementId and chargeId |
| [CaptureCharge][capture-charge-endpoint]           | 5 per minute   | agreementId + chargeId                            | Five calls per minute per unique agreementId and chargeId |
| [RefundCharge][refund-charge-endpoint]             | 5 per minute   | agreementId + chargeId                            | Five calls per minute per unique agreementId and chargeId |
| [ListAgreements][list-agreements-endpoint]         | 5 per minute   | (per merchant)                                    | Five calls per minute per merchant                        |
| [UpdateAgreement][update-agreement-patch-endpoint] | 5 per minute   | agreementId                                       | Five calls per minute per unique agreementId              |
| [FetchCharge][fetch-charge-endpoint]               | 10 per minute  | agreementId + chargeId                            | Ten calls per minute per unique agreementId and chargeId  |
| [ListCharges][list-charges-endpoint]               | 10 per minute  | agreementId                                       | Ten calls per minute per unique agreementId               |
| [FetchAgreement][fetch-agreement-endpoint]         | 120 per minute | agreementId                                       | 120 calls per minute per unique agreementId               |
| [DraftAgreement][draft-agreement-endpoint-v2]         | 300 per minute | (per merchant)                                    | 300 calls per minute per merchant                         |

**Please note:** The "Key" column is important. The above means that we allow two
CreateCharge calls per minute per unique agreementId and chargeId. This is to prevent
too many CreateCharge calls for the same charge. The overall limit for number of
different payments is far higher than 2.

## Partner keys

In addition to the normal [Authentication](#authentication-and-authorization) we offer _partner keys_,
which let a partner make API calls on behalf of a merchant.

If you are a Vipps partner managing agreements on behalf of Vipps merchants you
can use your own API credentials to authenticate, and then send
the `Merchant-Serial-Number` header to identify which of your merchants you
are acting on behalf of. The `Merchant-Serial-Number` must be sent in the header
of all API requests.

By including the [Vipps HTTP Headers](#vipps-http-headers) you will make
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
partner's MSN. This parameter is also recommended for regular Vipps
merchants making API calls for themselves.

See:
[Vipps Partners](https://github.com/vippsas/vipps-partner).

## Polling guidelines

General guidelines for polling with the
[`GET:/agreements/{agreementId}`][fetch-agreement-endpoint]
endpoint can be found at:


See [Polling guidelines](https://github.com/vippsas/vipps-developers/blob/master/common-topics/polling-guidelines.md) in Common topics, for details.

## Timeouts

See [Timeouts](https://github.com/vippsas/vipps-developers/blob/master/common-topics/timeouts.md)
in Common topics for details.

## Testing

To facilitate automated testing in the
[Vipps Test Environment (MT)][vipps-test-environment],
the Vipps Recurring API provides a [`force accept agreement`][force-accept-agreement-endpoint] endpoint to avoid manual
agreement acceptance in the Vipps app:

The "force approve" endpoint allows developers to approve a payment through the
Vipps Recurring API without the use of Vipps. This is useful for automated testing.
The endpoint is only available in our test environment.

## Recommendations regarding handling redirects

See [Recommendations regarding handling redirects](https://github.com/vippsas/vipps-developers/blob/master/common-topics/redirects.md) in Common topics for details.

## When to use campaigns or initial charge

Vipps recurring payments is a fairly flexible service, that allows you as a merchant to tailor the user experience in Vipps to your needs by utilising the normal agreements, initial charges, campaigns, or a combination of those.

This can be a bit confusing when deciding on which implementation to go for.
In short our advice is to implement support for all our flows, and also implement features in your own systems for moving between the flows depending on the use case.

First a short description on the flows.

### Normal agreement flow

![Normal_agreement](images/normal_agreement.png)

In the normal agreement, the user gets presented with the agreement, agrees to that, and gets sent to a confirmation screen.
On the agreement we present the start date, the price of the agreements, the `productName` and the `product description` which are all defined by the merchant.
We also present an agreement explanation which is used to describe the agreement interval to the user.
For example, for an agreement with `interval=WEEK` and `intervalCount=2`, the agreement explanation will be `hver 2.uke til du sier opp` or `every 2 weeks until cancelled`


This is the preferred flow whenever there is no campaigns or similar present.

### Initial charge flow

![flow_Initial_charge](images/normal_agreement_with_initial_charge.png)

When an initial charge is present and the amount is different from the agreement price (or campaign price), the flow in Vipps will change. First the user gets presented with an overview over both the agreement and the initial charge. The user then proceed to confirm the agreement, and finally they will have to go through the actual payment of the initial charge.

Here we also show `productName` and the agreement explanation on the agreement, as well as `description` on the initial charge. `productName` and `inital charge description` are defined by the merchant. The agreement explanation is created by Vipps based on the interval and the campaign if specified.

Initial charges are designed to be used whenever there is an additional cost in setting up the agreement. This could be bundling of a mobile phone together with a mobile subscription, or a TV setup-box when becoming a customer at a cable company. We do not recommend this flow to be used purely for campaigns, as it could be confusing to the user.

As an example: If you have a campaign of 10 NOK for a digital media subscription for 3 months, and the normal price is 299,- monthly, the user would see both the charge of 10 NOK, and have to confirm the agreement for 299,- monthly, which can lead the user to believe that both will be paid upon entering the agreement.

### Campaign

See [Campaigns](#campaigns) for details about campaigns.

When setting a campaign, this follows the normal agreement flow - with some changes. Instead of showing the ordinary price of the agreement, the campaign price will override this, and the ordinary price will be shown below together with information about when the change from the campaign price to the ordinary price will happen.

This is the preferred flow whenever you have a type of campaign where the subscription has a certain price for a certain interval or time, before it switches over to ordinary price.

**Note:** Campaign is not supported for `variableAmount` agreements.

### Initial charge and campaign

In addition to campaigns and initial charges being available as individual flows, they can also be combined. In this case, the user would see first a summary of both the agreement, including the campaign as described in the sections on campaigns, as well as the initial charge. Again, all fields described in previous flows are available for the merchant to display information to the user.

Ideally, this flow is intended for when you have a combination of an additional cost when setting up the agreement, presented as the initial charge, as well as having a limited time offer on the actual subscription.

**Agreement screens with initial and campaign v2**
![screen_initial_charge_legacy_campaign](images/campaigns/screens/legacy-campaign-with-initial-charge.png)

**(Coming soon WORK IN PROGRESS) Agreement screens with initial and campaign v3**
![screen_initial_charge_legacy_campaign](images/campaigns/screens/price-campaign-with-initial-charge.png)


## Questions?

We're always happy to help with code or other questions you might have!
Please create an [issue](https://github.com/vippsas/vipps-recurring-api/issues),
a [pull request](https://github.com/vippsas/vipps-recurring-api/pulls),
or [contact us](https://vippsas.github.io/vipps-developer-docs/docs/vipps-developers/contact).

Sign up for our [Technical newsletter for developers](https://vippsas.github.io/vipps-developer-docs/docs/vipps-developers/newsletters).

[list-agreements-endpoint]: https://vippsas.github.io/vipps-developer-docs/api/recurring#tag/Agreement-v2-endpoints/operation/ListAgreements
[draft-agreement-endpoint-v2]: https://vippsas.github.io/vipps-developer-docs/api/recurring#tag/Agreement-v2-endpoints/operation/DraftAgreement
[draft-agreement-endpoint-v3]: https://vippsas.github.io/vipps-developer-docs/api/recurring/#tag/Agreement-v3-endpoints/operation/DraftAgreementV3
[fetch-agreement-endpoint]: https://vippsas.github.io/vipps-developer-docs/api/recurring#tag/Agreement-v2-endpoints/operation/FetchAgreement
[update-agreement-patch-endpoint]: https://vippsas.github.io/vipps-developer-docs/api/recurring#tag/Agreement-v2-endpoints/operation/UpdateAgreementPatch
[force-accept-agreement-endpoint]: https://vippsas.github.io/vipps-developer-docs/api/recurring#tag/Agreement-v2-endpoints/operation/acceptUsingPATCH
[list-charges-endpoint]: https://vippsas.github.io/vipps-developer-docs/api/recurring#tag/Charge-v2-endpoints/operation/ListCharges
[create-charge-endpoint]: https://vippsas.github.io/vipps-developer-docs/api/recurring#tag/Charge-v2-endpoints/operation/CreateCharge
[fetch-charge-endpoint]: https://vippsas.github.io/vipps-developer-docs/api/recurring#tag/Charge-v2-endpoints/operation/FetchCharge
[cancel-charge-endpoint]: https://vippsas.github.io/vipps-developer-docs/api/recurring#tag/Charge-v2-endpoints/operation/CancelCharge
[capture-charge-endpoint]: https://vippsas.github.io/vipps-developer-docs/api/recurring#tag/Charge-v2-endpoints/operation/CaptureCharge
[refund-charge-endpoint]: https://vippsas.github.io/vipps-developer-docs/api/recurring#tag/Charge-v2-endpoints/operation/RefundCharge
[userinfo-endpoint]: https://vippsas.github.io/vipps-developer-docs/api/recurring#tag/Userinfo-Endpoint/operation/getUserinfo
[access-token-endpoint]: https://vippsas.github.io/vipps-developer-docs/api/recurring#tag/Authorization-Service/operation/getAccessToken
[vipps-test-environment]: https://github.com/vippsas/vipps-developers/blob/master/developer-resources/test-environment.md
