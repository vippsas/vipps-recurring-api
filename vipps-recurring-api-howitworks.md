<!-- START_METADATA
---
title: How It Works
sidebar_position: 10
---
END_METADATA -->

# Vipps Recurring API: How It Works

<!-- START_COMMENT -->

ℹ️ Please use the new documentation:
[Vipps Technical Documentation](https://vippsas.github.io/vipps-developer-docs/).

<!-- END_COMMENT -->

Vipps Recurring can be used for subscriptions, donations or other recurring payments.

For technical documentation go to:
[Vipps Recurring API](https://vippsas.github.io/vipps-developer-docs/docs/APIs/recurring-api).

## The recurring payment process

![recurring process](images/vipps-recurring-api-howitworks/vipps-recurring-process.svg)


## 1. Buy a subscription with Vipps

A user chooses Vipps as payment method for a subscription on a merchant’s website or app.

![Buy subscription with Vipps](images/vipps-recurring-api-howitworks/vipps-recurring-step1.svg)

## 2. The Vipps landing page

If the agreement was started on a desktop device, the user will be sent to the Vipps landing page.
The user confirms their number and is prompted to log in to Vipps.

If the agreement was started from a mobile device, the app will automatically switch over to Vipps.

![Vipps landing page](images/vipps-recurring-api-howitworks/vipps-recurring-step2.png)

## 3. Accept agreement in Vipps

The user receives a push notification on their phone. They log in to Vipps and accept the agreement.

If there is an initial charge to be paid, the user confirms the payment in this step as well.

![Accept agreement](images/vipps-recurring-api-howitworks/vipps-recurring-step3.png)

## 4. Confirmation of subscription

The user is redirected back to the merchant’s website or app, and the subscription is confirmed on the merchant’s page.

![Confirmation of subscription](images/vipps-recurring-api-howitworks/vipps-recurring-step4.svg)

## 5. Overview of agreements and charges

The user can see their active subscriptions, as well as stopped subscriptions, and get an overview of previous charges in Vipps.

![Overview of agreements](images/vipps-recurring-api-howitworks/vipps-recurring-step5.png)

## 6. Manage agreements

If the user clicks on “Manage agreement” in the previous step, they will be sent to the merchant’s site where they can manage their subscriptions. We recommend using [Vipps Login](https://vippsas.github.io/vipps-developer-docs/docs/APIs/login-api) to make login to the merchant’s site go smoothly.

![Manage agreements](images/vipps-recurring-api-howitworks/vipps-recurring-step6.svg)

## Great! Now you know how the Vipps Recurring payment process works.

Take a look at the technical documentation in the [Vipps Recurring API Guide](vipps-recurring-api.md).
