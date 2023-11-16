<!-- START_METADATA
---
title: Recurring API changelog
sidebar_label: Changelog
sidebar_position: 200
pagination_next: null
pagination_prev: null
---
END_METADATA -->

# Changelog

All notable changes to the current API will be documented in this file.
See:
[API Lifecycle](https://developer.vippsmobilepay.com/docs/knowledge-base/api-lifecycle/).

## November 2023

* Landing page update. Due to complexity of which price applies to an agreement, whether it's the agreement price, initial charge price, promotional price etc, the [landing page](https://developer.vippsmobilepay.com/docs/knowledge-base/landing-page/) will show agreement names instead of prices.


## October 2023

* Fixes in the Recurring YAML file: 
  * The **optional** field `externalId` can be set on a initial charge in the request body of the [`POST:/recurring/v3/agreements`][draft-agreement-endpoint] request.
  * `createdAfter` query parameter on the [`GET:/recurring/v3/agreements][list-agreements-endpoint] is of type `int64`
  * `failureReason` field returned in the `ChargeResponseV3` can be null
  * `chargeId` field in the `DraftAgreementResponseV3` can be null
  * `campaign` field in the `AgreementResponseV3` can be null
  * `PricingResponse` can either be of type `VariableAmountPricingResponse` or `LegacyPricingResponse`
 
* New optional field `interval` in the [`PATCH:/recurring/v3/agreements/{agreementId}`][update-agreement-patch-endpoint] endpoint, 
  which allows to update the interval of an active agreement.

## August 2023

* Removed charge-amount-too-high-for-interval validation used in charge creation endpoint [`POST:/recurring/v3/agreements/{agreementId}/charges`][create-charge-endpoint].
  The `suggestedMaxAmount` field for a variable amount agreement should be set to what the maximum amount could be for each charge.

## June 2023

* A new field is returned by the [`GET:/recurring/v3/agreements/{agreementId}`][fetch-agreement-endpoint] endpoint:
  * `vippsConfirmationUrl`: used to redirect the user to the
    [landing page](https://developer.vippsmobilepay.com/docs/knowledge-base/landing-page/)
    in a desktop flow (with `https://`), or to the Vipps or MobilePay app in a mobile flow (with `vipps://`), where the user can then approve the agreement

## May 2023

* New fields returned by the [`GET:/recurring/v3/agreements/{agreementId}`][fetch-agreement-endpoint] endpoint:
  * `merchantAgreementUrl`: URL where Vipps can send the customer to view/manage their subscription
  * `merchantRedirectUrl`: URL where customer should be redirected after the agreement has been approved/rejected in the Vipps mobile application
  * `created`: Date when agreement was created in ISO 8601 format

* New fields returned by the [`GET:/recurring/v3/agreements/{agreementId}/charges/{chargeId}`](https://developer.vippsmobilepay.com/api/recurring/#tag/Charge-v3-endpoints/operation/FetchChargeV3) endpoint:
  * `retryDays`: The service will attempt to charge the customer for the number of days specified in `retryDays` after the `due` date
  * `externalAgreementId`: Can be used by the merchant to map the `agreementId` to an ID in a subscription system or similar

## April 2023

* New **optional** field `externalId` for agreements introduced. This field can be used to store an external ID for the agreement. Can be set in the request body of the [`POST:/recurring/v3/agreements`][draft-agreement-endpoint] request. `externalId` will be returned by the [`GET:/recurring/v3/agreements/{agreementId}`][fetch-agreement-endpoint] endpoint. It is also possible to update it using the [`PATCH:/recurring/v3/agreements/{agreementId}`][update-agreement-patch-endpoint] endpoint.

* New **optional** field `countryCode` for agreements introduced. Can be set in the request body of the [`POST:/recurring/v3/agreements`][draft-agreement-endpoint] request. `countryCode` will also be returned by the [`GET:/recurring/v3/agreements/{agreementId}`][fetch-agreement-endpoint] endpoint.

* New field `uuid` (UUID representation of agreement ID) returned by the [`GET:/recurring/v3/agreements/{agreementId}`][fetch-agreement-endpoint] response.

* New **optional** field `externalId` for charges introduced. This field can be used to store an external ID for the charge. Can be set in the [`POST:/recurring/v3/agreements/{agreementId}/charges`][draft-agreement-endpoint] request.

## March 2023

* The new endpoint [`GET:/recurring/v3/charges/{chargeId}`](https://developer.vippsmobilepay.com/api/recurring/#tag/Charge-v3-endpoints/operation/FetchChargeByIdV3) makes it possible to retrieve a charge specified by `chargeId`, without knowing the `agreementId` (unlike [`GET:/recurring/v3/agreements/{agreementId}/charges/{chargeId}`](https://developer.vippsmobilepay.com/api/recurring/#tag/Charge-v3-endpoints/operation/FetchChargeV3)). The resulting charge now contains the `agreementId`. Its purpose is to simplify investigations when the merchant lost track of which charge belongs to which agreement.

## December 2022

* v3 is now available. v3 includes new and improved functionality for campaigns, the ability to reserve and capture charges, and several technical improvements. The [migration guide](https://developer.vippsmobilepay.com/docs/APIs/recurring-api/v2-to-v3-migration-guide/) and [quick start](https://developer.vippsmobilepay.com/docs/APIs/recurring-api/recurring-api-quick-start/) provide more details for upgrading to v3.  v2 will be phased out and will no longer be available from November 1, 2023.

[fetch-agreement-endpoint]: https://developer.vippsmobilepay.com/api/recurring#tag/Agreement-v3-endpoints/operation/FetchAgreementV3
[list-agreements-endpoint]: https://developer.vippsmobilepay.com/api/recurring/#tag/Agreement-v3-endpoints/operation/ListAgreementsV3
[get-charge-by-id]: https://developer.vippsmobilepay.com/api/recurring#tag/Charge-v3-endpoints/operation/FetchChargeByIdV3
[draft-agreement-endpoint]: https://developer.vippsmobilepay.com/api/recurring#tag/Agreement-v3-endpoints/operation/DraftAgreementV3
[create-charge-endpoint]: https://developer.vippsmobilepay.com/api/recurring#tag/Charge-v3-endpoints/operation/CreateChargeV3
[update-agreement-patch-endpoint]: https://developer.vippsmobilepay.com/api/recurring#tag/Agreement-v3-endpoints/operation/UpdateAgreementPatchV3
