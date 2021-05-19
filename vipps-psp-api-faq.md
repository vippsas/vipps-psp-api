# Vipps PSP API FAQ

API version: 3.

Document version 1.1.0.

Vipps Netthandel (eCommerce) via PSP offers functionality for payments on
websites and apps (P2M).

See the [guide](vipps-psp-api.md) for more details.

# Table of Contents

- [How can I check if a merchant or sale unit is active?](#how-can-i-check-if-a-merchant-or-sale-unit-is-active)
- [How do we update a transaction?](#how-do-we-update-a-transaction)
- [What is a network token?](#what-is-a-network-token)
- [What is the card data structure?](#what-is-the-card-data-structure)
- [How can I view the card data?](#how-can-i-view-the-card-data)
- [Is there a unique PSP ID for all merchants?](#is-there-a-unique-psp-id-for-all-merchants)
- [Do we have a test environment?](#do-we-have-a-test-environment)
- [What should we reply to MakePayment() service call in case field "Confirmed" has value: TimeOut or Cancel?](#what-should-we-reply-to-makepayment-service-call-in-case-field-confirmed-has-value-timeout-or-cancel)
- [Can you confirm that Vipps will send makePayment() service with corresponding values in field "Confirmed" if customer declines payment transaction or time-out occurs?](#can-you-confirm-that-vipps-will-send-makepayment-service-with-corresponding-values-in-field-confirmed-if-customer-declines-payment-transaction-or-time-out-occurs)
- [Would it be correct to say that by responding to makePayment() we are informing Vipps about Authorization status and transactionStatusUpdate() informs Vipps about further actions with payment, like Capture/Void/Refund?](#would-it-be-correct-to-say-that-by-responding-to-makepayment-we-are-informing-vipps-about-authorization-status-and-transactionstatusupdate-informs-vipps-about-further-actions-with-payment-like-capturevoidrefund)
- [Is it possible to skip the landing page?](#is-it-possible-to-skip-the-landing-page)
- [Questions?](#questions)

# How can I check if a merchant or sale unit is active?

You can try to initiate payment, specifying the `Merchant-Serial-Number` header:
[`POST:/v3/psppayments/init/`](https://vippsas.github.io/vipps-psp-api/#/Vipps%20PSP%20API/initiatePaymentV3UsingPOST).

If the merchant or sale unit is not active, you will get an error.

See the eCom FAQ:
[`Why do I get errorCode 37 "Merchant not available or deactivated or blocked"?`](https://github.com/vippsas/vipps-ecom-api/blob/master/vipps-ecom-api-faq.md#why-do-i-get-errorcode-37-merchant-not-available-or-deactivated-or-blocked).

# How do we update a transaction?

Every operation done to a transaction after it has been processed should be
updated to our
[Update status endpoint](https://vippsas.github.io/vipps-psp-api/#/Vipps%20PSP%20API/updatestatusUsingPOST).
This includes partial refunds, captures etc. This is critical for support work
and user experience. This goes both for the single payment flow and for recurring payments.

Note that you do not need to send an update for the _reservation_ part of the
single payment flow, since that is handled by the synchronous response to the
Makepayment call. But you must send it for the Recurring flow.

# What is a network token?

A token is a "representative card number" for performing card payments, without using the actual
card details. The EMVco token number is not considered PCI DSS sensitive.

See the
[EMVco documentation](https://www.emvco.com/emv-technologies/payment-tokenisation/)
for more.

# What is the card data structure?

**Please note:** This is now deprecated. See:
[Differences from PSP API v2 to v3](https://github.com/vippsas/vipps-psp-api/blob/master/vipps-psp-api.md#differences-from-psp-api-v2-to-v3).

An example of the card data structure.
CardData format: `{CardNumber:16-19},{ExpiryDate:4},{SessionId:1-32}`

CardNumber: Numeric  
ExpiryDate: YYMM  
SessionId: Guid

eg. `4925********4444,2212, 3854ba014e184e0d8ba259253f4advfa`

During onboarding the PSP needs to send a 2048-bit RSA public key for test and production. CardData will be encrypted using this key.
This data is then transformed into a 256 bytes OEAP cryptogram which is encoded as 344-characters base64 string.

## How can I view the card data?

**Please note:** This is now deprecated. See:
[Differences from PSP API v2 to v3](https://github.com/vippsas/vipps-psp-api/blob/master/vipps-psp-api.md#differences-from-psp-api-v2-to-v3).

Nets are our TSP (Token Service Provider) and all PSPs must exchange keys with Nets in order to decrypt card data.

Vipps only has a reference to the card data at the TSP, which we use to fetch the encrypted data to pass it through to the PSP for processing.

# Is there a unique PSP ID for all merchants?

No, the PSP ID is unique for the PSP and used for all the PSP's merchants.

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

Yes.

# Is it possible to skip the landing page?

Skipping the landing page is reserved for special cases, where displaying it is not possible.
See the details in the
[skip landing page section](https://github.com/vippsas/vipps-ecom-api/blob/master/vipps-ecom-api.md#skip-landing-page)
in the eCom API guide.

See the eCom FAQ for more details:
[Is it possible to skip the landing page?](https://github.com/vippsas/vipps-ecom-api/blob/master/vipps-ecom-api-faq.md#is-it-possible-to-skip-the-landing-page)

# Questions?

We're always happy to help with code or other questions you might have!
Please create an [issue](https://github.com/vippsas/vipps-psp-api/issues),
a [pull request](https://github.com/vippsas/vipps-psp-api/pulls),
or [contact us](https://github.com/vippsas/vipps-developers/blob/master/contact.md).

Sign up for our [Technical newsletter for developers](https://github.com/vippsas/vipps-developers/tree/master/newsletters).
