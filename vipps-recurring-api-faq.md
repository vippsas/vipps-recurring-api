<!-- START_METADATA
---
title: FAQ
sidebar_position: 45
---
END_METADATA -->

# Vipps Recurring API: Frequently Asked Questions

<!-- START_COMMENT -->

ℹ️ Please use the new documentation:
[Vipps Technical Documentation](https://vippsas.github.io/vipps-developer-docs/).

<!-- END_COMMENT -->

See the
[Vipps Recurring API](vipps-recurring-api.md)
for all the details.

See also:
[Vipps API FAQ](https://vippsas.github.io/vipps-developer-docs/docs/vipps-developers/faqs/).

See also:
[Getting Started](https://vippsas.github.io/vipps-developer-docs/docs/vipps-developers/vipps-getting-started)
guide.

Document version: 1.6.3.

<!-- START_TOC -->

## Table of Contents
- [Migration](#migration)
  - [How do I migrate to the v3 API?](#how-do-i-migrate-to-the-v3-api)
- [Charges/Payments](#chargespayments)
  - [Do I need to store card data?](#do-i-need-to-store-card-data)
  - [Does Vipps automatically create charges for an agreement?](#does-vipps-automatically-create-charges-for-an-agreement)
  - [At what time during the day are charges processed?](#at-what-time-during-the-day-are-charges-processed)
  - [I don't want a charge to fail the first time the transaction fails (insufficient funds / networking issues etc.)](#i-dont-want-a-charge-to-fail-the-first-time-the-transaction-fails-insufficient-funds--networking-issues-etc)
  - [How can I require an initial payment from the user for setting up the agreement?](#how-can-i-require-an-initial-payment-from-the-user-for-setting-up-the-agreement)
  - [Can the charge amount be different from the agreement price?](#can-the-charge-amount-be-different-from-the-agreement-price)
  - [Are there any limits on charging a user?](#are-there-any-limits-on-charging-a-user)
  - [When can I send charges to a user?](#when-can-i-send-charges-to-a-user)
  - [A charge failed, but the customer did not receive any warning](#a-charge-failed-but-the-customer-did-not-receive-any-warning)
  - [What happens to charges if the corresponding agreement is cancelled?](#what-happens-to-charges-if-the-corresponding-agreement-is-cancelled)
  - [If a user's card expires: What happens on the next charge?](#if-a-users-card-expires-what-happens-on-the-next-charge)
  - [What happens to pending charges if the user deletes the payment card?](#what-happens-to-pending-charges-if-the-user-deletes-the-payment-card)
  - [How does a user see the charges I create?](#how-does-a-user-see-the-charges-i-create)
  - [If a user adds or updates a card in vipps, will new recurring charges be made to that card?](#if-a-user-adds-or-updates-a-card-in-vipps-will-new-recurring-charges-be-made-to-that-card)
  - [For how long is a payment reserved?](#for-how-long-is-a-payment-reserved)
- [Variable amount](#variable-amount)
  - [Can I decide the users suggested max amount list](#can-i-decide-the-users-suggested-max-amount-list)
- [Agreements and users](#agreements-and-users)
  - [How do I check my customer's status?](#how-do-i-check-my-customers-status)
  - [Can I look up a user's information?](#can-i-look-up-a-users-information)
  - [Can I look up which user owns an agreement?](#can-i-look-up-which-user-owns-an-agreement)
  - [How can I convert existing agreements to Vipps agreements?](#how-can-i-convert-existing-agreements-to-vipps-agreements)
  - [How can I delete an agreement?](#how-can-i-delete-an-agreement)
  - [Can a user cancel the agreement through the Vipps app?](#can-a-user-cancel-the-agreement-through-the-vipps-app)
  - [How can I move agreements between merchants and sale units?](#how-can-i-move-agreements-between-merchants-and-sale-units)
- [Common problems/errors](#common-problemserrors)
  - [Why do I get the error `merchant.not.allowed.for.recurring.operation`?](#why-do-i-get-the-error-merchantnotallowedforrecurringoperation)
  - [Is there an API for retrieving information about a Vipps user?](#is-there-an-api-for-retrieving-information-about-a-vipps-user)
- [Notifications and error messages](#notifications-and-error-messages)
  - [When do users get push messages?](#when-do-users-get-push-messages)
  - [What is shown to users when charge processing fails?](#what-is-shown-to-users-when-charge-processing-fails)
- [Admin/partners](#adminpartners)
  - [How can I change partners for a merchant?](#how-can-i-change-partners-for-a-merchant)
  - [How can I change partner for my integration with Vipps?](#how-can-i-change-partner-for-my-integration-with-vipps)
  - [Settlement](#settlement)
  - [Invoicing](#invoicing)
- [Questions?](#questions)

<!-- END_TOC -->

## Migration

### How do I migrate to the v3 API?

Please check the [migration guide](v2-to-v3-migration-guide.md) to see the differences between Recurring API v2 and v3.
Please also check the [V3 API definitions](https://vippsas.github.io/vipps-developer-docs/api/recurring).


## Charges/Payments

### Do I need to store card data?

No. Vipps handles all payment details.

### Does Vipps automatically create charges for an agreement?

 No. Vipps does _not_ create charges based on the agreement, this is left up to the merchant to create.
 When a merchant creates a charge, Vipps will actually attempt to charge the customer, starting on the `due date` and for as long as specified in `retryDays`.

 For more details, see our [call-by-call guide](vipps-recurring-api.md#call-by-call-guide)

### At what time during the day are charges processed?
Charges are processed up to two times every day: 07:00 and 15:00 UTC.
Retries are attempted according to the `retryDays` specified.
This applies for both our production and test environment (MT).

### I don't want a charge to fail the first time the transaction fails (insufficient funds / networking issues etc.)
The field `retryDays` in an Agreement allows for this functionality, Vipps will
retry once each day until the value is reached. The valid values are none
(defaulting to 0) or 0-14. We strongly recommend having more than 1 retry day
to account for possible networking issues etc.

### How can I require an initial payment from the user for setting up the agreement?

You need to use
[Initial charge](https://vippsas.github.io/vipps-developer-docs/docs/APIs/recurring-api/vipps-recurring-api#initial-charge).

### Can the charge amount be different from the agreement price?
Yes. See [Are there any limits on charging a user?](#are-there-any-limits-on-charging-a-user).

### Are there any limits on charging a user?
Yes. Within the `interval` period of the agreement, the merchant can charge at
most **5 times** the agreement price cumulatively. There is no limit on the
number of charges which can be sent in the `interval` period.

It is, however, recommended that you update the agreements pricing if there is a price change.
In case of a more significant change, we recommend creating a new Agreement.

It is the merchant's responsibility to make sure the user is informed and understands
the price of the Agreement.

### When can I send charges to a user?
You can create charges once the user's agreement is ACTIVE

See [How do I check my customer's status?](#how-do-i-check-my-customers-status).

### A charge failed, but the customer did not receive any warning
The customer may not have notifications turned on. We always send notifications to the user when a charge processing attempt is not successful, and the user gets a more detailed message when looking at the charge/agreement in the app.

### What happens to charges if the corresponding agreement is cancelled?
All charges in a `PENDING`, `DUE` or `RESERVED` state will be cancelled if the Agreement is stopped.  
**Note**: This also includes the `initial charge` if it's currently `RESERVED`.
So if the merchant needs to charge the user for the initial charge; then this needs to be done before the agreement is stopped.

### If a user's card expires: What happens on the next charge?

The user is responsible for keeping their payment sources updated.

If a payment fails the user will receive a push notification, informing them to
update their payment source.

Vipps does not automatically select a new card if a card expires, as users may
have multiple cards registered in Vipps.

Vipps also has standard functionality that automatically sends the user a push
notification when a card that is _not_ used for recurring payments expires.

### What happens to pending charges if the user deletes the payment card?

See [If a user's card expires: What happens on the next charge?](#if-a-users-card-expires-what-happens-on-the-next-charge)

### How does a user see the charges I create?

The charge will then appear in the app after it goes into the `DUE`-state.
A charge will remain in `PENDING` state until the dueDate is less than 30 days away.

You can retrieve all relevant charges through the [`GET:/agreements/{agreementId}/charges`][list-charges-endpoint] endpoint.

### If a user adds or updates a card in vipps, will new recurring charges be made to that card?
No, currently the payment card tied to an agreement will not be updated automatically.

Users may want to charge different Agreements to different cards, and we do
not want to automatically make changes to payment sources. Instead, we notify
users as described in
[What happens to pending charges if the user deletes the payment card?](#what-happens-to-pending-charges-if-the-user-deletes-the-payment-card)

### For how long is a payment reserved?

See [For how long is a payment reserved?](https://vippsas.github.io/vipps-developer-docs/docs/vipps-developers/faqs/reserve-and-capture-faq#for-how-long-is-a-payment-reserved)
in Vipps FAQs.


## Variable amount

### Can I decide the users suggested max amount list

No.
The suggested amount list that the user can choose a max amount from,
is automatically generated by Vipps based on the suggestedMaxAmount sent in.

If the suggestedMaxAmount is changed, the suggested amount list will also be different if the user goes to change max amount.

## Agreements and users

### How do I check my customer's status?

Get details about a specific Agreement:
[`fetch agreement endpoint`][fetch-agreement-endpoint].

### Can I look up a user's information?

Vipps can only provide user information with the user's consent.
The merchant must ask the user for consent when creating the agreement using
[Userinfo](vipps-recurring-api.md#userinfo)
with the correct `scope`. The requested information is only available for a limited amount of time after the consent is given.

### Can I look up which user owns an agreement?

No. See: [Can I look up a user's information?](#can-i-look-up-a-users-information) and [Is there an API for retrieving information about a Vipps user?](#is-there-an-api-for-retrieving-information-about-a-vipps-user).

### How can I convert existing agreements to Vipps agreements?

If you have existing agreements or subscriptions that use eFaktura, AvtaleGiro,
PDF invoices by email or other payment methods, you can let your customers
change to Vipps in different ways:

* Email: Email your customers and inform them that they can change
  to Vipps. The email should contain a unique URL that leads directly to a
  webpage with a Vipps payment for that customer. The customer may have to
  log in
  ([Vipps Logg Inn](https://www.vipps.no/produkter-og-tjenester/bedrift/logg-inn-med-vipps/logg-inn-med-vipps/)
  may be useful for this).
* PDF invoice: Include a unique URL for the customer, same as above. You can
  use both a URL and a QR code containing the same URL. Keep in mind that some
  users will see the PDF invoice on their phone, and can not easily scan the QR.
* Paper invoice: Include a QR code with unique URL for the customer, same as
  above.

### How can I delete an agreement?

Agreements can be stopped by calling the [`PATCH:/agreements/{agreementId}`][update-agreement-endpoint] endpoint.
See [Stop an agreement](vipps-recurring-api.md#stop-an-agreement).

### Can a user cancel the agreement through the Vipps app?
No, the user needs to contact the merchant which can then cancel or modify the
agreement as they see fit.

The merchant must provide a link to the "my page" on the merchant's website when creating the agreement.
See [Create an agreement](vipps-recurring-api.md#create-an-agreement).

Users may want to make other changes to the Agreement than to simply cancel it.
A subscription may be paused for a period, it may be changed to a lower
or higher frequency, additional products or services may be added, etc.
This can add more complexity than Vipps can present in a good way, and our
approach is therefore to send the user to the merchant for managing the
Agreement.

### How can I move agreements between merchants and sale units?

**Please note:** We are working out the details for this process, please contact
us if you need help with this.

Merchants sometimes need to move customer agreements from one merchant
to another, or from one sale unit to another.

Clarification of terms:
* Merchant: A juridical unit, typically called a business or company, identified with organization number ("orgno").
* Sale unit: A merchant can have one or more sale units. It may be different
  brands, different physical locations, different services, etc.
* MSN: The unique id of a sale unit.
  MSN is short for "Merchant Serial Number", but (sadly) identifies a sale unit, not a merchant.

First of all: A merchant very rarely "changes organization number".
That only occurs in special cases.
An organization number uniquely identifies a juridical unit and cannot usually be changed.

Typical cases:

1. If the merchant wants to move agreements from
   one MSN to another,
   and both MSNs are under the same orgno,
   Vipps may be able to help.
2. If the merchant wants to move agreements from
   one orgno to another,
   and both orgnos are owned by the same parent orgno,
   Vipps may be able to help.
3. If the merchant wants to move agreements from
   one orgno to another,
   and the orgnos are not owned by a parent company,
   Vipps may be able to help.
   The users have entered agreements with an orgno, and Vipps can not
   automatically move the agreements without the user's consent to enter
   agreements with the new orgno.
   The users may need to enter new agreements with the new orgno.
4. If the merchant wants to move agreements between two independent orgnos
   (not owned by a parent orgno),
   splits or fissions out a new orgno,
   is merged with another orgno,
   is acquired by another orgno,
   etc:
   Vipps may *_not_* be able to help.
   The users have entered agreements with an orgno, and Vipps may not be able to
   automatically move the agreements without the user's consent to enter
   agreements with the new orgno.
   The users may need to enter new agreements with the new orgno.

The general process is:
- Contact Vipps (your KAM or partner manager) to plan the switch, and
  the date and time for moving the agreements.
- Log in on
  [portal.vipps.no](https://portal.vipps.no)
  and enter a new agreement with Vipps for the new orgno.
- Order "Vipps Faste betalinger" for the new orgno. on
  [portal.vipps.no](https://portal.vipps.no).
- Inform all existing customers of the new orgno,
  and the planned date of the change.
- Send a confirmation to avtale@vipps.no that all customers have been informed,
  including a copy of the information sent.

Agreements are not actually moved. Your new saleunit will get new agreements
identical to the old ones, but with new agreementIds. The old agreements will
be stopped. This is done to prevent any confusion around charges, refunds, etc.
between the two MSNs. Charges done by the old MSN are in the old agreement,
while the new MSN uses only the new agreement

For all requests to move agreements: Please contact your KAM, your partner or
[Vipps customer service](https://vipps.no/kontakt-oss/).

**Please note:** If the merchant deactivates a MSN that has active
agreements, it will no longer be possible to:
- Manage the agreements
- Perform new charges
- Make refunds

## Common problems/errors

### Why do I get the error `merchant.not.allowed.for.recurring.operation`?

The `merchant.not.allowed.for.recurring.operation` error indicates
that the Vipps Recurring API is not yet activated for this sale unit.

The Vipps Recurring API is available for existing customers that
have "Vipps på Nett", a direct integration with the
[Vipps eCom API](https://vippsas.github.io/vipps-developer-docs/docs/APIs/ecom-api/),
and have completed some additional Know Your Customer (KYC) checks required by [Finanstilsynet](https://www.finanstilsynet.no).

Vipps is required to perform some extra compliance checks before
activating the Vipps Recurring API.

Please order "Vipps Faste betalinger" on
[portal.vipps.no](https://portal.vipps.no)
to get access to the Recurring API in production.

### Is there an API for retrieving information about a Vipps user?

Yes.

Vipps offers the possibility for merchants, as part of the payment flow, to request the user's information.
For example if you wish to retrieve the user's email as part of a User profile setup.
This is supported in both [Vipps eCom API](https://vippsas.github.io/vipps-developer-docs/docs/APIs/ecom-api/vipps-ecom-api#userinfo)
and [Vipps Recurring API](vipps-recurring-api.md#userinfo).

This is done by adding a `scope` parameter to the initiate calls:
[`initiate payment`](https://vippsas.github.io/vipps-developer-docs/api/ecom#tag/Vipps-eCom-API/operation/initiatePaymentV3UsingPOST) (eCom)
and
[`POST:/agreements`][draft-agreement-endpoint] (Recurring):

- address
- birthDate
- email
- name
- phoneNumber
- nin (fødselsnummer)
- accountNumbers

The userinfo endpoint is shared with
[Vipps Login](https://vippsas.github.io/vipps-developer-docs/docs/APIs/login-api/)
and the merchant needs to have activated Vipps Login on their account to use
this feature. Information on how to do this can be found
[here](https://vippsas.github.io/vipps-developer-docs/docs/APIs/login-api/vipps-login-api-faq#how-can-i-activate-and-set-up-vipps-login).
Using the Vipps Login service itself is optional.

**Please note:** Vipps users have not consented to Vipps providing any
information to third parties, and Vipps does not allow it. There is no
other API to look up a user's address, retrieve a user's purchases, etc.

## Notifications and error messages

### When do users get push messages?

| Event                                           | Push message text (Norwegian)                                                       | Push message text (English)                                                             |
|-------------------------------------------------|-------------------------------------------------------------------------------------|-----------------------------------------------------------------------------------------|
| Agreement request                               | Bekreft fast betaling                                                               | Confirm recurring payment                                                               |
| Charge processed successfully                   | Fast betaling til `sales unit name` har blitt gjennomført                           | Payment to `sales unit name` has been processed                                         |
| Insufficient funds                              | Pass på at du har nok penger. Vi prøver igjen litt senere.                          | Make sure you have enough money. We'll try again later.                                 |
| Invalid payment source (retryable)              | Kortet er ugyldig, prøv et annet.                                                   | The card is invalid, try another                                                        |
| Invalid payment source (not retryable)          | Kortet er ugyldig, prøv å endre det og kontakt %s å fortsette avtalen.              | The card is invalid, try changing it and contacting %s to continue the agreement.       |
| Charge amount too high (variable amount)        | Beløpet er høyere enn det avtalte maksimumsbeløpet.                                 | The amount is higher than the agreed maximum amount                                     |
| Future charge amount too high (variable amount) | Du må øke maksbeløpet i avtalen med `sales unit name`                               | You have to change the maximum amount in the agreement. If not, the payment will fail.  |

### What is shown to users when charge processing fails?

We set the failure reason on the charge based on why the processing failed.

| Reason                                           | Charge failure text (Norwegian)                                                                   | Charge failure text (English)                                                            |
|--------------------------------------------------|---------------------------------------------------------------------------------------------------|------------------------------------------------------------------------------------------|
| Insufficient funds                               | Betalingen feilet. Pass på at du har nok penger på konto eller bytt kort. Vi prøver igjen senere. | Payment failed. Make sure you have enough money or change card. We'll try again later.   |
| Invalid payment source                           | Kortet som er knyttet til avtalen din er ugyldig. Bytt til et annet.                              | The card associated with your agreement is invalid. Switch to another.                   |
| Charge amount too high (variable amount)         | Betalingen feilet. Endre maksbeløpet i avtalen under. Vi prøver igjen senere.                     | Payment failed. Change the maximum amount in the agreement below. We'll try again later. |
| Future charge amount too high (variable amount)  | Betalingen kommer til å feile. Endre maksbeløpet i avtalen under.                                 | Payment will fail. Change the maximum amount in the agreement below.                     |

## Admin/partners

### How can I change partners for a merchant?

**Please note:** We are working out the details for this process, please contact
us if you need help with this.

Here is the procedure for what you must do when changing partners:
We need consent from the user site that they will change partners.
They can submit this directly to Vipps (via partnerbestelling@vipps.no) or
they can submit it through you as a new partner.

The partner orders at Vipps (partnerbestelling@vipps.no) must receive an e-mail
from the user site or partner stating that they wish to change partner with the
following information:
* Name of user location
* Organization number
* Name of new partner
* Name of old partner
* Which MSN it should be "switched to"
* Test page if the website changes
* Date and time of desired transfer from old partner to new partner
* Signature from customer
* Possible new price

One thing you should be aware of is that in each agreement there is a `merchantAgreementUrl`
which is the link each user clicks on to be able to change their subscription.
In other words, a "My page" for the user. If the link structure is not the same
in both solutions, you must update all existing agreements with a new URL as
soon as possible after the move so that the customers can manage the agreements
further without coming to a blank page.

### How can I change partner for my integration with Vipps?

See [How to change partners for a merchant](https://vippsas.github.io/vipps-developer-docs/docs/vipps-partner/#how-to-change-partners-for-a-merchant).

**Please note:** The MSN (the number) does _not_ change when changing partners.

### Settlement
The settlements are done trough Vipps.
The merchant does not need any other partner or agreement.

### Invoicing
Merchants with a "net settlement" contract receive the users' payments excluding the Vipps fees.
Merchants with a "gross settlement" contract receive the users' payments including the Vipps fees,
and are then invoiced for the Vipps fees.

## Questions?

We're always happy to help with code or other questions you might have!
Please create an [issue](https://github.com/vippsas/vipps-recurring-api/issues),
a [pull request](https://github.com/vippsas/vipps-recurring-api/pulls),
or [contact us](https://vippsas.github.io/vipps-developer-docs/docs/vipps-developers/contact).

Sign up for our [Technical newsletter for developers](https://vippsas.github.io/vipps-developer-docs/docs/vipps-developers/newsletters).

[draft-agreement-endpoint]: https://vippsas.github.io/vipps-developer-docs/api/recurring#tag/Agreement-v2-endpoints/operation/DraftAgreement
[fetch-agreement-endpoint]: https://vippsas.github.io/vipps-developer-docs/api/recurring#tag/Agreement-v2-endpoints/operation/FetchAgreement
[update-agreement-endpoint]: https://vippsas.github.io/vipps-developer-docs/api/recurring#tag/Agreement-v2-endpoints/operation/UpdateAgreementPatch
[list-charges-endpoint]: https://vippsas.github.io/vipps-developer-docs/api/recurring#tag/Charge-v2-endpoints/operation/ListCharges
