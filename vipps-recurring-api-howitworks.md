# Vipps Recurring API: How It Works

Vipps Recurring can be used for subscriptions, donations or other recurring payments.

For technical documentation go to:
[Vipps Recurring API](https://github.com/vippsas/vipps-recurring-api).

## The recurring payment process

![recurring process](images/vipps-recurring-process.svg)


## 1. Buy a subscription with Vipps

A user chooses Vipps as payment method for a subscription on a merchant’s website or app.

![Buy subscription with Vipps](images/vipps-recurring-step1.svg)

## 2. The Vipps landing page

If the payment was started on a desktop device the user will be sent to the Vipps landing page.
The user confirms their number, and is prompted to log in to Vipps.

If the payment was started from a mobile device, the app wil automatically switch over to Vipps.

![Vipps landing page](images/vipps-recurring-step2.svg)

## 3. Accept agreement in Vipps

The user receives a push notification on their phone. They log in to Vipps, and accept the agreement.

If there is an initial charge to be made, the user confirms the payment in this step as well

![Accept agreement](images/vipps-recurring-step3-v3.svg)

## 4. Confirmation of subscription

The user is redirected back to the merchant’s web site or app, and the subscription is confirmed on the merchant’s page.

![Confirmation of subscription](images/vipps-recurring-step4.svg)

## 5. Overview of agreements and charges

The user can see their active subscriptions, as well as stopped subscriptions, and get an overview of previous charges in Vipps.

![Overview of agreements](images/vipps-recurring-step5-v3.svg)

## 6. Manage agreements

If the user click on “Manage agreement” in the previous step, they will be sent to the merchant’s site and the user can manage their subscriptions. We recommend using [Vipps Login](https://github.com/vippsas/vipps-login-api) to make login to the merchant’s site go smoothly.

![Manage agreements](images/vipps-recurring-step6.svg)

## Great! Now you know how the Vipps Recurring payment process works.

Take a look at the technical documentation in the [Vipps Recurring API Guide](https://github.com/vippsas/vipps-recurring-api/blob/master/vipps-recurring-api.md)


## Questions?

We're always happy to help with code or other questions you might have!
Please create an [issue](https://github.com/vippsas/vipps-recurring-api/issues),
a [pull request](https://github.com/vippsas/vipps-recurring-api/pulls),
or [contact us](https://github.com/vippsas/vipps-developers/blob/master/contact.md).

Sign up for our [Technical newsletter for developers](https://github.com/vippsas/vipps-developers/tree/master/newsletters).
