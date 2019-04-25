# Vipps Recurring API

**Please note:** This API is not officially launched.

The Vipps Recurring API delivers recurring payment functionality for a merchant to create a payment agreement with a customer for fixed interval payments. When the agreement is accepted by the end user the merchant can send charges that will be automatically processed on the due date.

**API documentation:** https://vippsas.github.io/vipps-recurring-api/

**Terminology**

| Term |  Description                                    |
|:-----|:----------------------------------------------- |
| Agreement         | A payment subscription with a set of parameters that a customer agrees to  |
| Charge         | A single payment within an agreement |
| Idempotency | The property of endpoints to be called multiple times without changing the result beyond the initial application. |

## How to perform recurring payments

1. Draft an agreement to be approved with [`POST:/draftAgreement`](https://vippsas.github.io/vipps-recurring-api/#/draft-agreement-controller/registerUsingPOST). In the response an `agreementResource` is created with an `agreementId`. This `agreementResource` is a complete URL for performing a [`GET:/agreement/{agreementId}`](https://vippsas.github.io/vipps-recurring-api/#/agreement-controller/getUsingGET) request.

2. The approved agreement is retrieved from [`GET:/agreement/{agreementId}`](https://vippsas.github.io/vipps-recurring-api/#/agreement-controller/getUsingGET) with `"status":"active"` when customer approves the agreement.

3. Create charges on the agreement with [`POST:/charge/{agreementId}`](https://vippsas.github.io/vipps-recurring-api/#/charge-controller/createUsingPOST).

4. Manage charges and agreements with:  
* [`DELETE:/charge/{agreementId}/{chargeId}`](https://vippsas.github.io/vipps-recurring-api/#/charge-controller/cancelUsingDELETE)  
* [`POST:/charge/{agreementId}/{chargeId}/refund`](https://vippsas.github.io/vipps-recurring-api/#/charge-controller/refundUsingPOST)  
* [`POST:/agreement/{agreementId}`](https://vippsas.github.io/vipps-recurring-api/#/agreement-controller/updateUsingPOST)

### Step 1: Draft an agreement

The following code illustrates how to create an agreement:

[`POST:/draftAgreement`](https://vippsas.github.io/vipps-recurring-api/#/draft-agreement-controller/registerUsingPOST)
```json
{
  "currency": "NOK",
  "customerPhoneNumber":"90000000",
  "interval": "WEEK",
  "intervalCount": 2,
  "isApp": false,
  "merchantRedirectUrl": "https://vipps.no",
  "merchantAgreementUrl": "https://vipps.io/terms",
  "price": 135,
  "productDescription": "Access to all games of English top football",
  "productName": "Premier League Package"
}
```

Agreements can be initiated with or i without an initial charge.

| # | Agreement      | Description                                                                          |
|:--|:-----------|:-------------------------------------------------------------------------------------|
| 1 | `Agreement starting now`  | Agreement with an `initialcharge` will only be `active` if the initial charge is processed successfully |
| 2 | `Agreement starting in future`  | Agreement without an `initialcharge` can be approved but no payment will happen until the first charge is provided |

**Intervals**\
Intervals are defined with a interval type `YEAR`, `MONTH`, `WEEK`, or `DAY` and frequency as a count.\
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

#### Initial charge
Initial charge will be performed if the `initialcharge` is provided when creating an agreement. The `amount` has to correspond to the `price` of the agreement.

```json
"initialCharge": {
  "amount": 135,
  "currency": "NOK",
  "description": "Payment for September"
},
```

#### Campaigns
A campaign in recurring is a period where the price is lower than usual, and this is communicated to the customer with the original price shown for comparison. This Functionality is currently being developed and is subject to change.

<img src="images/CampaignExample.PNG" width="185">

In order to start a campaign the campaign field has to be added either to the agreement [`POST:/draftAgreement`](https://vippsas.github.io/vipps-recurring-api/#/draft-agreement-controller/registerUsingPOST) for a campaign in the start of an agreement or update an agreement [`POST:/agreement/{agreementId}`](https://vippsas.github.io/vipps-recurring-api/#/agreement-controller/updateUsingPOST) for an ongoing agreement.
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
| `originalPrice`       | The price that will be shown for comparison   |
### Step 2: Retrieve the approved agreement
The agreement will be possible to accept for 5 minutes before it expires. When customer approves the agreement status will change to `active`

[`GET:/agreement/{agreementId}`](https://vippsas.github.io/vipps-recurring-api/#/agreement-controller/getUsingGET)
```json
{
  "currency": "NOK",
  "id": "agr_5kSeqzFAMkfBbc",
  "interval": "WEEK",
  "intervalCount": 2,
  "price": 135,
  "merchantAgreementUrl": "https://vipps.io/terms",
  "productDescription": "Access to all games of English top football",
  "productName": "Premier League Package",
  "startDate": "2018-08-22",
  "status": "ACTIVE",
}
```

### Step 3: Create a charge
Create a charge for a given agreement. `dueDate` will define for which date the charge will be performed. `hasPriceChanged` must be `true` if the amount for the charge is different from price of the agreement.

**NOTE:** The charges need to have a due date at least 8 days in the future.

[`POST:/charge/{agreementId}`](https://vippsas.github.io/vipps-recurring-api/#/charge-controller/createUsingPOST)
```json
{
  "amount": 234,
  "currency": "NOK",
  "description": "Payment for October",
  "dueDate": "2030-10-28",
  "hasPriceChanged": true,
  "retryDays": 5
}
```

#### Charge retries
Vipps will retry the charge for the number of days specified in `retryDays`. If `retryDays=0` it will be failed after the first attempt.

### Step 4: Manage charges and agreements
Manage charges and agreement

* Cancel charges with [`DELETE:/charge/{agreementId}/{chargeId}`](https://vippsas.github.io/vipps-recurring-api/#/charge-controller/cancelUsingDELETE).
* Refund performed charges with [`POST:/charge/{agreementId}/{chargeId}/refund`](https://vippsas.github.io/vipps-recurring-api/#/charge-controller/refundUsingPOST).
* Update agreements with [`POST:/agreement/{agreementId}`](https://vippsas.github.io/vipps-recurring-api/#/agreement-controller/updateUsingPOST) in case there are any changes.

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
| 2 | `DUE` | The charge will be drawn in 8 days, and can now be viewed by the user in the app                                      |
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
| `204 No Content`    | Request successful, but empty result        |
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
client_id: "<client_id>""
client_secret: "<client_secret>"
Ocp-Apim-Subscription-Key:  <Ocp-Apim-Subscription-Key>

```

The Ocp-Apim-Subscription-Key can be found in Vipps developer portal

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

Every request to the API, needs to have the `Authorization` header with the generated token and the `Ocp-Apim-Subscription-Key`. The header in the request to this API should look like this:

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
