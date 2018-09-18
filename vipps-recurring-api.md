# Vipps Recurring API

This is the API documentation for **Vipps Recurring**

**Status:** 🛠 While we have worked closely with selected partners, and believe that this is
_very_ close to production quality, we are more than happy to receive feedback.

Please use GitHub's built-in functionality for
[issues](https://github.com/vippsas/vipps-recurring-api/issues),
[pull requests](https://github.com/vippsas/vipps-recurring-api/pulls),
or contact us at integration@vipps.no.

Document version: 0.1.0.

## External documentation

### Technical details about the API

Swagger/OAS API documentation is available on GitHub: https://vippsas.github.io/vipps-recurring-api/

### Getting access to the Vipps Developer Portal

See
[the getting started guide](https://github.com/vippsas/vipps-developers/blob/master/vipps-developer-portal-getting-started.md)
for the Vipps eCommerce API for general information about the Vipps Developer Portal.
This is where you create keys to the API.

### Getting an access token

A valid access token is required in order to call this API. This API is provided by
a service called API Management in Azure - think of it as the gateway to the API.
To get a token, follow
[the guide for Vipps eCommerce API](https://github.com/vippsas/vipps-ecom-api/blob/master/vipps-ecom-api.md)

## Terminology

| Term |  Description                                    |
|:-----|:----------------------------------------------- |
| Agreement         | An agreement to a subscription with a set of conditions that a end user agrees to.  |
| Charge         | A single payment within an agreement. |
| NIN         | A national identification number, e.g. SSN in Norway ("fødselsnummer", 11 digits).   |
| MSISDN      | A number uniquely identifying a subscription in a GSM or a UMTS mobile network. Simply put, it is the mapping of the telephone number to the SIM card in a mobile phone. See [MSISDN](https://en.wikipedia.org/wiki/MSISDN). |
| Idempotency | The property of endpoints to be called multiple times without changing the result beyond the initial application. |

# Core functionality

## Facilitation of user subscriptions through Vipps

*Vipps Recurring* offers the ability to facilitate a subscription based payment


# Agreement states

| # | State      | Description                                                                          |
|:--|:-----------|:-------------------------------------------------------------------------------------|
| 1 | `pendingUserApproval`  | Agreement has been created, but not approved by the user in the app yet. |
| 2 | `active` | The Agreement has been confirmed by the end user in the app and can recieve charges                                      |
| 3 | `stoppedByMerchant`  | Agreement has been stopped by the merchant                                   |
| 4 | `stoppedByUser`  | Optional status where if the merchant wants to indicate the Agreement be stopped by user requests                                        |
| 5 | `stoppedByVipps` | Optional status where for some reason Vipps needs to to stop the agreement, for exampel if a user terminates their Vipps Account                                                |

See the detailed state descriptions, and state transitions, at the end of this document.

# HTTP responses

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
| `429 Too Many Requests`  | There is currently a limit of max 20 calls per second\*  |
| `500 Server Error`  | An internal Vipps problem.                  |

All error responses contains an `error` object in the body, with details of the problem.

\*: The limit is cautiously set quite low in the production environment, as we want to
monitor performance closely before increasing the limit.
We count HTTP requests per `client_id` and product (ISP and IPP).
For now, all HTTP requests are counted and rate-limited.
We have previously requested data from integrators about volume, times, etc,
but only received this from one integrator.
If you are able to provide data for your solution, please let us know.

# Authentication and authorization

Vipps has to ensure compliance with governing regulations, including
[GDPR](https://ec.europa.eu/info/law/law-topic/data-protection_en).
This means that we have to make sure that Vipps:

* Does not provide information about client, explicitly or implicitly, to unauthorised entities.
* Does not store information accidentally received, which we are not authorised to see.
* Propagate changes quickly if a user opts out or in to Vipps Recurring.
* Does not store personal information, i.e. invoices, which we do not have the right to see.

## API access token

The first thing that is required is to get the access token to the API. This is described
in the [external documentation section](#external-documentation).

Shortly summarized, you will have to make the following request
(`client_id`, `client_secret` and `Ocp-Apim-Subscription-Key` placeholders must be replaced with real values):

```http
POST https://apitest.vipps.no/api/access-token/jwt-token HTTP/1.1
Host: apitest.vipps.no
Content-Type: application/json
Ocp-Apim-Subscription-Key: <Ocp-Apim-Subscription-Key>

{
	"client_id":"<client_id>",
	"client_secret":"<client_secret>",
	"resource":"https://testapivipps.no/vippsas/recurring-payment-service"
}

```



The request above will return a response similar to this, with the `access_token`:

```http
HTTP 200 OK
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

Every request to the API, , needs to have an `Authorization` header with the generated token.

The header in the request to this API should look like this:

```http
Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1Ni <continued>
```

## Typical flow

1. A user requests a subscription from the merchant.

2. An agreement is sent with [`POST:/api/v1/draftAgreement`](https://vippsas.github.io/vipps-recurring-api/#/draft-agreement-controller/registerUsingPOST). In the  respons is a "agreementResource" field that can be used in step 4.

3. The user approves the agreement.

4. The Merchant retrieves the agreement status. Either by getting all agreements
 [`GET:/api/v1/agreement`](https://vippsas.github.io/vipps-recurring-api/#/agreement-controller/listUsingGET) or the specific agreement
 [`GET:/api/v1/agreement/{agreementId}`](https://vippsas.github.io/vipps-recurring-api/#/agreement-controller/getUsingGET) per now we do not send callback when the user accepts.

5. The merchant is now free to post charges to the given charge using [`POST:/api/v1/charge/{agreementId}`](https://vippsas.github.io/vipps-recurring-api/#/charge-controller/createUsingPOST)

6. Later a merchant might want to check if a user has any failed charges and therefore terminate the subscription. This can be done by getting all charges on the users agreement on  [`GET:/api/v1/charge/{agreementId}`](https://vippsas.github.io/vipps-recurring-api/#/charge-controller/listUsingGET_1)

# Questions or comments?

Please use GitHub's built-in functionality for
[issues](https://github.com/vippsas/vipps-recurring-api/issues),
[pull requests](https://github.com/vippsas/vipps-recurring-api/pulls),
or contact us at integration@vipps.no.
