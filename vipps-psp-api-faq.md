# Vipps PSP API FAQ

API version: 2.

Document version 0.1.2.

_**IMPORTANT:**_ This document is a work in progress.

# About this API

Vipps Netthandel (eCommerce) via PSP offers functionality for payments on
websites and apps (P2M).

See the [guide](vipps-psp-api.md) for more details.

# What is the card data structure?

An example of the card data structure.
CardData format: `[Cardnumber],[expiryDate],[sessionId]`

CardNumber: Numeric
ExpiryDate: YYMM
SessionId: Guid

eg. `4925********4444,2212, 3854ba014e184e0d8ba259253f4advfa`

Service page encrypts data of the following format: `{CardNumber:16-19},{ExpiryDate:4},{SessionId:1-32}`.
This data is then transformed into a 256 bytes OEAP cryptogram which is encoded as 344-characters base64 string.
The cryptorgam is then included into the 652-bytes length XML response.

# Is there a unique PSP ID for all merchants?

No, the PSP ID is unique for the PSP and used for all merchants.

# Do we have a test environment?

Yes, please see: [The Vipps test environment (MT)](https://github.com/vippsas/vipps-developers#the-vipps-test-environment-mt).

# What should we reply to MakePayment() service call in case field "Confirmed" has value: No/TimeOut/Cancel?

Should it be: paymentInfo.status = "OK" or "FAIL"? If it is fail, what error code should be returned?

It is suggested that you send “FAIL” in case of failure either at your end or as in this case at our end.

If field "Confirmed" has value: `No` / `TimeOut` / `Cancel` -> `paymentInfo.status` = "FAIL"?

# Can you confirm that Vipps will send makePayment() service with corresponding values in field "Confirmed" if customer declines payment transaction or time-out occurs?

From the `makePayment` specification:

| Name | Type | Size | Optional | Values |
| ---- | ---- | ---- | -------- | ------ |
| confirmed	| String | 7 | No | YES/NO/TIMEOUT/CANCEL |

# Would it be correct to say that by responding to makePayment() we are informing Vipps about Authorization status and transactionStatusUpdate() informs Vipps about further actions with payment, like Capture/Void/Refund?

*TODO: Complete the remaining questions from Confluence*

# More questions or comments?

Please use GitHub's built-in functionality for
[issues](https://github.com/vippsas/vipps-invoice-api/issues) and
[pull requests](https://github.com/vippsas/vipps-invoice-api/pulls),
or [contact us](https://github.com/vippsas/vipps-developers/blob/master/contact.md).
