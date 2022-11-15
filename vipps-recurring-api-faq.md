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
[Vipps API FAQ](https://github.com/vippsas/vipps-developers/tree/master/faqs).

See also:
[Getting Started](https://github.com/vippsas/vipps-developers/blob/master/vipps-getting-started.md)
guide.

Document version: 1.6.2.

<!-- START_TOC -->

## Table of Contents

* [Does Vipps automatically create charges for an agreement?](#does-vipps-automatically-create-charges-for-an-agreement)
* [Do I need to store card data?](#do-i-need-to-store-card-data)
* [Why do I get the error `merchant.not.allowed.for.recurring.operation`?](#why-do-i-get-the-error-merchantnotallowedforrecurringoperation)
* [Can I look up a user's information?](#can-i-look-up-a-users-information)
* [How can I convert existing agreements to Vipps agreements?](#how-can-i-convert-existing-agreements-to-vipps-agreements)
* [How can I move agreements between merchants and sale units?](#how-can-i-move-agreements-between-merchants-and-sale-units)
* [How can I change partners for a merchant?](#how-can-i-change-partners-for-a-merchant)
* [At what time during the day are charges made?](#at-what-time-during-the-day-are-charges-made)
* [How do I check my customer's status?](#how-do-i-check-my-customers-status)
* [A customer's charge failed, but I did not receive any warning](#a-customers-charge-failed-but-did-not-receive-any-warning)
* [I don't want a charge to fail the first time the transaction fails (insufficient funds / networking issues etc.)](#i-dont-want-a-charge-to-fail-the-first-time-the-transaction-fails-insufficient-funds--networking-issues-etc)
* [Can the charge be different from the agreement's price?](#can-the-charge-be-different-from-the-agreements-price)
* [Are there any limits on charging a user?](#are-there-any-limits-on-charging-a-user)
* [When can I send charges for a user?](#when-can-i-send-charges-for-a-user)
* [How can I delete an agreement?](#how-can-i-delete-an-agreement)
* [Can a user cancel the agreement through the Vipps app?](#can-a-user-cancel-the-agreement-through-the-vipps-app)
* [What happens to charges if the corresponding agreement is cancelled?](#what-happens-to-charges-if-the-corresponding-agreement-is-cancelled)
* [If a user's card expires: What happens on the next charge?](#if-a-users-card-expires-what-happens-on-the-next-charge)
* [What happens to pending charges if the user deletes the payment card?](#what-happens-to-pending-charges-if-the-user-deletes-the-payment-card)
* [How does a user see any charges I send?](#how-does-a-user-see-any-charges-i-send)
* [If a user adds or updates a card in vipps, will new recurring charges be made to that card?](#if-a-user-adds-or-updates-a-card-in-vipps-will-new-recurring-charges-be-made-to-that-card)
* [How can I change partner for my integration with Vipps?](#how-can-i-change-partner-for-my-integration-with-vipps)
* [Is there an API for retrieving information about a Vipps user?](#is-there-an-api-for-retrieving-information-about-a-vipps-user)
* [For how long is a payment reserved?](#for-how-long-is-a-payment-reserved)
* [When do users get push messages?](#when-do-users-get-push-messages)
* [Settlement](#settlement)
* [Invoicing](#invoicing)
* [Questions?](#questions)

<!-- END_TOC -->

## Does Vipps automatically create charges for an agreement?

 No. Vipps does _not_ create charges based on the agreement, this is left up to the merchant to create.  
 When a merchant creates a charge, Vipps will actually attempt to charge the customer, starting on the `due date` and for as long as specified in `retryDays`.

 For more details, see our [call-by-call guide](vipps-recurring-api.md#call-by-call-guide)

## Do I need to store card data?

No. Vipps handles all payment details.

## Why do I get the error `merchant.not.allowed.for.recurring.operation`?

The `merchant.not.allowed.for.recurring.operation` error indicates
that the Vipps Recurring API is not yet activated for this sale unit.

The Vipps Recurring API is available for existing customers that
have "Vipps på Nett", a direct integration with the
[Vipps eCom API](https://github.com/vippsas/vipps-ecom-api),
and have completed some additional Know Your Customer (KYC) checks required by [Finanstilsynet](https://www.finanstilsynet.no).

Vipps is required to perform some extra compliance checks before
activating the Vipps Recurring API.

Please order "Vipps Faste betalinger" on
[portal.vipps.no](https://portal.vipps.no)
to get access to the Recurring API in production.

## Can I look up a user's information?

Vipps can only provide user information with the user's consent.
The merchant must ask the user for consent when creating the agreement using
[Userinfo](vipps-recurring-api.md#userinfo)
with the correct `scope`.

## How can I convert existing agreements to Vipps agreements?

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

## How can I move agreements between merchants and sale units?

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

## How can I change partners for a merchant?

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

## At what time during the day are charges made?
Charge _attempts_ are made two times during the day: 08:00 og 16:00 UTC.
Subsequent attempts are made according to the `retryDays` specified.
This applies for both our production and test environment (MT).

## How do I check my customer's status?

Get details about a specific Agreement:
[`fetch agreement endpoint`][fetch-agreement-endpoint].

## A customer's charge failed, but did not receive any warning
The customer may not have notifications turned on,
or they have not upgraded to the Vipps version that supports it.

## I don't want a charge to fail the first time the transaction fails (insufficient funds / networking issues etc.)
The field `retryDays` in an Agreement allows for this functionality, Vipps will
retry once each day until the value is reached. The valid values are none
(defaulting to 0) or 0-14. We strongly recommend having more than 1 retry day
to account for possible networking issues etc.

## Can the charge be different from the agreement's price?
Yes. See [Are there any limits on charging a user?](#are-there-any-limits-on-charging-a-user).

## Are there any limits on charging a user?
Yes, within the `interval` period of the Agreement the merchant can charge at
most **5 times** the Agreement price cumulatively. There is no limit on the
number of charges which can be sent in the `interval` period.

Vipps recommends creating a new Agreement if there is a significant price change.
It is the merchant's responsibility to make sure the user is informed and understands
the price of the Agreement.

## When can I send charges for a user?
You can send charges once you have polled and found a valid Agreement tied to
the user.

See [How do I check my customer's status?](#how-do-i-check-my-customers-status).

## How can I delete an agreement? 

Agreements can be stopped by calling the [`PATCH:/agreements/{agreementId}`][update-agreement-endpoint] endpoint. 
See [Stop an agreement](vipps-recurring-api.md#stop-an-agreement).

## Can a user cancel the agreement through the Vipps app?
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

## What happens to charges if the corresponding agreement is cancelled?
All charges in a `PENDING`, `DUE` or `RESERVED` state will be cancelled if the Agreement is stopped.  
**Note**: This also includes the `initial charge` if it's currently `RESERVED`. 
So if the merchant needs to charge the user for the initial charge; then this needs to be done before the agreement is stopped.

## If a user's card expires: What happens on the next charge?

The user is responsible for keeping their payment sources update.

If a payment fails the user will receive a push notification, informing them to
update their payment source.

Vipps does not automatically select a new card if a card expires, as users may
have multiple cards registered in Vipps.

Vipps also has standard functionality that automatically sends the user a push
notification when a card that is _not_ used for recurring payments expires.

## What happens to pending charges if the user deletes the payment card?

See [If a user's card expires: What happens on the next charge?](#if-a-users-card-expires-what-happens-on-the-next-charge)

## How does a user see any charges I send?

A charge will be displayed to the user 6 days before the charge is due to be processed.
The charge will then appear in the app.

You can still retrieve all relevant charges through the [`GET:/agreements/{agreementId}/charges`][list-charges-endpoint] endpoint.

## If a user adds or updates a card in vipps, will new recurring charges be made to that card?
No, currently the payment card tied to an agreement will not be updated automatically.

Users may want to charge different Agreements to different cards, and we do
not want to automatically make changes to payment sources. Instead, we notify
users as described in
[What happens to pending charges if the user deletes the payment card?](#what-happens-to-pending-charges-if-the-user-deletes-the-payment-card)

## How can I change partner for my integration with Vipps?

See [How to change partners for a merchant](https://github.com/vippsas/vipps-partner#how-to-change-partners-for-a-merchant).

**Please note:** The MSN (the number) does _not_ change when changing partners.

## Is there an API for retrieving information about a Vipps user?

Yes.

Vipps offers the possibility for merchants, as part of the payment flow, to request the user's information. 
For example if you wish to retrieve the user's email as part of a User profile setup. 
This is supported in both [Vipps eCom API](https://github.com/vippsas/vipps-ecom-api/blob/master/vipps-ecom-api.md#userinfo)
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
[Vipps Login](https://github.com/vippsas/vipps-login-api)
and the merchant needs to have activated Vipps Login on their account to use
this feature. Information on how to do this can be found
[here](https://github.com/vippsas/vipps-login-api/blob/master/vipps-login-api-faq.md#how-can-i-activate-and-set-up-vipps-login).
Using the Vipps Login service itself is optional.

**Please note:** Vipps users have not consented to Vipps providing any
information to third parties, and Vipps does not allow it. There is no
other API to look up a user's address, retrieve a user's purchases, etc.

## For how long is a payment reserved?

See [For how long is a payment reserved?](https://github.com/vippsas/vipps-developers/blob/master/faqs/reserve-and-capture-faq.md#for-how-long-is-a-payment-reserved)
in Vipps FAQs.

## When do users get push messages?

| Event                                           | Push message text (Norwegian)                                                       | Push message text (English)                                                                    |
|-------------------------------------------------|-------------------------------------------------------------------------------------|------------------------------------------------------------------------------------------------|
| Agreement request                               | Bekreft fast betaling                                                               | Confirm recurring payment                                                                      |
| Charge processed successfully                   | Fast betaling til `merchant name` har blitt gjennomført                             | Payment to `merchant name` has been processed                                                  |
| Insufficient funds                              | Pass på at du har nok penger. Vi prøver igjen litt senere.                          | Make sure you have enough money. We'll try again later.                                        |
| Invalid payment source (retryable)              | Kortet er ugyldig, prøv å endre det og kontakt `merchant name` å fortsette avtalen. | The card is invalid, try changing it and contacting `merchant name` to continue the agreement. |
| Invalid payment source (not retryable)          | Kortet er ugyldig, prøv et annet.                                                   | The card is invalid, try another.                                                              |
| Charge amount too high (variable amount)        | Beløpet er høyere enn det avtalte maksimumsbeløpet.                                 | The amount is higher than the agreed maximum amount.                                           |
| Future charge amount too high (variable amount) | Du må øke maksbeløpet i avtalen med `merchant name`                                 | You have to change the maximum amount in the agreement. If not, the payment will fail.         |


## Settlement
The settlements are done trough Vipps.
The merchant does not need any other partner or agreement.

## Invoicing
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

