# Vipps Recurring API

The Vipps Recurring API delivers recurring payment functionality for a merchant
to create a payment agreement with a customer for fixed interval payments.
When the agreement is accepted by the end user the merchant can send charges
that will be automatically processed on the due date.

If you have an issues or suggestions please 
create an [issue](https://github.com/vippsas/vipps-recurring-api/issues)
or a [pull request](https://github.com/vippsas/vipps-recurring-api/pulls). 
If it is a critical issue, or involves sensitive information please send an
email to integration@vipps.no

**API documentation:** https://vippsas.github.io/vipps-recurring-api/

**Terminology**

| Term |  Description                                    |
|:-----|:----------------------------------------------- |
| Agreement         | A payment subscription with a set of parameters that a customer agrees to  |
| Charge         | A single payment within an agreement |
| Idempotency | The property of endpoints to be called multiple times without changing the result beyond the initial application. |

## How to perform recurring payments

1. Draft a new agreement to be approved with [`POST:/agreements`](https://vippsas.github.io/vipps-recurring-api/#/draft-agreement-controller/draftAgreement). The response contains an `agreementResource`, a `vippsConfirmationUrl` and an `agreementId`. This `agreementResource` is a complete URL for performing a [`GET:/agreements/{agreementId}`](https://vippsas.github.io/vipps-recurring-api/#/agreement-controller/getAgreement) request. The `vippsConfirmationUrl` should be used to redirect the user to the Vipps landing page on a Desktop flow, or Vipps app in a mobile flow. Where the user can then approve the agreement.

2. The approved agreement is retrieved from [`GET:/agreements/{agreementId}`](https://vippsas.github.io/vipps-recurring-api/#/agreement-controller/getAgreement) with `"status":"active"` when customer has approved the agreement.

3. Charge the customer for each period with [`POST:/agreements/{agreementId}/charges`](https://vippsas.github.io/vipps-recurring-api/#/charge-controller/createCharge).<br>
Each specific charge on an agreement must be scheduled by the merchant, a minimum of six days before the payment will occur. <br>
**Note:** Vipps will *only* perform a payment transaction on an agreement after being told by the merchant through this endpoint.

4. Manage charges and agreements with:  
* [`DELETE:/agreements/{agreementId}/charges/{chargeId}`](https://vippsas.github.io/vipps-recurring-api/#/charge-controller/cancelCharge)  
* [`POST:/agreements/{agreementId}/charges/{chargeId}/refund`](https://vippsas.github.io/vipps-recurring-api/#/charge-controller/refundCharge)  
* [`PUT:/agreements/{agreementId}`](https://vippsas.github.io/vipps-recurring-api/#/agreement-controller/updateAgreement)

### Step 1: Draft an agreement

The following code illustrates how to create an agreement:

[`POST:/agreements`](https://vippsas.github.io/vipps-recurring-api/#/draft-agreement-controller/draftAgreement)
```json
{
  "currency": "NOK",
  "customerPhoneNumber":"90000000",
  "interval": "WEEK",
  "intervalCount": 2,
  "isApp": false,
  "merchantRedirectUrl": "https://vipps.no",
  "merchantAgreementUrl": "https://vipps.io/terms",
  "price": 49900,
  "productDescription": "Access to all games of English top football",
  "productName": "Premier League subscription"
}
```

The following size limits are inplace on certain variables:
* `productName`: Max length 45 characters
* `productDescription`: Max length 100 characters
* `price`: Greater than 100

Agreements may be initiated with or without an initial charge.

| # | Agreement      | Description                                                                          |
|:--|:-----------|:-------------------------------------------------------------------------------------|
| 1 | `Agreement starting now`  | Agreement with an `initialcharge` will only be `active` if the initial charge is processed successfully |
| 2 | `Agreement starting in future`  | Agreement without an `initialcharge` can be approved but no payment will happen until the first charge is provided |

**Intervals**

Intervals are defined with a interval type `MONTH`, `WEEK`, or `DAY` and frequency as a count.

Example for a bi-weekly subscription:
```json
"interval": "WEEK",
"intervalCount": 2,
```

Example for a quarterly subscription
```json
"interval": "MONTH",
"intervalCount": 3,
```

Example for a yearly subscription
```json
"interval": "MONTH",
"intervalCount": 12,
```

#### Initial charge
Initial charge will be performed if the `initialcharge` is provided when
creating an agreement.

See [Charge Titles](#charge-title) for explanation of how the charge description is shown to the user.

The initial charge has two forms of transaction, `DIRECT_CAPTURE` and `RESERVE_CAPTURE`.  

`DIRECT_CAPTURE` processes the payment imediately, while `RESERVE_CAPTURE` reserves the payment for capturing at a later date, 
this must be used when selling phyisical goods bundled with an agreement. Such as a phone, when subscribing to a agreement for example.

**Note:** `RESERVE_CAPTURE` is not yet implemented and is a currently a no-op.

```json
"initialCharge": {
  "transactionType": "DIRECT_CAPTURE",
  "amount": 19900,
  "currency": "NOK",
  "description": "September"
},
```

#### Campaigns
A campaign in recurring is a period where the price is lower than usual, and
this is communicated to the customer with the original price shown for comparison.

<img src="images/CampaignExample.PNG" width="185">

In order to start a campaign the campaign field has to be added either to the agreement draft [`POST:/agreements`](https://vippsas.github.io/vipps-recurring-api/#/draft-agreement-controller/draftAgreement) for a campaign in the start of an agreement or update an agreement [`PUT:/agreements/{agreementId}`](https://vippsas.github.io/vipps-recurring-api/#/agreement-controller/updateAgreement) for an ongoing agreement. When adding a campaign
while drafting a new agreement the start date is ignored and the current date-time is used. All dates must be in date-time format as according to [RFC-3999](https://www.ietf.org/rfc/rfc3339.txt).

```json
"campaign": {
  "start": "2019-05-01T00:00:00Z",
  "end": "2019-06-01T00:00:00Z",
  "campaignPrice": 49900
}
```

| Field         | Description                                 |
| ------------------- | ------------------------------------------- |
| `start`            | Start date of campaign offer, if you are creating a agreement this is set to default now, and not an available variable  |
| `end`            | End date of campaign offer, can not be in the past |
| `campaignPrice`       | The price that will be shown for comparison   |


### Step 2: Retrieve the approved agreement
The agreement will be in status `PENDING` for 5 minutes before it expires.
If the customer approves the agreement, and the initialCharge (if provided) is successfully 
processed, the agreement status will change to `active`.

[`GET:/agreements/{agreementId}`](https://vippsas.github.io/vipps-recurring-api/#/agreement-controller/getAgreement)
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
  "campaign": null,
}
```
#### Pausing an agreement
If there should be a pause in an agreement, like a temporary stop of a
subscription: Simply do not create any charges during the pause.

####

### Step 3: Create a charge
Create a charge for a given agreement. `due` will define for which date
the charge will be performed. `hasPriceChanged` must be `true` if the amount
for the charge is different from price of the agreement.

#### Charge Title
The title of the charge shown to a user in the Vipps app is in the format `{agreement.ProductName} - {charge.description}`. For example, with the charge below, and the *Premier League* agreement, the app title would read `Premier League subscription - October`

**NOTE:** The charges need to have a due date at least 6 days in the future.

[`POST:/agreements/{agreementId}/charges`](https://vippsas.github.io/vipps-recurring-api/#/charge-controller/createCharges)
```json
{
  "amount": 49900,
  "currency": "NOK",
  "description": "October",
  "due": "2030-10-28",
  "hasPriceChanged": true,
  "retryDays": 5
}
```

**Note** `description` cannot be longer than 45 characters.

#### Charge retries
Vipps will retry the charge for the number of days specified in `retryDays`.
If `retryDays=0` it will be failed after the first attempt.

### Step 4: Manage charges and agreements

* Cancel charges with [`DELETE:/agreements/{agreementId}/charges/{chargeId}`](https://vippsas.github.io/vipps-recurring-api/#/charge-controller/cancelCharge).
* Refund performed charges with [`POST:/agreements/{agreementId}/charges/{chargeId}/refund`](https://vippsas.github.io/vipps-recurring-api/#/charge-controller/refundCharge).
* Update agreements with [`PUT:/agreements/{agreementId}`](https://vippsas.github.io/vipps-recurring-api/#/agreement-controller/updateAgreement) in case there are any changes.

#### Agreement states

| # | State      | Description                                                                          |
|:--|:-----------|:-------------------------------------------------------------------------------------|
| 1 | `PENDING`  | Agreement has been created, but not approved by the user in the app yet |
| 2 | `ACTIVE` | The Agreement has been confirmed by the end user in the app and can receive charges |
| 3 | `STOPPED`  | Agreement has been stopped by the merchant most, typically when the end user wants to cancel the payment agreement |
| 4 | `EXPIRED` | The user did not accept the agreement within the app |

#### Charge states

<img src="images/charge_state.png">

| # | State      | Description                                                                          |
|:--|:-----------|:-------------------------------------------------------------------------------------|
| 1 | `PENDING`  | Charge has been created. |
| 2 | `DUE` | The charge will be drawn in 6 days, and can now be viewed by the user in the app                                      |
| 3 | `CHARGED`  | Charge has been completed
| 4 | `FAILED`  | Charge has failed for some reason. I.E Expired card, insufficient funds, etc.
| 5 | `REFUNDED` | Charge successfully refunded. Timeframe for issuing a refund for a payment is 365 days from the date payment has been captured
| 6 | `PARTIALLY_REFUNDED`| Charge successfully refunded, used if the refund is a partial ammount of the captured amount.


## HTTP responses

This API returns the following HTTP statuses in the responses:

| HTTP status         | Description                                 |
| ------------------- | ------------------------------------------- |
| `200 OK`            | Request successful                          |
| `201 Created`       | Request successful, resource created        |
| `400 Bad Request`   | Invalid request, see the error for details  |
| `401 Unauthorized`  | Invalid credentials                         |
| `403 Forbidden`     | Authentication ok, but credentials lacks authorization  |
| `404 Not Found`     | The resource was not found  |
| `409 Conflict`      | Unsuccessful due to conflicting resource   |
| `422 Unprocessable Entity`     |    |
| `429 Too Many Requests`  | There is currently a limit of max 20 calls per second\*  |
| `500 Server Error`  | An internal Vipps problem.                  |

All error responses contains an `error` object in the body, with details of the problem.

## Authentication and authorization - API access token

For all product request we require the use of a `Authorization` header. This header is required by making a Access Token request with the values `client_id`, `client_secret` and `Ocp-Apim-Subscription-Key`. See the [Access Token swagger](https://vippsas.github.io/vipps-accesstoken-api/#/Authorization_Service/fetchAuthorizationTokenUsingPost) and the [getting started guide](https://github.com/vippsas/vipps-developers/blob/master/vipps-getting-started.md#step-3) for more information.

[`POST:/get`](https://vippsas.github.io/vipps-accesstoken-api/#/Authorization_Service/fetchAuthorizationTokenUsingPost)

```http
POST https://apitest.vipps.no/accesstoken/get HTTP/1.1
Host: apitest.vipps.no
client_id: "<client_id>"
client_secret: "<client_secret>"
Ocp-Apim-Subscription-Key:  <Ocp-Apim-Subscription-Key>

```

The `Ocp-Apim-Subscription-Key` can be found in Vipps developer portal.
See the [Getting started guide](https://github.com/vippsas/vipps-developers/blob/master/vipps-getting-started.md).

The request above will return a response similar to this, with the `access_token`:

```http
HTTP 200 OK
```
```json
{
  "token_type": "Bearer",
  "expires_in": "86398",
  "ext_expires_in": "0",
  "expires_on": "1495271273",
  "not_before": "1495184574",
  "resource": "00000002-0000-0000-c000-000000000000",
  "access_token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1Ni <continued>"
}
```

Every request to the API, needs to have the `Authorization` header with the
generated token and the `Ocp-Apim-Subscription-Key`. The header in the request
to this API should look like this:

```http
Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1Ni <continued>
Ocp-Apim-Subscription-Key:  <Ocp-Apim-Subscription-Key>
...
```

# Questions?

We're always happy to help with code or other questions you might have!
Please create an [issue](https://github.com/vippsas/vipps-recurring-api/issues),
a [pull request](https://github.com/vippsas/vipps-recurring-api/pulls),
or [contact us](https://github.com/vippsas/vipps-developers/blob/master/contact.md).
