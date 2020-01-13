# Vipps PSP API v2

[Vipps på Nett](https://www.vipps.no/bedrift/vipps-pa-nett)
(eCommerce) via PSP offers functionality for payments on
websites and apps (P2M). Vipps på Nett offers merchants functionality to
provide a solution where the end user only enters her Norwegian mobile number
to retrieve a payment request in the Vipps app.

In the Vipps app the end user selects a payment source to complete the payment
request. Vipps initiates the payment transaction on the selected source and
provide feedback to the PSP of the payment card selected.

The PSP processes the payment transaction and provides feedback to merchant
and Vipps of the payment transaction success or failure.

API version: 2.0

Document version 1.2.0.

API details: [Swagger UI](https://vippsas.github.io/vipps-psp-api/#/),
[swagger.yaml](https://raw.githubusercontent.com/vippsas/vipps-psp-api/master/docs/swagger.yaml),
[swagger.json](https://raw.githubusercontent.com/vippsas/vipps-psp-api/master/docs/swagger.json).

# Table of Contents

- [Differences from PSP API version 1](#differences-from-psp-api-version-1)
- [PSP payment sequence](#psp-payment-sequence)
- [API overview](#api-overview)
  * [Authentication](#authentication)
  * [Initiate payment](#initiate-payment)
    + [Skip landing page](#skip-landing-page)
    + [Payment confirmation](#payment-confirmation)
    + [makePaymentUrl](#makepaymenturl)
        - [Public key](#public-key)
        - [Card Data format](#card-data-format)
  * [Status Updates](#status-updates)
  * [Cancelling pending transactions](#cancelling-pending-transactions)
- [Example request](#example-request)
  * [Example response](#example-response)
  * [Idempotency](#idempotency)
- [PSP API implementation checklist](#psp-api-implementation-checklist)
- [Errors](#errors)
- [PSD2 Compliance and Secure Customer Authentication (SCA)](#psd2-compliance-and-secure-customer-authentication--sca-)
  * [3DSecure Fallback](#3dsecure-fallback)
- [Recurring payments](#recurring-payments)
  * [Scopes](#scopes)
  * [Initialize a recurring payment](#initialize-a-recurring-payment)
  * [The userToken](#the-usertoken)
  * [Make the next recurring payment](#make-the-next-recurring-payment)
- [HTTP responses](#http-responses)
  * [Error codes](#error-codes)
- [Questions](#questions)

# Differences from PSP API version 1

* Added support for redirection of user after payment completion in the Vipps app
* Added support for providing the `makePaymentUrl` in the initiate payment call
* Improved authorization of the `makePaymentUrl` call by adding the authorization header value
* Improved and more consistent parameter names in the API

# PSP payment sequence

![PSP API sequence diagram](images/psp-sequence-diagram.png)

**Important:** Some users may close Vipps immediately after seeing the payment confirmation,
therefore not being "redirected" back to the merchant. Because of this it is important for the
merchant and the PSP to _not_ base their transaction logic on the redirect alone.
For example: Check for "reserved" status with the PSP's API (not Vipps' API),
then do "capture" when the goods have been shipped/delivered.

# API overview

## Authentication

Every API call is authenticated and authorized based on the application
Authorization token (JWT) and subscription key
(`Ocp-Apim-Subscription-Key`).
The following headers are required to be there in every API request
to successfully authenticate every API call.

| Header Name | Header Value | Description |
| :-------------------------- | :--------------------------- | :--------------------------------------------- |
| Authorization | Bearer '<JWT access token>'' | type: Authorization token. Value: Access token is obtained from accessToken/get |
| Ocp-Apim-Subscription-Key | Base 64 encoded string | Subscription key for the product, available on poertal.vipps.no. |

## Initiate payment

A payment request is initiated by the PSP to the Vipps API after end user has
request to pay with Vipps. Vipps creates the payment and returns a link to
the Vipps landing page where end user can confirm the mobile number.
Once user has confirmed number the payment can be considered initiated.

### Skip landing page

*Only available for whitelisted sale units.*

If this property is set to `true`, it will cause a push notification to be sent
to the given phone number immediately, without loading the landing page.

If the sale unit is not whitelisted, this property is ignored.

If you need to be whitelisted, instructions for this can be found in the
[FAQ](https://github.com/vippsas/vipps-psp-api/blob/master/vipps-psp-api-faq.md#can-i-skip-the-landing-page).

### Payment confirmation

After payment initiation, Vipps sends push notification or redirects user to
the Vipps app. End user verifies the Vipps profile by logging in to the Vipps
app. In the Vipps app the end user can select payment source and confirm the amount.

### makePaymentUrl

Once the end user has confirmed the payment, Vipps shares the encrypted card
details with the PSP to the `makePaymentUrl`. The PSP uses the card details to
process the payment through the acquirer and responds to the `makePaymentUrl`
call with the payment request status. The Vipps user receives confirmation in
Vipps. Vipps redirects the end user to the `redirectUrl` provided during payment initiation.

#### Public key

The public key that `cardData` is encrypted with is provided by the PSP during onboarding. This key needs to be a 
RSA 2048 public key in PKCS#8 format. The corresponding private key is then used to decrypt `cardData`. Under is 
provided a basic suggestion for generating keys.

**For generating public/private key**
```console
$ ssh-keygen -t rsa -b 2048 -C "email@email.email"
```
**For converting to PKCS#8 format**
```console
$ ssh-keygen -m PKCS8 -e
```

#### Card Data format

The `cardData` is a string in the format
`{CardNumber:16-19},{ExpiryDate:4},{SessionId:1-32}`
that has been transformed into a 256 bytes OAEP cryptogram using the public
key provided by the PSP. The cryptogram is encoded as 344-characters base64 string.

## Status Updates

To provide a consistent end user experience it is important that Vipps is notified by changes to the payment status when it is captured, cancelled or refunded: [`POST:/v2/psppayments/updatestatus`](https://vippsas.github.io/vipps-psp-api/#/Vipps_PSP_API/updatestatusUsingPOST)

Vipps also provides an endpoint to check the payment status: [`POST:/v2/psppayments/{pspTransactionId}/details`](https://vippsas.github.io/vipps-psp-api/#/Vipps_PSP_API/getPSPPaymentDetailsUsingGET)

## Cancelling pending transactions

A user might go back to the PSP's checkout without logging in to the Vipps App,
or aborting it in the Vipps App.
Vipps's recommendation is that the PSP then cancels the transaction in their
backend and return an error code.

```json
| 85      | Response received too late            |
```

As seen in the error codes section of this graph.

So a typical flow would be:

1. The user selects Vipps in a checkout.
2. The user returns without completing in the app. (No `makePaymentUrl` request has been received by the PSP)
3. The PSP cancels the payment on their end, and restarts the checkout.
4. The user might end up going back to the Vipps app. If that happens and a `makePaymentUrl` request is sent,
the PSP responds with error code 85.

# Example request

```json
Authorization: makePaymentToken
{
  "pspTransactionId": "7686f7788898767977",
  "merchantSerialNumber": "123456",
  "cardData": "f0a29801b4#d4ff30e221fa2980ff30e2",
  "confirmed": "YES/TIMEOUT/CANCEL"
}
```

## Example response

```json
{
  "errorMessage": {
    "errorId": 82,
    "errorText": "Refused by Issuer"
  },
  "paymentInfo": {
    "pspTransactionId": "7686f7788898767977",
    "status": "FAIL"
  }
}
```

## Idempotency

All API requests in Vipps eCommerce can be retried without any side effects
by providing idempotent key in a header of  the request. For example, in
case the request fails because of network error it can safely be retried
with the same idempotent key. The idempotency key is generated by the merchant.

```
Request-Id: slvnwdcweofjwefweklfwelf
```

# PSP API implementation checklist

See the [Vipps PSP API Checklist](vipps-psp-api-checklist.md).

# Errors

The PSP should return the following errorIds and errorTexts when applicable:

| errorId | errorText                            | Description (should not be included in response)                                                                                  |
| ------- | ------------------------------------ | --------------------------------------------------------------------------------------------------------------------------------- |
| 71      | Invalid request                      | The request received from Vipps failed validation.                                                                                     |
| 72      | Different texts                      | For errors that are not in this list. Please include text describing the error.                                                   |
| 81      | No such issuer                       | The PSP were unable to identify the card issuer.                                                                                  |
| 82      | Refused by Issuer                    | Generic response for cards that were refused by issuer, without the PSP knowing why.                                              |
| 83      | Suspected fraud                      | Suspected fraud.                                                                                                                  |
| 84      | Exceeds withdrawal amount limit      | The amount exceeds an amount limit. Could be per transaction limit, or for a period.                                             |
| 85      | Response received too late            | A third party didn't respond in time for the makePayment or [Cancelling pending transactions](#Cancelling-pending-transactions)                                                      |
| 86      | Expired card                         | Expired card.                                                                                                                     |
| 87      | Invalid card number (no such number) | The card provided is invalid.                                                                                                     |
| 88      | Merchant does not allow credit cards | If the PSP supports disallowing credit cards.                                                                                     |
| 89      | Insufficient funds                   | Insufficient funds.                                                                                                               |
| 91      | Internal error                       | Unhandled or unknown exception.                                                                                                   |
| 92      | Unable to decrypt                    | The PSP was unable to decrypt `cardData` from the makePayment request.                                                            |
| 93      | Status from Vipps:CANCEL             | Response to Vipps sending `CANCEL`. Caused by customer cancelling the payment from the Vipps App, or in the Vipps landing page.    |
| 93      | Status from Vipps:TIMEOUT            | Response to Vipps sending `TIMEOUT`. Caused by customer not acting on the payment.                                                |
| 93      | Status from Vipps:NO                 | Response to Vipps sending `NO`. Something failed during payment approval. Often caused by failure to retrieve and send `cardData`.|



# PSD2 Compliance and Secure Customer Authentication (SCA)

In order to be allow for delegated SCA through the PSD2 directive every
transaction needs to be marked with Vipps's WalletId.
Vipps's WalletId is A62.

## 3DSecure Fallback

In case of a soft decline (when the3 issuer requires 3DS), the PSP will host a
3DSecure session and needs to provide the URL to Vipps.

![PSP API sequence diagram](diagrams/3DSFallbackFlow.png)

The format of the `MakePaymentRequest` response provided by the PSP in case of
a soft decline:

```json
{
  "errorMessage": {
    "errorId": 1036,
    "errorText": "Soft Decline"
  },
  "paymentInfo": {
    "pspTransactionId": "7686f7788898767977",
    "status": "SOFT_DECLINE",
    "url3dSecure": "https://psp.com/3ds/123123"
  }
}
```

The Vipps App will then open the URL in an iframe, letting the user complete
the 3DSecure flow. The PSP will have to host and retrieve any necessary data
from the session. Once the session is completed it will have to finish with a
redirect to `https://www.vipps.no/mobileintercept`, where upon the app
will close the iframe. Vipps will then resend the `makePaymentUrl` request.

The status in the response to this `makePaymentUrl` should never be
`SOFT_DECLINE`, only `FAIL` or `OK`.
Once the status is returned it will be displayed to the user as normal in the app.

```
Final Redirect URL: https://www.vipps.no/mobileintercept
```

```json
Authorization: makePaymentToken
{
  "pspTransactionId": "7686f7788898767977",
  "merchantSerialNumber": "123456",
  "cardData": "f0a29801b4#d4ff30e221fa2980ff30e2",
  "confirmed": "YES/TIMEOUT/CANCEL"
}
```

# Recurring payments

The PSP API supports recurring payments, allowing the PSP to
perform recurring payments through Vipps, while retaining full transactional
control. This has been built as an extension to the existing PSP v2 API, and no
existing integrations will be affected, other than the possibility to initialize
and preform recurring payments.

## Scopes

As of now, there are is one possible way to perform a recurring payment: `subscription`.
This is referred to as the `scope` of the recurring agreement.
Only one `scope` can be used at a time, and it's not possible to change the scope of an agreement.

Subscription-based payments are created as a consent to an agreement that allows
the PSP to withdraw money on unknown intervals. This implies that the user won't
have to accept the payment on each occasion, only the first one when consenting
to the agreement. An example of this could be a subscription to a music streaming
service.

## Initialize a recurring payment

Initializing a recurring payment works in the same way as a non-recurring payment,
but with the inclusion of a `scope` and `agreementURL` in the init call.

The `scope` can at the time be set to  `subscription`.
The `agreementURL` should be a link to where the user can click to manage the agreement.

To start the initialization, create a standard /init call with the addition of
the required fields. If you want to initialize a recurring `subscription`,
the init request body could look like this.

```json
{
  "pspRedirectUrl": "https://example.com/yourRedirectUrl",
  "amount": 1337,
  "makePaymentToken": "tokenGoesHerenco3rt8y34obcwierunnw3oirtycbvwo",
  "makePaymentUrl": "https://example.com/yourCallbackEndpointUrl",
  "currency": "NOK",
  "merchantOrderId": "123123123",
  "isApp": false,
  "pspTransactionId": "{{psptransactionid}}",
  "paymentText": "Order id: 213213",
  "scope": "subscription",
  "agreementURL": "https://example.com/linkToTheAgreement"
}
```

In the same way as a normal, non-recurring PSP v2 payment, the PSP will receive a
[`POST:makePaymentURL`](https://vippsas.github.io/vipps-psp-api/#/Endpoints%20required%20by%20Vipps%20from%20the%20PSP/makePaymentSwaggerUsingPOST)
callback.
In the body of this callback, you will now also find a `userToken`.

## The userToken

The user token is a token generated when the user has given a consent. This
token is provided to the PSP in the makePayment callback when initializing or
preforming a recurring payment.

A `userToken` contains the `scope` of the consent, in the claim named `scope`.
The token also includes various useful information:

```json
{
  "sub" // agreement_id
  "aud" // audience (usually something like "https://vipps.no")
  "iat" // issue_day_time
  "scope"
}
```

## Make the next recurring payment

After initialisation, the next payment can be made by passing your `userToken`
to the [`POST:/v2/psppayments/payments`](https://vippsas.github.io/vipps-psp-api/#/Vipps%20PSP%20API/processPaymentOnToken)
endpoint as a header with the name `userToken`.

```json
HEADER: "
        Authorization: Bearer eyJhbGciOiJSUzI1NiIsInR5c<snip>
        Ocp-Apim-Subscription-Key: c1b1a8846ec56d09db39bd3b5403bf9
        PSP-ID: C948DFD1546347568874C4DDC93A2E3C
        Meerchant-Serial-Number: 123456
        UserToken: eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9
        "
{
  "pspTransactionId": "7686f7788898767977",
  "merchantOrderId": "8874C4DDC93A2E3C",
  "amount": 39900,
  "currency": "NOK",
  "makePaymentUrl": "https://example.com/yourPaymentUrl",
  "makePaymentToken": "ynuiu",
  "paymentText": "Description of payment"
}
```

# HTTP responses

This API returns the following HTTP statuses in the responses:

| HTTP status         | Description                                 |
| ------------------- | ------------------------------------------- |
| `200 OK`            | Request successful                          |
| `400 Bad Request`   | Invalid request, see the error for details  |
| `401 Unauthorized`  | Invalid credentials                         |
| `403 Forbidden`     | Authentication ok, but credentials lacks authorization  |
| `404 Not Found`     | The resource was not found  |
| `500 Server Error`  | An internal Vipps problem.                  |

## Error codes

| errorCode        | errorMessage                               |
| ---------------- | ------------------------------------------ |
| `21`             | Merchant not available or active           |
| `42`             | Invalid payment model type                 |
| `51`             | Invalid request                            |
| `99`             | PSP Transaction id already exists in Vipps |
| `51`             | Invalid pspRedirectUrl                     |
| `99`             | OrderId already exists                     |
| `amount`         | amount.less.than.one                       |
| `currency`       | transaction.currency.invalid               |
| `makePaymentUrl` | Invalid makePaymentUrl                     |

# Questions

We're always happy to help with code or other questions you might have!
Please create an [issue](https://github.com/vippsas/vipps-psp-api/issues),
a [pull request](https://github.com/vippsas/vipps-psp-api/pulls),
or [contact us](https://github.com/vippsas/vipps-developers/blob/master/contact.md).

# Proposals

## Recurring 3DS Update Card

The following is a proposal for triggering 3DS when a user changes the card attached to a PSP recurring agreement in the Vipps app.

A new optional `updateCardUrl` property would be added to the initate call. This callback will be triggered when the user changes card with the following request;

```json
HEADER: "
        PSP-ID: C948DFD1546347568874C4DDC93A2E3C
        Meerchant-Serial-Number: 123456
        UserToken: eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9
        "
{
  "agreementId": "7686f7788898767977",
  "cardData": "f0a29801b4#d4ff30e221fa2980ff30e2"
}
```

To which the PSP should respond with a `require3DS` boolean flag which, if true, will cause a 3DS session to be hosted in the Vipps app.


```json
{
  "require3DS": true,
  "url3dSecure": "https://psp.com/3ds/123123"
}
```

The Vipps app will the host a 3DS session from the PSP, once this session is completed 
the PSP should redirect to `https://www.vipps.no/mobileintercept`.

Once the user completes the 3DS session the `updateCardUrl` will be called again.
The PSP should then only approve or deny the request.
