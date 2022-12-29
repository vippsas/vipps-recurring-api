<!-- START_METADATA
---
title: Implementation Inspiration
sidebar_position: 13
---
END_METADATA -->

# Implementation Inspiration

<!-- START_COMMENT -->

ℹ️ Please use the new documentation:
[Vipps Technical Documentation](https://vippsas.github.io/vipps-developer-docs/).

<!-- END_COMMENT -->

## Introduction
This page is meant to give some inspiration for how you can utilize the features of recurring to fullfill a variety of needs, from special one-of agreements to referral discounts and loyalty programs.

## Unique agreements/campaigns
In recurring, every agreement is unique, and every campaign is unique to the specific agreement they belong to. This means that you as an integrator have the ability to taylor agreements and campaigns to your individual users.
This can be used to give customers special deals on special occations, based on referrals, etc.

## Birthday discount
If you have a user that has deactivated their subscription, and you want to give them an incentive to re-subscribe, why not give them a personalized discount?
```json
{
  ...
  "initialCharge": {
     "amount": 24900,
     "description": "Initial charge",
     "transactionType": "DIRECT_CAPTURE"
  },
  "pricing": {
    "amount": 34900,
    "currency": "NOK"
  },
  "campaign": {
    "type": "PERIOD_CAMPAIGN",
    "price": 24900,
    "period": {
      "unit": "MONTH",
      "count": 1
    }
  },
  "productDescription": "Birthday discount"
  ...
}
```
![Drafted agrement with birthday discount for one month](images/howitworks-inspiration/birthday-discount.jpg)

## Referrals
If you want to use referrals, you can do this in many ways. A commonly used way is to assign your users a referral-code (in your systems), which new users then can enter when registering. You can then use a special campaign when drafting their agreements, like this:
</br>[`POST:/agreements`]
```json
{
  ...
  "initialCharge": {
     "amount": 24900,
     "description": "Initial charge",
     "transactionType": "DIRECT_CAPTURE"
  },
  "pricing": {
    "amount": 34900,
    "currency": "NOK"
  },
  "campaign": {
    "type": "PERIOD_CAMPAIGN",
    "price": 24900,
    "period": {
      "unit": "MONTH",
      "count": 1
    }
  },
  "productDescription": "Referral discount"
  ...
}
```
![Drafted agrement with birthday discount for one month](images/howitworks-inspiration/referral-discount.jpg)
To reward the user that referred them, you can change their agreement price and description like this:
</br>[`PATCH:/agreements/{agreementId}`]
```json
{
  "productDescription": "Referral discount active (-100 kr)",
  "pricing": {
    "amount": 24900
  }
}
```
![Drafted agrement with birthday discount for one month](images/howitworks-inspiration/agreement-with-referral-discount.jpg)

## Additional services/purchases
Sometimes your customers might want to extend their subscription with additional services or one-time purchases. For additional services that are recurring in nature, it might be apt to have separate agreements for each. For others, you might just want to change the price of the existing agreement, or even create a new agreement alltogether in order to collect an initial charge to activate the additional service. An alternative to creating a new agreement is to utilize a normal ecommerce-transaction to charge your customer before altering the agreement.

## Loyalty programs
A good way to ensure customers stay is to have a good loyalty program. As with referrals, this can be achieved by using campaigns and/or updating the agreement to give a discount for loyal customers. Some scenarios:

>### Reward purchasing additional services
>If you want to give members a discounted subscription fee when they have purchased additional services for a certain amount,
>you can achieve this by simply updating their agreement with the new pricing and a product description that explains the discount.
></br>To remove the discount, you simply update the agreement again with the original pricing and description
></br><b/>NB:</b> Remember to also use the new pricing when creating charges

>### Reward being subscribed for an amount of time
>If you want to give members a discounted subscription fee when they have been subscribed for a long time, you can do this in a couple of ways:
></br>- Update existing agreements as previous scenario
></br>- Use a `FULL_FLEX_CAMPAIGN` to display to the new customers that they will get a better prioce if they stay subscribed
></br>Here is an example of the latter, where they pay 50kr/week until march 1st, then 200kr/month after that:
```json
{
  ...
  "initialCharge": {
     "amount": 5000,
     "description": "Initial charge",
     "transactionType": "DIRECT_CAPTURE"
  },
  "pricing": {
    "amount": 20000,
    "currency": "NOK"
  },
  "interval": { "unit": "MONTH", "count": 1 },
  "campaign": {
    "type": "FULL_FLEX_CAMPAIGN",
    "price": 5000,
    "interval": {
      "unit": "WEEK",
      "count": 1
    }
  }
  ...
}
```
![Agreement with loyalty campaign](images/howitworks-inspiration/loyalty-campaign.jpg)

