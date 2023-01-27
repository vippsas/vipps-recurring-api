<!-- START_METADATA
---
title: Introduction
sidebar_position: 1
hide_table_of_contents: true
pagination_next: null
pagination_prev: null
---
END_METADATA -->

# Vipps Recurring API

<!-- START_COMMENT -->

ℹ️ Please use the new documentation:
[Vipps Technical Documentation](https://vippsas.github.io/vipps-developer-docs/).

<!-- END_COMMENT -->

The Recurring API allows you to set up recurring payments for your customers.
This allows your customers to pay subscriptions, donate, or make other recurring purchases without keying in card numbers or filling in long forms. In addition, your customers get a good overview of their fixed payments and they get notification of withdrawals through Vipps.

Information about this product in Norwegian, see
[Faste betalinger med Vipps](https://www.vipps.no/produkter-og-tjenester/bedrift/faste-betalinger/faste-betalinger/).

**IMPORTANT:** The Vipps Recurring API is available for existing customers that
have "Vipps på Nett", a direct integration with the
[Vipps eCom API](https://vippsas.github.io/vipps-developer-docs/docs/APIs/recurring-api),
and have completed some additional KYC checks required by Finanstilsynet.
Vipps is required to perform some extra compliance checks before
activating the Vipps Recurring API.
please order “Vipps Faste betalinger" on
[portal.vipps.no](https://portal.vipps.no)
to get access to the Recurring API in production.

## How it works

* [How Vipps Recurring works](./how-it-works/vipps-recurring-api-howitworks.md): An overview of how the Recurring API works.
* [How Recurring works with Vipps Login](./how-it-works/vipps-login-recurring-howitworks.md): Learn how you can use Login and Recurring together.
* [How Recurring works with variable amount](./how-it-works/vipps-recurring-api-variable-howitworks.md): Learn about using variable amounts.
* [How Recurring works with campaign](./how-it-works/vipps-recurring-api-campaigns-howitworks.md): Learn about using Recurring campaigns.
* [How Recurring works with Vipps Login](./how-it-works/vipps-login-recurring-howitworks.md): Learn about using Recurring with Vipps login.
* [Implementation inspiration](./how-it-works/vipps-recurring-api-inspiration-howitworks.md): Get inspiration for how to use Recurring.


## Getting started

See
[Vipps Getting started guide](https://vippsas.github.io/vipps-developer-docs/docs/vipps-developers/vipps-getting-started)
for information about API keys, product activation, how to make API calls, etc.

Review the detailed documentation found here:

* [API quick start](vipps-recurring-api-quick-start.md): Quick start.
* [API guide](vipps-recurring-api.md): Developer guide for Vipps Recurring API.
* [API checklist](vipps-recurring-api-checklist.md): For direct and POS integrations.
* [API FAQ](vipps-recurring-api-faq.md): Questions and answers.
* [API reference](https://vippsas.github.io/vipps-developer-docs/api/recurring): Recurring API reference specifications.
