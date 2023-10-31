<!-- START_METADATA
---
title: Recurring API frequently asked questions
sidebar_label: FAQ
sidebar_position: 45
description: Recurring API frequently asked questions
pagination_next: null
pagination_prev: null
---
END_METADATA -->

# Frequently asked questions

See the
[Recurring API](recurring-api-guide.md)
for all the details.

For general information and questions, please check in the
[Knowledge base](https://developer.vippsmobilepay.com/docs/knowledge-base/).

<!-- START_COMMENT -->
ℹ️ Please use the website:
[Vipps MobilePay Technical Documentation](https://developer.vippsmobilepay.com/docs/APIs/recurring-api).
<!-- END_COMMENT -->

## Migration

### How do I migrate from the Recurring API v2 to v3?

See the [migration guide](v2-to-v3-migration-guide.md) for the differences between the Recurring API v2 and v3.
Also, check the [V3 API definitions](https://developer.vippsmobilepay.com/api/recurring).

### How do I migrate from the MobilePay subscription solution?

See:
[Migration guide](https://developer.vippsmobilepay.com/docs/mp-migration-guide#subscriptions-vs-recurring).

### Can I manage agreements and charges created with v2 API using v3 API?

Yes. All agreements and charges created with the v2 API can be retrieved and managed using the v3 API and vice-versa.
Also, if an agreement was created with v2 API, it is possible to create a charge for this agreement with v3 API and vice-versa.

## Charges/Payments

### Do I need to store card data?

No. We handle all payment details. Merchants don't need to know how the users pay.

### Do you automatically create charges for an agreement?

No. We do *not* create charges based on the agreement, every charge must be created by the merchant.

When a merchant creates a charge, we attempt to charge the customer, starting on the `due`
date, and for as long after that as specified in `retryDays`.

For more details, see our [call-by-call guide](recurring-api-guide.md#call-by-call-guide)

### When are charges processed?

Charges are processed from the `due` date, and for
the specified number of `retryDays` after the `due` date.

Charges are processed two times every day: 07:00 and 15:00 UTC
(this may change without notice).

Retries are attempted according to the number of `retryDays` specified.
This applies to both our production and
[test environment](https://developer.vippsmobilepay.com/docs/test-environment/).

### I don't want a charge to fail the first time the transaction fails (insufficient funds, networking issues, etc.)

The field `retryDays` in an agreement allows for this functionality. We will
retry once each day until the value is reached. The valid values are none
(defaulting to 0) or 0-14. We strongly recommend having more than 1 retry day
to account for possible networking issues etc.

### How can I require an initial payment from the user for setting up the agreement?

You need to use
[Initial charge](https://developer.vippsmobilepay.com/docs/APIs/recurring-api/recurring-api-guide#initial-charge).

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

You can create charges once the user's agreement is `ACTIVE`.

See [How do I check my customer's status?](#how-do-i-check-my-customers-status).

### Can I change the charge interval?

No, it is not possible to change intervals. If the user has accepted a yearly interval,
the agreement cannot be changed to a monthly agreement.
This requires a new agreement and new consent from the user.

It *is* possible to make a monthly agreement and charge for some months only.
The general rule: Be as customer friendly and easy to understand as possible.

See:
[`PATCH:/recurring/v3/agreements/{agreementId}`](https://developer.vippsmobilepay.com/api/recurring/#tag/Agreement-v3-endpoints/operation/UpdateAgreementPatchV3).

### A charge failed, but the customer did not receive any warning

The customer may not have push notifications turned on for the app.

We always send push notifications to the user in the app when a charge processing attempt is not
successful, and the user gets a more detailed message when looking at the
charge/agreement in the app.

Merchants should always ask the user to check the status of charges in the app.
We are not able to manually check charges unless there is a clear indication that
there is an error on our end.

### What happens to charges if the corresponding agreement is canceled?

All charges in a `PENDING`, `DUE` or `RESERVED` state will be canceled if the Agreement is stopped.  
**Note**: This also includes the `initial charge` if it's currently `RESERVED`.
So if the merchant needs to charge the user for the initial charge; then this needs to be done before the agreement is stopped.

### If a user's card expires: What happens on the next charge?

The user is responsible for keeping their payment cards updated.

If a payment fails the user will receive a push notification, informing them to
update their card.

We don't automatically select a new card if a card expires, as users may
have multiple cards registered in the app. There are also regulatory requirements:
The user must actively choose a card.

We automatically send the user a push notification when a card that
is *not* used for recurring payments expires.

### What happens to pending charges if the user deletes the payment card?

See [If a user's card expires: What happens on the next charge?](#if-a-users-card-expires-what-happens-on-the-next-charge)

### How does a user see the charges I create?

The charge will be visible to the user in the app when it goes into the `DUE` state.
A charge will remain in `PENDING` state until the `due` date is less than 30 days away.

You can retrieve all relevant charges through the
[`GET:/recurring/v3/agreements/{agreementId}/charges`][list-charges-endpoint] endpoint.

### If a user adds or updates a card in the Vipps or MobilePay app, will new recurring charges be made to that card?

No, currently the payment card tied to an agreement will not be updated automatically.

Users may want to charge different Agreements to different cards, and we do
not want to automatically make changes to payment sources. Instead, we notify
users as described in
[What happens to pending charges if the user deletes the payment card?](#what-happens-to-pending-charges-if-the-user-deletes-the-payment-card)

### For how long is a payment reserved?

See [For how long is a payment reserved?](https://developer.vippsmobilepay.com/docs/knowledge-base/reserve-and-capture/#for-how-long-is-a-payment-reserved).

### How can I make refunds?

Refunds must always be done using the API, through the merchant's administration solution.

From the [API checklist](https://developer.vippsmobilepay.com/docs/APIs/recurring-api/recurring-api-checklist/):

> Make sure your customer service, etc. have all the tools and information they need available in your system, through the APIs listed in the first item in this checklist, and that they do not need to visit portal.vipps.no for normal work.

**Important:** If you do manual refunds on
[portal.vipps.no](https://portal.vipps.no),
the API may return incorrect details for charges until the same refund operation is
done using the API.

## Variable amount

### Can I decide the user's suggested max amount list

No. The suggested amount list from which the user can choose a maximum amount
is automatically generated based on the `suggestedMaxAmount` sent in.

If the `suggestedMaxAmount` is changed, the suggested amount list will also be different
the next time the user goes to change the maximum amount.

## Agreements and users

### Why do you allow drafting multiple agreements for the same user?

There are cases where
one merchant might want to draft multiple subscriptions at the same time for the same user
(i.e., the user subscribes to multiple services from the merchant), and there are also cases
where one person might pay for multiple subscriptions for other reasons (family/relationships/guardianship).

It is the merchant's responsibility to know which of their users each drafted agreement belongs to,
and to prevent drafting multiple agreements if that is not desirable.

### How do I prevent drafting multiple agreements for the same user?

In order to prevent drafting multiple agreements for the same user, you need to keep track of
which of your users any drafted agreement belongs to. Then, when a user does something that
triggers a draft, you should first check if they already have an agreement that is `PENDING`,
and inform the user that they should finish the agreement activation in the app or the landing page.

There might be some cases where the process fails in such a way that the user cannot complete
activation and the agreement is stuck as `PENDING` for an extended period of time. Because of
this, it might be a good idea to allow the user to choose to draft a new agreement regardless,
but then you need to keep track of that "abandoned" agreement in case you might need to manage
it later (stop and issue a refund if it gets undesirably activated or similar).

### How do I check my customer's status?

Check the status of the user's agreement:
[Retrieve an agreement](https://developer.vippsmobilepay.com/docs/APIs/recurring-api/recurring-api-guide/#retrieve-an-agreement).

### Can I look up a user's information?

We can  provide a user's information with the user's consent.
The merchant must ask the user for consent when creating the agreement using
[Userinfo](recurring-api-guide.md#userinfo)
with the correct
[`scope`](https://developer.vippsmobilepay.com/docs/APIs/userinfo-api/#scope).
The requested information is only available for a
limited amount of time after the consent is given.

### Can I look up which user owns an agreement?

No. See:
[Can I look up a user's information?](#can-i-look-up-a-users-information)
and
[Is there an API for retrieving information about a Vipps MobilePay user?](#is-there-an-api-for-retrieving-information-about-a-vipps-mobilepay-user).

### How can I convert existing agreements to Vipps MobilePay agreements?

If you have existing agreements or subscriptions that use eFaktura, AvtaleGiro,
PDF invoices by email or other payment methods, you can allow your customers to
change to Vipps MobilePay in different ways:

* Email: Email your customers and inform them that they can change
  to Vipps MobilePay. The email should contain a unique URL that leads directly to a
  webpage with a Vipps MobilePay payment for that customer. The customer may have to
  [log in](https://www.vipps.no/produkter-og-tjenester/bedrift/logg-inn-med-vipps/logg-inn-med-vipps/)
  may be useful for this.
* PDF invoice: Include a unique URL for the customer, same as above. You can
  use both a URL and a QR code containing the same URL. Keep in mind that some
  users will see the PDF invoice on their phone, and can not easily scan the QR.
* Paper invoice: Include a QR code with a unique URL for the customer, same as
  above.

### How can I delete an agreement?

Agreements can be stopped by calling the
[`PATCH:/recurring/v3/agreements/{agreementId}`][update-agreement-endpoint] endpoint.
See [Stop an agreement](recurring-api-guide.md#stop-an-agreement).

### Can a user cancel the agreement through the Vipps or MobilePay app?

No, the user needs to contact the merchant which can then cancel or modify the
agreement as they see fit.

The merchant must provide a link to the "my page" on the merchant's website when creating the agreement.
See [Create an agreement](recurring-api-guide.md#create-an-agreement).

Users may want to make other changes to the Agreement than simply cancel it.
A subscription may be paused for a period, it may be changed to a lower
or higher frequency, additional products or services may be added, etc.
This can add more complexity than we can present in a good way, and our
approach is, therefore, to send the user to the merchant for managing the
Agreement.

### How can I move agreements between merchants and sales units?

**Please note:** We are working out the details for this process, please contact
us if you need help with this.

Merchants sometimes need to move customer agreements from one merchant
to another, or from one sales unit to another.

Clarification of terms:

* Merchant: A juridical unit, typically called a business or company, identified with
  organization number ("orgno").
* Sales unit: A merchant can have one or more sales units. It may be different
  brands, different physical locations, different services, etc.
* MSN: The unique ID of a sales unit.
  MSN is short for "Merchant Serial Number" which identifies a sales unit.

First: A merchant very rarely "changes organization number".
That only occurs in special cases.
An organization number uniquely identifies a juridical unit and cannot usually be changed.

Typical cases:

* If the merchant wants to move agreements from
   one MSN to another,
   and both MSNs are under the same orgno,
   we may be able to help.
* If the merchant wants to move agreements from
   one orgno to another,
   and both orgnos are owned by the same parent orgno,
   we may be able to help.
* If the merchant wants to move agreements from
   one orgno to another,
   and the orgnos are not owned by a parent company,
   we may be able to help.
   The users have entered agreements with an orgno, and we can't
   automatically move the agreements without the user's consent to enter
   agreements with the new orgno.
   The users may need to enter new agreements with the new orgno.
* If the merchant wants to move agreements between two independent orgnos
   (not owned by a parent orgno),
   splits out a new orgno,
   is merged with another orgno,
   is acquired by another orgno,
   etc.:
   We may *not* be able to help.
   The users have entered agreements with an orgno, and we may not be able to
   automatically move the agreements without the user's consent to enter
   agreements with the new orgno.
   The users may need to enter new agreements with the new orgno.

The general process is:

1. Contact your KAM or partner manager to plan the switch, and
   the date and time for moving the agreements.
2. Log in on
   [portal.vipps.no](https://portal.vipps.no)
   and enter a new agreement with Vipps MobilePay for the new orgno.
3. Order *Faste betalinger* for the new orgno on
   [portal.vipps.no](https://portal.vipps.no).
4. Inform all existing customers of the new orgno,
   and the planned date of the change. If it's a new company number, we need confirmation from you that you have informed your end users that the agreement is being transferred. If it's just a transfer from the old to the new sales unit on the same company number, we don't need this.
5. Send a confirmation to [agreement@vippsmobilepay.com](mailto:agreement@vippsmobilepay.com) that all customers have been informed,
   including a copy of the information sent.
6. Contact your KAM or partner manager to exchange the necessary information to perform the migration:
   * *Vippsnummer* of the sales unit the agreements should be migrated from and to
   * A decision on how to handle existing agreements in the new sales unit (in case you have started using it)
   * Final confirmation of date and time for migration.
7. After the migration is performed, you will get a file that maps from the old to the new agreement ID.
   This is a CSV where each line is an agreement, and the columns are old ID and new ID.

8. You then need to update your systems, so you start using the new agreement IDs

**Please note:** Any downtime experienced by the end-users depend on how your systems deal with not having access to agreement/charge data of existing customers while migrating. If your systems handle this gracefully, the end-users might not experience downtime.

**Please note:** Agreements are not actually moved. Your new sales unit will get new agreements
identical to the old ones, but with new agreement IDs. As a result, it will not be possible for you to see or do anything with previous charges using the new sales unit and agreement IDs. This is done because the underlying payments are tied to the sales unit. So, the new sales unit would not have the authority to capture/refund etc. old charges in any case. This means that in order to perform refunds etc., you need to request to get the old sale unit re-opened or perform them by other means than Vipps MobilePay.

The agreements in the old sales unit will be stopped, and it will not be possible to make new charges on them. The user will still be able to see the old agreement in the app, under "Stopped agreements". There they will find the payment history up to the time of migration.
We recommend keeping track of all agreements a customer has ever been associated with so that you have the opportunity to retrieve previous agreements, charges, etc. if needed.

For all requests to move agreements, please contact your KAM, your partner or
[customer service](https://vipps.no/kontakt-oss/).

**Please note:** If the merchant deactivates an MSN that has active
agreements, it will no longer be possible to:

* Manage the agreements
* Perform new charges
* Make refunds

### Can I look up an agreementId if I have the chargeId?

Yes. See
[Retrieve a charge](https://developer.vippsmobilepay.com/docs/APIs/recurring-api/recurring-api-guide/#retrieve-a-charge).

## Common problems/errors

### Why do I get the error `merchant.not.allowed.for.recurring.operation`?

The `merchant.not.allowed.for.recurring.operation` error indicates
that the Recurring API is not yet activated for this sales unit.

The Recurring API is available for existing customers that
have *Vipps på Nett*, a direct integration with the
[ePayment API](https://developer.vippsmobilepay.com/docs/APIs/epayment-api/) or
[eCom API](https://developer.vippsmobilepay.com/docs/APIs/ecom-api),
and have completed some additional Know Your Customer (KYC) checks required by [Finanstilsynet](https://www.finanstilsynet.no).

Vipps MobilePay is required to perform some extra compliance checks before
activating the Recurring API.

Please order *Faste Betalinger* on
[portal.vipps.no](https://portal.vipps.no)
to get access to the Recurring API in production.

### Is there an API for retrieving information about a Vipps MobilePay user?

Yes. We offer the possibility for merchants, as part of the payment flow, to request the user's information.

For details, see [Userinfo API](https://developer.vippsmobilepay.com/docs/APIs/userinfo-api/userinfo-api-guide/).

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

### How can I change partners for a merchant using the Recurring API?

**Please note:** We are working out the details for this process, please contact
us if you need help with this.

Here is the procedure for what you must do when changing partners:
We need consent from the user site that they will change partners.
They can submit this directly to Vipps MobilePay (via [partner@vippsmobilepay.com](mailto:partner@vippsmobilepay.com)) or
they can submit it through you as a new partner.

The partner orders at ([partner@vippsmobilepay.com](mailto:partner@vippsmobilepay.com)) must receive an e-mail
from the user site or partner stating that they wish to change partners with
the following information:

* Name of user location
* Organization number
* Name of the new partner
* Name of the old partner
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

### How can I change partner for my integration?

See [How to change partners for a merchant](https://developer.vippsmobilepay.com/docs/partner/how-to-change-partners).

**Please note:** The MSN (the number) does *not* change when changing partners.

[draft-agreement-endpoint]: https://developer.vippsmobilepay.com/api/recurring/#tag/Agreement-v3-endpoints/operation/DraftAgreementV3
[fetch-agreement-endpoint]: https://developer.vippsmobilepay.com/api/recurring/#tag/Agreement-v3-endpoints/operation/FetchAgreementV3
[update-agreement-endpoint]: https://developer.vippsmobilepay.com/api/recurring/#tag/Agreement-v3-endpoints/operation/UpdateAgreementPatchV3
[list-charges-endpoint]: https://developer.vippsmobilepay.com/api/recurring/#tag/Charge-v3-endpoints/operation/ListChargesV3
