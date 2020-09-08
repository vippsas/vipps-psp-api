# Vipps PSP API version 3

This repository contains developer resources for the Vipps PSP API v3.

For more information about this product, please see
[Vipps på Nett via PSP](https://vipps.no/produkter-og-tjenester/bedrift/ta-betalt-paa-nett/ta-betalt-paa-nett/#kom-i-gang-med-vipps-pa-nett-category-2).

* [Getting Started](https://github.com/vippsas/vipps-developers/blob/master/vipps-getting-started.md): Information about API keys, product activation.
* [Vipps PSP API guide](vipps-psp-api.md): Developer guide for Vipps PSP API.
* [Vipps PSP API Checklist](vipps-psp-api-checklist.md): For PSP integrations.
* [Frequently Asked Questions](vipps-psp-api-faq.md): Questions and answers.
* [Postman collection](tools/): Collection and environment for testing using Postman.

You can peruse the API reference documentation as:
* [Swagger](https://vippsas.github.io/vipps-psp-api)

Vipps do not handle the settlements for PSP integrations. These are handled by the PSP.

See the [Vipps Developers repository](https://github.com/vippsas/vipps-developers)
for contact information, contribution guidelines, etc.

## Differences between v2 and v3

The PSP API v3 adds functionality for network tokens: PSPs use the API to
obtain tokens, not the actual card details.

From January 1 2021 all PSPs must be able to process network tokens.

See the
[EMVco documentation](https://www.emvco.com/emv-technologies/payment-tokenisation/)
for more information.

The PSP API v2 has been archived: https://github.com/vippsas/vipps-psp-api-v2

# Questions?

We're always happy to help with code or other questions you might have!
Please create an [issue](https://github.com/vippsas/vipps-psp-api/issues),
a [pull request](https://github.com/vippsas/vipps-psp-api/pulls),
or [contact us](https://github.com/vippsas/vipps-developers/blob/master/contact.md).

Sign up for our [Technical newsletter for developers](https://github.com/vippsas/vipps-developers/tree/master/newsletters).
