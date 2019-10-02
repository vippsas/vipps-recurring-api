# Vipps Recurring API FAQ

## How do I check my customer's status?
By using a `GET:/v2/agreements` you can get an overview of all agreements,
if you are uncertain about a specific customer's agreement you can do  
`GET:/v2/agreements/{agreementId}`.

## A customer's charge failed but I did not receive any warning
The customer does not have notification turned on,
or they have not upgraded to the app version that supports it.

## I don't want a charge to fail the first time the transaction fails (insufficient funds / networking issues etc.)
The field "retryDays" in Agreement allows for this functionality, Vipps will
retry once each day until the value is reached. The valid values are none
(defaulting to 0) or 0-14. We strongly recommend having more than 1 retry day
to account for possible networking issues etc.

## Are there any limits on charging a user?
Yes, within the Interval period of the agreement the merchant can charge at most 10
times the agreement price cumulatively. There is no limit on the number of charges
which can be sent in the Interval period. For example, a monthly agreement of 500NOK,
can have at most 5000NOK charged in that month, either through one 5000NOK charge,
5000 1NOK charges, or anywhere in between. 

## When can I send charges for a user?
You can send charges once you have polled and found a valid agreement tied to
the user.

## Can a user cancel the agreement through the Vipps app
No, the user needs to contact the Merchant which can then cancel or modify the
agreement as they see fit.

## How does a user see any charges I send?
A charge will be displayed to the user 6 days before the charge is due to be processed.
The charge will then appear in the app. You can still retrieve all relevant
charges through the API.

## If a user changes the default payment card in Vipps, can new charges be made to that card?
No, currently the payment card tied to an agreement will not be updated automatically.
