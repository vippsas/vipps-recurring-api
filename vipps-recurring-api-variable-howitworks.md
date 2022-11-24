<!-- START_METADATA
---
title: How It Works
sidebar_position: 10
---
END_METADATA -->

# Vipps Recurring API with variable amount: How It Works

<!-- START_COMMENT -->

ℹ️ Please use the new documentation:
[Vipps Technical Documentation](https://vippsas.github.io/vipps-developer-docs/).

<!-- END_COMMENT -->

For technical documentation go to:
[Vipps Recurring API](https://vippsas.github.io/vipps-developer-docs/docs/APIs/recurring-api/).

## Recurring variable amount pricing

![recurring process](images/vipps-recurring-api-howitworks/vipps-recurring-process.svg)

## 1. Buy a subscription with Vipps

A user choose Vipps as payment method for a subscription on a merchant's website or app. 

![Buy subscription with Vipps](images/vipps-recurring-api-howitworks/vipps-recurring-step1.svg)

## 2. The Vipps landing page

If the agreement was started on a desktop device, the user will be sent to the Vipps landing page.
The user confirms their number and is prompted to log in to Vipps.

If the agreement was started from a mobile device, the app will automatically swtich over to Vipps.

![Vipps landing page](images/vipps-recurring-api-howitworks/vipps-recurring-step2.png)

## 3. Accept agreement in Vipps

The user receives a push notification on their phone. They log in to Vipps and accept the agreement.

If there is an initial charge to be paid, the user confirms the payment in this step as well.

In this step the user can choose the max amount that they will allow the merchant to charge each interval. 
The suggested amount from the merchant is preselected in the list.

![Buy subscription with Vipps](images/vipps-recurring-api-variable-howitworks/variable_amount_accept.png)

## 5. Confirmation of subscription

The user is redirected back to the merchant’s website or app, and the subscription is confirmed on the merchant’s page.

![Confirmation of subscription](images/vipps-recurring-api-howitworks/vipps-recurring-step4.svg)

## 6. Due charges

Upcoming charges is displayed in Vipps for the user to see.
If a charge fails for any reason and there is retry days left, a failure text is visible for the user on the charge.
This text will explain to the user what went wrong and how they can fix it.

### Extra notification if users max amount is too low

If the user has a lower max amount than the charge that is created, the user will get a push notification.
An explanation text is also shown on the charge in Vipps.

This notification and failure text is triggered and shown when the charge is set as due.

![Failure text](images/vipps-recurring-api-variable-howitworks/variable_amount_notification.png)

## 7. Overview and manage agreement

The user can see their active subscriptions, as well as stopped subscriptions, and get an overview of previous charges in Vipps.

If the user clicks on max amount, they can choose a new max amount for the agreement.

![Change max amount](images/vipps-recurring-api-variable-howitworks/variable_amount_manage.png)

If the user clicks on `Manage agreement` (Administrer abonnement) in the previous step, they will be sent to the merchant’s site where they can manage their subscriptions. We recommend using [Vipps Login](https://vippsas.github.io/vipps-developer-docs/docs/APIs/login-api/) to make login to the merchant’s site go smoothly.

![Manage agreements](images/vipps-recurring-api-howitworks/vipps-recurring-step6.svg)

## Questions?

We're always happy to help with code or other questions you might have!
Please create an [issue](https://github.com/vippsas/vipps-recurring-api/issues),
a [pull request](https://github.com/vippsas/vipps-recurring-api/pulls),
or [contact us](https://vippsas.github.io/vipps-developer-docs/docs/vipps-developers/contact).

Sign up for our [Technical newsletter for developers](https://vippsas.github.io/vipps-developer-docs/docs/vipps-developers/newsletters).
