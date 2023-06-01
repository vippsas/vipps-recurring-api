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
[API Lifecycle](https://developer.vippsmobilepay.com/docs/vipps-developers/common-topics/api-lifecycle/).

## June 2023
* New field returned by the [`GET:/agreements/{agreementId}`][fetch-agreement-endpoint] endpoint: 
    * `vippsConfirmationUrl`: used to redirect the user to the Vipps landing page in a desktop flow (with `https://`), or to the Vipps app in a mobile flow (with `vipps://`), where the user can then approve the agreement

## May 2023

* New fields returned by the [`GET:/agreements/{agreementId}`][fetch-agreement-endpoint] endpoint: 
    * `merchantAgreementUrl`: URL where Vipps can send the customer to view/manage their subscription
    * `merchantRedirectUrl`: URL where customer should be redirected after the agreement has been approved/rejected in the Vipps mobile application
    * `created`: Date when agreement was created in ISO 8601 format

* New fields returned by the [`GET:/agreements/{agreementId}/charges/{chargeId}`](https://developer.vippsmobilepay.com/api/recurring/#tag/Charge-v3-endpoints/operation/FetchChargeV3) endpoint: 
    * `retryDays`: The service will attempt to charge the customer for the number of days specified in `retryDays` after the `due` date
    * `externalAgreementId`: Can be used by the merchant to map the `agreementId` to an id in a subscription system or similar

## April 2023 

* New **optional** field `externalId` for agreements introduced. This field can be used to store an external id for the agreement. Can be set in the request body of the [`POST:/agreements`][draft-agreement-endpoint] request. `externalId` will be returned by the [`GET:/agreements/{agreementId}`][fetch-agreement-endpoint] endpoint. It is also possible to update it using the [`PATCH:/agreements/{agreementId}`][update-agreement-patch-endpoint] endpoint. 

* New **optional** field `countryCode` for agreements introduced. Can be set in the request body of the [`POST:/agreements`][draft-agreement-endpoint] request. `countryCode` will also be returned by the [`GET:/agreements/{agreementId}`][fetch-agreement-endpoint] endpoint.

* New field `uuid` (UUID representation of agreement id) returned by the [`GET:/agreements/{agreementId}`][fetch-agreement-endpoint] response. 

* New **optional** field `externalId` for charges introduced. This field can be used to store an external id for the charge. Can be set in the [`POST:/agreements/{agreementId}/charges`][draft-agreement-endpoint] request.


## March 2023

* The new endpoint [`GET:/charges/{chargeId}`](https://developer.vippsmobilepay.com/api/recurring/#tag/Charge-v3-endpoints/operation/FetchChargeByIdV3) makes it possible to retrieve a charge specified by `chargeId`, without knowing the `agreementId` (unlike [`GET:/agreements/{agreementId}/charges/{chargeId}`](https://developer.vippsmobilepay.com/api/recurring/#tag/Charge-v3-endpoints/operation/FetchChargeV3)). The resulting charge now contains the `agreementId`. Its purpose is to simplify investigations when the merchant lost track of which charge belongs to which agreement.

## December 2022

* v3 is now available. v3 includes new and improved functionality for campaigns, the ability to reserve and capture charges, and several technical improvements. The [migration guide](https://developer.vippsmobilepay.com/docs/APIs/recurring-api/v2-to-v3-migration-guide/) and [quick start](https://developer.vippsmobilepay.com/docs/APIs/recurring-api/vipps-recurring-api-quick-start/) provide more details for upgrading to v3.  v2 will be phased out and will no longer be available from November 1, 2023.


[fetch-agreement-endpoint]: https://developer.vippsmobilepay.com/api/recurring#tag/Agreement-v3-endpoints/operation/FetchAgreementV3
[get-charge-by-id]: https://developer.vippsmobilepay.com/api/recurring#tag/Charge-v3-endpoints/operation/FetchChargeByIdV3
[draft-agreement-endpoint]: https://developer.vippsmobilepay.com/api/recurring#tag/Agreement-v3-endpoints/operation/DraftAgreementV3
[create-charge-endpoint]: https://developer.vippsmobilepay.com/api/recurring#tag/Charge-v3-endpoints/operation/CreateChargeV3
[update-agreement-patch-endpoint]: https://developer.vippsmobilepay.com/api/recurring#tag/Agreement-v3-endpoints/operation/UpdateAgreementPatchV3

 
