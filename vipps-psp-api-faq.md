<!-- START_METADATA
---
title: FAQ
sidebar_position: 40
---
END_METADATA -->

# Vipps PSP API FAQ

API version: 3.

Document version 1.2.2.

Vipps Netthandel (eCommerce) via PSP offers functionality for payments on
websites and apps (P2M).

See the [Vipps PSP API guide](vipps-psp-api.md) for more details.

<!-- START_TOC -->

## Table of Contents

- [How can I check if a merchant or sale unit is active?](#how-can-i-check-if-a-merchant-or-sale-unit-is-active)
- [How do we update a transaction?](#how-do-we-update-a-transaction)
- [What is a network token?](#what-is-a-network-token)
- [How can I view the card data?](#how-can-i-view-the-card-data)
- [Is there a unique PSP ID for all merchants?](#is-there-a-unique-psp-id-for-all-merchants)
- [Do we have a test environment?](#do-we-have-a-test-environment)
- [What should we reply to MakePayment() service call in case field "Confirmed" has value: TimeOut or Cancel?](#what-should-we-reply-to-makepayment-service-call-in-case-field-confirmed-has-value-timeout-or-cancel)
- [Can you confirm that Vipps will send makePayment() service with corresponding values in field "Confirmed" if customer declines payment transaction or time-out occurs?](#can-you-confirm-that-vipps-will-send-makepayment-service-with-corresponding-values-in-field-confirmed-if-customer-declines-payment-transaction-or-time-out-occurs)
- [Would it be correct to say that by responding to makePayment() we are informing Vipps about Authorization status and transactionStatusUpdate() informs Vipps about further actions with payment, like Capture/Void/Refund?](#would-it-be-correct-to-say-that-by-responding-to-makepayment-we-are-informing-vipps-about-authorization-status-and-transactionstatusupdate-informs-vipps-about-further-actions-with-payment-like-capturevoidrefund)
- [Is it possible to skip the landing page?](#is-it-possible-to-skip-the-landing-page)
- [What functionality is included in the eCom API, but not the PSP API?](#what-functionality-is-included-in-the-ecom-api-but-not-the-psp-api)
- [Questions?](#questions)

<!-- END_TOC -->

## How can I check if a merchant or sale unit is active?

You can try to initiate payment, specifying the `Merchant-Serial-Number` header:
[`POST:/v3/psppayments/init/`](https://vippsas.github.io/vipps-developer-docs/api/psp#tag/Vipps-PSP-API/operation/initiatePaymentV3UsingPOST).

If the merchant or sale unit is not active, you will get an error:
"Merchant not available or active".

See the Vipps FAQ:
[`Why do I get errorCode 37 "Merchant not available or deactivated or blocked"?`](https://github.com/vippsas/vipps-developers/blob/master/faqs/common-errors-faq.md#why-do-i-get-errorcode-37-merchant-not-available-or-deactivated-or-blocked).

## How do we update a transaction?

Every operation done to a transaction after it has been processed should be
updated to our
[Update status endpoint](https://vippsas.github.io/vipps-developer-docs/api/psp#tag/Vipps-PSP-API/operation/updatestatusUsingPOST).
This includes partial refunds, captures etc. This is critical for support work
and user experience. This goes both for the single payment flow and for recurring payments.

Note that you do not need to send an update for the _reservation_ part of the
single payment flow, since that is handled by the synchronous response to the
`Makepayment` call. But you must send it for the Recurring flow.

## What is a network token?

A token is a "representative card number" for performing card payments, without using the actual
card details. The EMVco token number is not considered PCI DSS sensitive.

See the
[EMVco documentation](https://www.emvco.com/emv-technologies/payment-tokenisation/)
for more.

## Why do I get `No network token available for this Agreement` or similar?

The most common reason for this problem is that the user has a card issuer that
does not support network tokens.

Banks can block tokens, for instance if the card is blocked, and are not always
great at reactivating them.

It could also be that the user has not added his/her new card in Vipps.

## Is there a unique PSP ID for all merchants?

No, the PSP ID is unique for the PSP and used for all the PSP's merchants.

## Do we have a test environment?

Yes, please see: [The Vipps test environment (MT)](https://github.com/vippsas/vipps-developers/blob/master/developer-resources/test-environment.md).

## What should we reply to MakePayment() service call in case field "Confirmed" has value: TimeOut or Cancel?

Continued:
Should it be: paymentInfo.status = "OK" or "FAIL"?
If it is "FAIL", what error code should be returned?

It is suggested that you send "FAIL" in case of failure either at your end or as in this case at our end.

If field "Confirmed" has value: `TimeOut` or `Cancel` -> `paymentInfo.status` = "FAIL"?

## Can you confirm that Vipps will send makePayment() service with corresponding values in field "Confirmed" if customer declines payment transaction or time-out occurs?

From the `makePayment` specification:

| Name | Type | Size | Optional | Values |
| ---- | ---- | ---- | -------- | ------ |
| confirmed	| String | 7 | No | YES/TIMEOUT/CANCEL |

## Would it be correct to say that by responding to makePayment() we are informing Vipps about Authorization status and transactionStatusUpdate() informs Vipps about further actions with payment, like Capture/Void/Refund?

Yes.

## Is it possible to skip the landing page?

Skipping the landing page is reserved for special cases, where displaying it is not possible.

See the FAQ for more details:
[Is it possible to skip the landing page?](https://github.com/vippsas/vipps-developers/blob/master/faqs/vipps-landing-page-faq.md#is-it-possible-to-skip-the-landing-page)

## What functionality is included in the eCom API, but not the PSP API?

See the
[Vipps eCom API FAQ](https://github.com/vippsas/vipps-ecom-api/blob/master/vipps-ecom-api-faq.md#what-functionality-is-included-in-the-ecom-api-but-not-the-psp-api).

## Questions?

We're always happy to help with code or other questions you might have!
Please create an [issue](https://github.com/vippsas/vipps-psp-api/issues),
a [pull request](https://github.com/vippsas/vipps-psp-api/pulls),
or [contact us](https://vippsas.github.io/vipps-developer-docs/docs/vipps-developers/contact).

Sign up for our [Technical newsletter for developers](https://vippsas.github.io/vipps-developer-docs/docs/vipps-developers/newsletters).
