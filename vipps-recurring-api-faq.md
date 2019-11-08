# Vipps Recurring API FAQ

Document version: 1.1.1.

# Table of Contents

- [At what time during the day charges made](#At-what-time-during-the day-charges-made)
- [How do I check my customer's status?](#how-do-i-check-my-customer-s-status-)
- [A customer's charge failed but I did not receive any warning](#a-customer-s-charge-failed-but-i-did-not-receive-any-warning)
- [I don't want a charge to fail the first time the transaction fails (insufficient funds / networking issues etc.)](#i-don-t-want-a-charge-to-fail-the-first-time-the-transaction-fails--insufficient-funds---networking-issues-etc-)
- [Can the charge for an agreement be changed?](#can-the-charge-for-an-agreement-be-changed-)
- [Are there any limits on charging a user?](#are-there-any-limits-on-charging-a-user-)
- [When can I send charges for a user?](#when-can-i-send-charges-for-a-user-)
- [Can a user cancel the agreement through the Vipps app](#can-a-user-cancel-the-agreement-through-the-vipps-app)
- [How does a user see any charges I send?](#how-does-a-user-see-any-charges-i-send-)
- [If a user changes the default payment card in Vipps, can new charges be made to that card?](#if-a-user-changes-the-default-payment-card-in-vipps--can-new-charges-be-made-to-that-card-)
- [If a user's card expires: What happens on the next charge?](#if-a-user-s-card-expires--what-happens-on-the-next-charge-)
- [Settlement](#settlement)
- [Invoicing](#invoicing)

# At what time during the day charges made
Charge _attempts_ are made two times during the day: 08:00 og 16:00 UTC.
Subsequent attempts are made accoring to the `retryDays` specified.

# How do I check my customer's status?
By using a
[`GET:/v2/agreements`]()
you can get an overview of all agreements,
if you are uncertain about a specific customer's agreement you can do  
`GET:/v2/agreements/{agreementId}`.

# A customer's charge failed but I did not receive any warning
The customer does not have notification turned on,
or they have not upgraded to the app version that supports it.

# I don't want a charge to fail the first time the transaction fails (insufficient funds / networking issues etc.)
The field "retryDays" in Agreement allows for this functionality, Vipps will
retry once each day until the value is reached. The valid values are none
(defaulting to 0) or 0-14. We strongly recommend having more than 1 retry day
to account for possible networking issues etc.

# Can the charge for an agreement be changed?
Yes. See "Are there any limits on charging a user?"

# Are there any limits on charging a user?
Yes, within the Interval period of the agreement the merchant can charge at most 10
times the agreement price cumulatively. There is no limit on the number of charges
which can be sent in the Interval period.

For example, a monthly agreement of 500NOK,
can have at most 5000NOK charged in that month, either through one 5000NOK charge,
5000 1NOK charges, or anywhere in between.

Vipps recommends creating a new agreement if there is a significant price change.
It is the merchant's responsibility to make sure the user is informed and understands
the price of the agreement.

# When can I send charges for a user?
You can send charges once you have polled and found a valid agreement tied to
the user.

# Can a user cancel the agreement through the Vipps app
No, the user needs to contact the Merchant which can then cancel or modify the
agreement as they see fit.

# How does a user see any charges I send?
A charge will be displayed to the user 6 days before the charge is due to be processed.
The charge will then appear in the app. You can still retrieve all relevant
charges through the API.

# If a user changes the default payment card in Vipps, can new charges be made to that card?
No, currently the payment card tied to an agreement will not be updated automatically.

# If a user's card expires: What happens on the next charge?
The user will get a notification in Vipps about the expired card and that the charge failed.
The user has to update the card, and also update the agreement to use the new card.
Vipps does not automatically select a new card if a card expires, as users may
have multiple cards registered in Vipps.

# Settlement
The settlements are done trough Vipps.
The merchant does not need any other partner or agreement.

# Invoicing
Merchants with a "net settlement" contract receive the users' payments excluding the Vipps fees.
Merchants with a "gross settlement" contract receive the users' payments including the Vipps fees,
and are then invoiced for the Vipps fees.
