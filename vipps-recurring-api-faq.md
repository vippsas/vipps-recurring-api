# Vipps Recurring API FAQ

# How do I check my customer's status?
By using a `GET:/api/v1/agreement` you can get an overview of all agreements,
if you are uncertain about a specific customer's agreement you can do  
`GET:/api/v1/charge/{agreementId}`.

# A customer's charge failed but I did not receive any warning
As per right now there is no warning or callback tied to charges, a merchant
has to poll to check their user's status. A warning on a failed charge will
quite likely be implemented shortly based on feedback.

# I don't want a charge to fail the first time the transaction fails (insufficient funds / networking issues etc.)
The field "retryDays" in Agreement allows for this functionality, Vipps will
retry once each day until the value is reached. The valid values are none
(defaulting to 3) or 0-14. We strongly recommend having more than 1 retry day
to account for possible networking issues etc.

# When can I send charges for a user?
You can send charges once you have polled and found a valid agreement tied to
the user.

# Can a user cancel the agreement through the Vipps app
No, the user needs to contact the Merchant which can then cancel or modify the
agreement as they see fit.

# I have not yet integrated with the app. How can I set an agreement to Active in test?
By sending an update agreement call in test with the status "Active" the
agreement can be turned to active. An example of this call can be seen in the
Postman collection.

# How does a user see any charges I send?
A charge will be displayed to the user 8 days before the money are reserved.
The charge will then appear in the app. You can still retrieve all relevant
charges through the API.

# If a user changes the default payment card in Vipps, can new charges be made to that card?
No, all charges will be made to the payment card.

# Is is necessary to reserve 1 NOK to verify a card?
Yes. The reservation should have an understandable description, like
"Temporary reservation to confirm card for Acme Inc."
