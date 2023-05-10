<!-- START_METADATA
---
title: Recurring API changelog
sidebar_label: Changelog
sidebar_position: 26
pagination_next: null
pagination_prev: null
---
END_METADATA -->

# Changelog

All notable changes to the current API will be documented in this file. 
 

## March 2023

* The new endpoint GET:/charges/{chargeId} makes it possible to retrieve a charge specified by `chargeId`, without knowing the `agreementId`. The resulting charge now contains the `agreementId`. Its purpose is to simplify investigations when the merchant lost track of which charge belongs to which agreement.

## December 2022

* v3 is now available. This version includes new and improved functionality for campaigns, the ability to reserve and capture charges, and several technical improvements.
* Upgrading to v3 requires some technical changes, but the overall flow and logic remain the same as in v2.
* The [migration guide](https://developer.vippsmobilepay.com/docs/APIs/recurring-api/v2-to-v3-migration-guide/) and [quick start](https://developer.vippsmobilepay.com/docs/APIs/recurring-api/vipps-recurring-api-quick-start/) provide more details for upgrading to v3.
* The button text for agreements without an initial charge has been improved to "Confirm and start" (for no charge) and "Pay and start" (for agreements with an initial charge).
* Vipps Recurring API v2 will be phased out and will no longer be available from June 1, 2023.

 