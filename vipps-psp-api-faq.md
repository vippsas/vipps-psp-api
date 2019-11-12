# Vipps PSP API FAQ

API version: 2.

Document version 0.1.3.

_**IMPORTANT:**_ This document is a work in progress.

# About this API

Vipps Netthandel (eCommerce) via PSP offers functionality for payments on
websites and apps (P2M).

See the [guide](vipps-psp-api.md) for more details.

# Table of Contents

- [What is the card data structure?](#what-is-the-card-data-structure-)
  * [How can I view the card data?](#how-can-i-view-the-card-data-)
- [Is there a unique PSP ID for all merchants?](#is-there-a-unique-psp-id-for-all-merchants-)
- [Do we have a test environment?](#do-we-have-a-test-environment-)
- [What should we reply to MakePayment() service call in case field "Confirmed" has value: TimeOut or Cancel?](#what-should-we-reply-to-makepayment---service-call-in-case-field--confirmed--has-value--timeout-or-cancel-)
- [Can you confirm that Vipps will send makePayment() service with corresponding values in field "Confirmed" if customer declines payment transaction or time-out occurs?](#can-you-confirm-that-vipps-will-send-makepayment---service-with-corresponding-values-in-field--confirmed--if-customer-declines-payment-transaction-or-time-out-occurs-)
- [Would it be correct to say that by responding to makePayment() we are informing Vipps about Authorization status and transactionStatusUpdate() informs Vipps about further actions with payment, like Capture/Void/Refund?](#would-it-be-correct-to-say-that-by-responding-to-makepayment---we-are-informing-vipps-about-authorization-status-and-transactionstatusupdate---informs-vipps-about-further-actions-with-payment--like-capture-void-refund-)
- [Is it possible to skip the landing page?](#is-it-possible-to-skip-the-landing-page-)
- [More questions or comments?](#more-questions-or-comments-)

# What is the card data structure?

An example of the card data structure.
CardData format: `{CardNumber:16-19},{ExpiryDate:4},{SessionId:1-32}`

CardNumber: Numeric
ExpiryDate: YYMM
SessionId: Guid

eg. `4925********4444,2212, 3854ba014e184e0d8ba259253f4advfa`

During onboarding the PSP needs to send a 2048-bit RSA public key for test and production. CardData will be encrypted using this key.
This data is then transformed into a 256 bytes OEAP cryptogram which is encoded as 344-characters base64 string.


## How can I view the card data?

Nets are our TSP (Token Service Provider) and all PSPs must exchange keys with Nets in order to decrypt card data.

Vipps only has a reference to the card data at the TSP, which we use to fetch the encrypted data to pass it through to the PSP for processing.
  

# Is there a unique PSP ID for all merchants?

No, the PSP ID is unique for the PSP and used for all merchants.

# Do we have a test environment?

Yes, please see: [The Vipps test environment (MT)](https://github.com/vippsas/vipps-developers#the-vipps-test-environment-mt).

# What should we reply to MakePayment() service call in case field "Confirmed" has value: TimeOut or Cancel?

Should it be: paymentInfo.status = "OK" or "FAIL"? If it is fail, what error code should be returned?

It is suggested that you send "FAIL" in case of failure either at your end or as in this case at our end.

If field "Confirmed" has value: `TimeOut` or `Cancel` -> `paymentInfo.status` = "FAIL"?

# Can you confirm that Vipps will send makePayment() service with corresponding values in field "Confirmed" if customer declines payment transaction or time-out occurs?

From the `makePayment` specification:

| Name | Type | Size | Optional | Values |
| ---- | ---- | ---- | -------- | ------ |
| confirmed	| String | 7 | No | YES/TIMEOUT/CANCEL |

# Would it be correct to say that by responding to makePayment() we are informing Vipps about Authorization status and transactionStatusUpdate() informs Vipps about further actions with payment, like Capture/Void/Refund?

Yes

# Is it possible to skip the landing page?

Skipping the landing page is reserved for special cases, where displaying it is not possible.
See the details in the 
[skip landing page section](https://github.com/vippsas/vipps-ecom-api/blob/master/vipps-ecom-api.md#skip-landing-page) 
in the API guide.

This feature has to be specially enabled by Vipps for eligible sale units: The sale units must be whitelisted by Vipps.
This functionality is typically used at physical points of sale, where there is no display available.

To request this feature, [contact Vipps](https://github.com/vippsas/vipps-developers/blob/master/contact.md) with a detailed description of why it is not possible to display the landing page.

# More questions or comments?

Please use GitHub's built-in functionality for
[issues](https://github.com/vippsas/vipps-invoice-api/issues) and
[pull requests](https://github.com/vippsas/vipps-invoice-api/pulls),
or [contact us](https://github.com/vippsas/vipps-developers/blob/master/contact.md).
