# Vipps Recurring API: Frequently Asked Questions

See the
[Vipps Recurring API](https://github.com/vippsas/vipps-recurring-api/blob/master/vipps-recurring-api.md)
for all the details.

See also:
[Vipps eCom API FAQ](https://github.com/vippsas/vipps-ecom-api/blob/master/vipps-ecom-api-faq.md).

See also:
[Getting Started](https://github.com/vippsas/vipps-developers/blob/master/vipps-getting-started.md)
guide.

Document version: 1.5.3.

## Table of Contents

* [Does Vipps automatically create charges for an agreement?](#does-vipps-automatically-create-charges-for-an-agreement)
* [Do I need to store card data?](#do-i-need-to-store-card-data)
* [Why do I get the error `merchant.not.allowed.for.recurring.operation`?](#why-do-i-get-the-error-merchantnotallowedforrecurringoperation)
* [Can I look up a user's information?](#can-i-look-up-a-users-information)
* [How can I convert existing agreements to Vipps agreements?](#how-can-i-convert-existing-agreements-to-vipps-agreements)
* [At what time during the day are charges made?](#at-what-time-during-the-day-are-charges-made)
* [How do I check my customer's status?](#how-do-i-check-my-customers-status)
* [A customer's charge failed, but I did not receive any warning](#a-customers-charge-failed-but-i-did-not-receive-any-warning)
* [I don't want a charge to fail the first time the transaction fails (insufficient funds / networking issues etc.)](#i-dont-want-a-charge-to-fail-the-first-time-the-transaction-fails-insufficient-funds--networking-issues-etc)
* [Can the charge for an Agreement be changed?](#can-the-charge-for-an-agreement-be-changed)
* [Are there any limits on charging a user?](#are-there-any-limits-on-charging-a-user)
* [When can I send charges for a user?](#when-can-i-send-charges-for-a-user)
* [How can I delete an agreement?](#how-can-i-delete-an-agreement)
* [Can a user cancel the agreement through the Vipps app?](#can-a-user-cancel-the-agreement-through-the-vipps-app)
* [What happens to charges if the corresponding agreement is cancelled?](#what-happens-to-charges-if-the-corresponding-agreement-is-cancelled)
* [If a user's card expires: What happens on the next charge?](#if-a-users-card-expires-what-happens-on-the-next-charge)
* [What happens to pending charges if the user deletes the payment card?](#what-happens-to-pending-charges-if-the-user-deletes-the-payment-card)
* [How does a user see any charges I send?](#how-does-a-user-see-any-charges-i-send)
* [If a user changes the default payment card in Vipps, can new charges be made to that card?](#if-a-user-changes-the-default-payment-card-in-vipps-can-new-charges-be-made-to-that-card)
* [How can I change partner for my integration with Vipps?](#how-can-i-change-partner-for-my-integration-with-vipps)
* [Is there an API for retrieving information about a Vipps user?](#is-there-an-api-for-retrieving-information-about-a-vipps-user)
* [Does the recent (24.09.2020) changes in terms regarding valid reservations affect my agreements?](#does-the-recent-24092020-changes-in-terms-regarding-valid-reservations-affect-my-agreements)
* [Settlement](#settlement)
* [Invoicing](#invoicing)
* [Questions?](#questions)

## Does Vipps automatically create charges for an agreement?

 No. Vipps does _not_ create charges based on the agreement, this is left up to the merchant to create.  
 When a merchant creates a charge, Vipps will actually attempt to charge the customer, starting on the `due date` and for as long as specified in `retryDays`.

 For more details, see our [call-by-call guide](https://github.com/vippsas/vipps-recurring-api/blob/master/vipps-recurring-api.md#call-by-call-guide)

 ## Do I need to store card data?

 No. Vipps handles all payment details.

## Why do I get the error `merchant.not.allowed.for.recurring.operation`?

The `merchant.not.allowed.for.recurring.operation` error indicates
that the Vipps Recurring API is not yet activated for this sale unit.

The Vipps Recurring API is available for existing customers that
have "Vipps på Nett" and a direct integration with the
[Vipps eCom API](https://github.com/vippsas/vipps-recurring-api)
and have completed some additional KYC checks required by Finanstilsynet.

Vipps is required to perform some extra compliance checks before
activating the Vipps Recurring API.

Please order "Vipps Faste betalinger" on
[portal.vipps.no](https://portal.vipps.no)
to get access to the Recurring API in production.

## Can I look up a user's information?

Vipps can only provide user information with the user's consent.
The merchant must ask the user for consent when creating the agreement using
[Userinfo](https://github.com/vippsas/vipps-recurring-api/blob/master/vipps-recurring-api.md#userinfo)
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

## At what time during the day are charges made?
Charge _attempts_ are made two times during the day: 08:00 og 16:00 UTC.
Subsequent attempts are made according to the `retryDays` specified.  
This applies for both our production and test environment (MT).

## How do I check my customer's status?
Get all Agreements for a customer:
[`GET:/recurring/v2/agreements`](https://vippsas.github.io/vipps-recurring-api/#/Agreement%20Endpoints/draftAgreement)

Get details about a specific Agreement:
[`GET:/recurring/v2/agreements/{agreementId}`](https://vippsas.github.io/vipps-recurring-api/#/Agreement%20Endpoints/getAgreement).

## A customer's charge failed, but I did not receive any warning
The customer may not have notifications turned on,
or they have not upgraded to the Vipps version that supports it.

## I don't want a charge to fail the first time the transaction fails (insufficient funds / networking issues etc.)
The field `retryDays` in an Agreement allows for this functionality, Vipps will
retry once each day until the value is reached. The valid values are none
(defaulting to 0) or 0-14. We strongly recommend having more than 1 retry day
to account for possible networking issues etc.

## Can the charge for an Agreement be changed?
Yes. See
[Are there any limits on charging a user?](#are-there-any-limits-on-charging-a-user)

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

See [How do I check my customer's status?](#how-do-i-check-my-customers-status)

## How can I delete an agreement?

Agreements are marked as deleted by
[updating the Agreement](https://github.com/vippsas/vipps-recurring-api/blob/master/vipps-recurring-api.md#updating-an-agreement)
to `STOPPED` or `EXPIRED`.

## Can a user cancel the agreement through the Vipps app?
No, the user needs to contact the merchant which can then cancel or modify the
agreement as they see fit.

There is a link in Vipps to "my page" on the merchant's website, where the
user can manage the agreement.

Users may want to make other changes to the Agreement than to simply cancel it.
A subscription may be paused for a period, it may be changed to a lower
or higher frequency, additional products or services may be added, etc.
This can add more complexity than Vipps can present in a good way, and our
approach is therefore to send the user to the merchant for managing the
Agreement.

## What happens to charges if the corresponding agreement is cancelled?
All charges in a `PENDING`, `DUE` or `RESERVED` state will be cancelled if the Agreement is stopped.  
**Note**: This also includes the `initial charge` if it's currently `RESERVED`. So if the merchant needs to charge the user for the initial charge; then this needs to be done before the agreement is stopped.

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

You can still retrieve all relevant charges through the API:
[`GET:/recurring/v2/agreements/{agreementId}/charges`](https://vippsas.github.io/vipps-recurring-api/#/Charge%20Endpoints/listCharges).

## If a user changes the default payment card in Vipps, can new charges be made to that card?
No, currently the payment card tied to an agreement will not be updated automatically.

Users may want to charge different Agreements to different cards, and we do
not want to automatically make changes to payment sources. Instead, we notify
users as described in
[What happens to pending charges if the user deletes the payment card?](#what-happens-to-pending-charges-if-the-user-deletes-the-payment-card)

## How can I change partner for my integration with Vipps?

See:
[How to change partners for a merchant](https://github.com/vippsas/vipps-partner#how-to-change-partners-for-a-merchant).

**Please note:** The MSN (the number) does _not_ change when changing partners.

## Is there an API for retrieving information about a Vipps user?

Yes.

Vipps offers the possibility for merchants to as part of the payment flow to request the user's information. For example if you wish to retrieve the user's email as part of a User profile setup. This is supported in both [Vipps eCom API v2](https://github.com/vippsas/vipps-ecom-api/blob/master/vipps-ecom-api.md#userinfo)
and
[Vipps Recurring API](https://github.com/vippsas/vipps-recurring-api/blob/master/vipps-recurring-api.md#userinfo).

This is done by adding a `scope` parameter to the initiate calls:
[`POST:/ecomm/v2/payments`](https://vippsas.github.io/vipps-ecom-api/#/Vipps%20eCom%20API/initiatePaymentV3UsingPOST) (eCom)
and
[`POST:/recurring/v2/agreements`](https://vippsas.github.io/vipps-recurring-api/#/Agreement%20Controller/draftAgreement) (Recurring):

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

## Does the recent (24.09.2020) changes in terms regarding valid reservations affect my agreements?

If you are using reserve-capture on your initial charge, then capture will only be possible within the first 30 days.

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
or [contact us](https://github.com/vippsas/vipps-developers/blob/master/contact.md).

Sign up for our [Technical newsletter for developers](https://github.com/vippsas/vipps-developers/tree/master/newsletters).
