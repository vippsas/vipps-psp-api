<!-- START_METADATA
---
title: FAQ
sidebar_position: 40
pagination_next: null
---
END_METADATA -->

# FAQs

Vipps Netthandel (eCommerce) via PSP offers functionality for payments on
websites and apps (P2M).

See the [Vipps PSP API guide](vipps-psp-api.md) for all the details.

For more common Vipps questions, see:

* [Vipps API General FAQ](https://vippsas.github.io/vipps-developer-docs/docs/vipps-developers/faqs)

API version: 3.0.0.

ℹ️ Please use the new documentation:
[Vipps Technical Documentation](https://vippsas.github.io/vipps-developer-docs/).

## Table of contents

* [How can I check if a merchant or sale unit is active?](#how-can-i-check-if-a-merchant-or-sale-unit-is-active)
* [Why do I get an `Invalid MSN` error?](#why-do-i-get-an-invalid-msn-error)
* [How can I update the status of a payment?](#how-can-i-update-the-status-of-a-payment)
* [What is a network token?](#what-is-a-network-token)
* [What is a PAN?](#what-is-a-pan)
* [Why do I get `No network token available for this Agreement` or similar?](#why-do-i-get-no-network-token-available-for-this-agreement-or-similar)
* [Is there a unique PSP ID for all merchants?](#is-there-a-unique-psp-id-for-all-merchants)
* [Does Vipps have a test environment?](#does-vipps-have-a-test-environment)
* [What is the correct response to the `MakePayment` request?](#what-is-the-correct-response-to-the-makepayment-request)
* [Can you confirm that Vipps will send makePayment() service with corresponding values in field "Confirmed" if customer declines payment transaction or time-out occurs?](#can-you-confirm-that-vipps-will-send-makepayment-service-with-corresponding-values-in-field-confirmed-if-customer-declines-payment-transaction-or-time-out-occurs)
* [Is it possible to skip the landing page?](#is-it-possible-to-skip-the-landing-page)
* [What functionality is included in the eCom API, but not the PSP API?](#what-functionality-is-included-in-the-ecom-api-but-not-the-psp-api)

<!-- END_COMMENT -->

## How can I check if a merchant or sale unit is active?

You can try to initiate payment, specifying the `Merchant-Serial-Number` header:
[`POST:/v3/psppayments/init/`](https://vippsas.github.io/vipps-developer-docs/api/psp#tag/Vipps-PSP-API/operation/initiatePaymentV3UsingPOST).

If the merchant or sale unit is not active, you will get an error:
"Merchant not available or active".

See the Vipps FAQ:
[`Why do I get errorCode 37 "Merchant not available or deactivated or blocked"?`](https://vippsas.github.io/vipps-developer-docs/docs/vipps-developers/faqs/common-errors-faq#why-do-i-get-errorcode-37-merchant-not-available-or-deactivated-or-blocked).

## Why do I get an `Invalid MSN` error?

You may get an error like this:

```
Invalid MSN: 123456. This MSN is not valid for the provided PSP id.
Check that you are using the correct credentials for the right environment.
```

This happens if the MSN is created with one PSP ID and you attempt to initiate a
payment with a different PSP ID.

## How can I update the status of a payment?

When the PSP has new information about a payment, it is important to send the
new information to Vipps.

The PSP must use the
[`POST:/v3/psppayments/updatestatus`](https://vippsas.github.io/vipps-developer-docs/api/psp#tag/Vipps-PSP-API/operation/updatestatusUsingPOST)
endpoint to notify Vipps of changes to the payment,

See:
[Status updates](vipps-psp-api#status-updates).

## What is a network token?

Network tokens are payment credentials that replace the card details for online
payments. Every merchant gets a unique token for the card, so a network token
can not be shared between businesses.

The EMVco token is not considered
[PCI DSS sensitive](https://www.pcisecuritystandards.org/document_library/?document=pci_dss).
See the
[EMVco documentation](https://www.emvco.com/emv-technologies/payment-tokenisation/)
for more.

## What is a PAN?

The PAN (Primary account numbers) is the 15- or 16-digit number found on all
credit or debit cards. Since PANs are accepted for online payments, anyone with
access to the PAN may be able to make payments without the physical card.
A PAN is unique per card, but may be shared between businesses.

## Why do I get `No network token available for this Agreement` or similar?

The most common reason for this problem is that the user has a card issuer that
does not support network tokens.

Banks can block tokens, for instance if the card is blocked, and are not always
great at reactivating them.

It could also be that the user has not added his/her new card in Vipps.

## Is there a unique PSP ID for all merchants?

No, the PSP ID is unique for the PSP and used for all the PSP's merchants.

## Does Vipps have a test environment?

Yes, please see: [The Vipps test environment (MT)](https://vippsas.github.io/vipps-developer-docs/docs/vipps-developers/test-environment).

## What is the correct response to the `MakePayment` request?

**TODO: This needs a rewrite by someone who knows the details.**

If `Confirmed` has value `TimeOut` or `Cancel`

Continued:
Should it be: paymentInfo.status = "OK" or "FAIL"?
If it is "FAIL", what error code should be returned?

It is suggested that you send "FAIL" in case of failure either at your end or as in this case at our end.

If field "Confirmed" has value: `TimeOut` or `Cancel` -> `paymentInfo.status` = "FAIL"?

## Can you confirm that Vipps will send makePayment() service with corresponding values in field "Confirmed" if customer declines payment transaction or time-out occurs?

**TODO: This (including the heading) needs a rewrite by someone who knows the details.**

From the `makePayment` specification:

| Name      | Type   | Size | Optional | Values             |
| --------- | ------ | ---- | -------- | ------------------ |
| confirmed | String |  7   | No       | YES/TIMEOUT/CANCEL |

## Is it possible to skip the landing page?

Skipping the landing page is reserved for special cases, where displaying it is not possible.

See the FAQ for more details:
[Is it possible to skip the landing page?](https://vippsas.github.io/vipps-developer-docs/docs/vipps-developers/faqs/vipps-landing-page-faq#is-it-possible-to-skip-the-landing-page)

## What functionality is included in the eCom API, but not the PSP API?

See the
[Vipps API FAQ](https://vippsas.github.io/vipps-developer-docs/docs/vipps-developers/faqs/other-faq#what-functionality-is-included-in-the-ecom-api-but-not-the-psp-api).
