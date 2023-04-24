---
title: How Recurring works with campaign
sidebar_label: Campaign
sidebar_position: 14
description: How Recurring works with campaign
pagination_next: null
pagination_prev: null
---

# How Recurring works with campaign

For technical documentation go to:
[Vipps Recurring API](https://developer.vippsmobilepay.com/docs/APIs/recurring-api).

## 1. Buy a subscription 

A user chooses Vipps or MobilePAy as payment method for a subscription on a merchant's website or app.

![Buy subscription with Vipps](../images/vipps-recurring-api-howitworks/vipps-recurring-step1.svg)

## 2. The landing page

If the agreement was started on a desktop device, the user will be sent to the Vipps or MobilePay landing page.
The user confirms their number and is prompted to log in to Vipps or MobilePay.

If the agreement was started from a mobile device, the app will automatically switch over to Vipps or MobilePay.

![Vipps or MobilePay landing page](../images/vipps-recurring-api-howitworks/vipps-recurring-step2.png)

## 3. Accept agreement  

The user receives a push notification on their phone. They log in to Vipps or MobilePay and accept the agreement.

If there is an initial charge to be paid, the user confirms the payment in this step as well.

### Agreement with campaign

![Accept agreement](../images/vipps-recurring-api-howitworks-campaigns/agreement-campaign-flow.png)

### Agreement with campaign and initial charge

![Accept agreement](../images/vipps-recurring-api-howitworks-campaigns/agreement-campaign-initial-charge-flow.png)

## 4. Confirmation of subscription

The user is redirected back to the merchant’s website or app, and the subscription is confirmed on the merchant’s page.

![Confirmation of subscription](../images/vipps-recurring-api-howitworks/vipps-recurring-step4.svg)

## 5. Overview of agreements and charges

The user can see their active subscriptions, as well as stopped subscriptions, and get an overview of previous charges in Vipps or MobilePay.
Campaign information will be displayed on the active and stopped subscriptions until the campaign has expired. After that, the regular price and interval will be displayed.

### Agreement with running campaign

![Overview of agreements](../images/vipps-recurring-api-howitworks-campaigns/manage-agreement-with-campaign.png)

### Agreement after campaign as expired

![Overview of agreements](../images/vipps-recurring-api-howitworks-campaigns/manage-agreement-with-campaign-expired.png)

## 6. Manage agreements

If the user clicks on “Manage agreement” in the previous step, they will be sent to the merchant’s site where they can manage their subscriptions. We recommend using [Vipps MobilePay Login](https://developer.vippsmobilepay.com/docs/APIs/login-api) to make login to the merchant’s site go smoothly.

![Manage agreements](../images/vipps-recurring-api-howitworks/vipps-recurring-step6.svg)

## Great! Now you know how the Recurring agreements with campaign process works.
