# Vipps PSP API v3

[Vipps på Nett](https://www.vipps.no/bedrift/vipps-pa-nett)
(eCommerce) via PSP offers functionality for payments
where the end user simply enters their Norwegian mobile number
to complete a payment in the Vipps app.

In the Vipps app, the user selects a payment card (a payment source).
Vipps then gives the PSP a token for that card so the PSP can process the payment.

The PSP processes the payment, provides feedback to merchant,
and sends Vipps an update of the payment transaction success or failure.
The user is shown the correct status for the payment in Vipps.

Since January 1, 2021, all PSPs must be able to process network tokens.
See the
[EMVco documentation](https://www.emvco.com/emv-technologies/payment-tokenisation/)
for more information.

**Important:** Cards that were added to Vipps after January 1, 2021 are only enrolled with
Vipps' new PSP. That means Vipps is only able to provide a token for those
cards, not the card details. If you are not able to process tokens, you should respond with
`HTTP 403 Forbidden`, as that gives the best (least bad) customer experience in Vipps.

These OpenAPI representations may be useful to get a quick overview:
* [Swagger](https://vippsas.github.io/vipps-psp-api)
* [ReDoc](https://vippsas.github.io/vipps-psp-api/redoc.html)

API details: [Github Repository](https://github.com/vippsas/vipps-psp-api),
[Checklist](vipps-psp-api-checklist.md),
[FAQ](vipps-psp-api-faq.md),
[Postman Guide](vipps-psp-postman.md)

API version: 3.0

Document version 3.1.14.

# Table of Contents

- [PSP payment sequence](#psp-payment-sequence)
- [Differences from previous versions](#differences-from-previous-versions)
  - [Differences from PSP API v2 to v3](#differences-from-psp-api-v2-to-v3)
  - [Differences from PSP API v1 to v2](#differences-from-psp-api-v1-to-v2)
- [API overview](#api-overview)
  - [Authentication](#authentication)
  - [Initiate payment](#initiate-payment)
    - [Skip landing page](#skip-landing-page)
    - [Payment confirmation](#payment-confirmation)
    - [makePaymentUrl](#makepaymenturl)
  - [EMVCo token processing](#emvco-token-processing)
    - [Scheme specific details](#scheme-specific-details)
      - [Visa](#visa)
      - [Mastercard](#mastercard)
    - [Token Requestor Ids](#token-requestor-ids)
    - [3DSecure and Network tokens](#3dsecure-and-network-tokens)
    - [Magic Numbers for EMVCo Tokens](#magic-numbers-for-emvco-tokens)
  - [Status Updates](#status-updates)
    - [Batch processing of status updates](#batch-processing-of-status-updates)
  - [Cancelling pending transactions](#cancelling-pending-transactions)
- [Example request](#example-request)
  - [Example response](#example-response)
  - [Idempotency](#idempotency)
- [PSP API implementation checklist](#psp-api-implementation-checklist)
- [Errors](#errors)
- [PSD2 Compliance and Secure Customer Authentication (SCA)](#psd2-compliance-and-secure-customer-authentication-sca)
  - [3DSecure Fallback](#3dsecure-fallback)
- [Recurring payments](#recurring-payments)
  - [Scopes](#scopes)
  - [Initialize a recurring payment](#initialize-a-recurring-payment)
  - [The userToken](#the-usertoken)
  - [Make the next recurring payment](#make-the-next-recurring-payment)
- [URL Validation](#url-validation)
- [HTTP responses](#http-responses)
  - [Error codes](#error-codes)
- [Recommendations regarding handling redirects](#recommendations-regarding-handling-redirects)
- [PSP Signup API](#psp-signup-api)
- [Get all merchants](#get-all-merchants)
- [Get information about a specific merchant](#get-information-about-a-specific-merchant)
- [Create a new merchant sale unit](#create-a-new-merchant-sale-unit)
- [Update an existing merchant sale unit](#update-an-existing-merchant-sale-unit)
- [Proposals](#proposals)
  - [Recurring 3DS Update Card](#recurring-3ds-update-card)
  - [Questions?](#questions-1)


# Differences from previous versions

## Differences from PSP API v2 to v3

The PSP API v3 adds functionality for network tokens and allows PSPs to use the API to
obtain tokens from Vipps, not the actual card details.

Additionally `$.makePaymentRequest.confirmed` has been renamed to `$.makePaymentRequest.paymentState`

Values for this enum have changed accordingly:
| Old Value | New Value |
|-----------|-----------|
| Yes | ACCEPTED |
|TIMEOUT| TIMEOUT|
|CANCEL| USER_CANCEL|
|NO| USER_CANCEL|

## Differences from PSP API v1 to v2

* Added support for redirection of user after payment completion in the Vipps app
* Added support for providing the `makePaymentUrl` in the initiate payment call
* Improved authorization of the `makePaymentUrl` call by adding the `Authorization` header value
* Improved and more consistent parameter names in the API

# PSP payment sequence

![PSP API sequence diagram](images/psp-sequence-diagram.png)

**Important:** Some users may close Vipps immediately after seeing the payment confirmation,
therefore not being "redirected" back to the merchant. Because of this, it is important for the
merchant and the PSP to _not_ base their transaction logic on the user reaching the `pspRedirectUrl`.

For example: Check for "reserved" status with the PSP's API (not Vipps' API),
then do "capture" when the goods have been shipped/delivered.

# API overview

## Authentication

All Vipps API calls are authenticated and authorized with an access token
(JWT bearer token) and an API subscription key:

| Header Name | Header Value | Description |
| ----------- | ------------ | ----------- |
| `Authorization` | `Bearer <JWT access token>` | Type: Authorization token. This is available on [portal.vipps.no](https://portal.vipps.no). |
| `Ocp-Apim-Subscription-Key` | Base 64 encoded string | The subscription key for this API. This is available on [portal.vipps.no](https://portal.vipps.no). |

For more information about how to obtain an access token and all details around this, please see:
[Quick overview of how to make an API call](https://github.com/vippsas/vipps-developers/blob/master/vipps-getting-started.md#quick-overview-of-how-to-make-an-api-call)
in the
[Getting started guide](https://github.com/vippsas/vipps-developers/blob/master/vipps-getting-started.md).

## Initiate payment

When a user chooses Vipps as the payment method, a payment request is initiated
by the PSP to the Vipps API. Vipps creates the payment and returns a link that does the following:

* For users on a mobile device, the Vipps app is opened.
* For other users, the Vipps landing page is opened in a browser.  The user can then enter their phone number and continue in the Vipps app on their phone.

The user then confirms the payment in Vipps.

**Important:** Never show the Vipps landing page inside an iframe.
That will make it impossible for the user to reliably be redirected back to the
merchant's website and will result in a lower success rate.
In general, any "optimization" of the payment flow may break the Vipps payment flow - if not today, then later.

### Skip landing page

*Only available for whitelisted sale units.*

If `skipLandingpage` is set to `true`, Vipps will _not_ display the landing page,
but instead send a push notification to the phone number immediately.
This must only be used when the payment does not start on the user's device and
when there is no customer-facing display available.

If the sale unit is not whitelisted for `skipLandingpage`, an attempt to use it
will result in an error.

If you need to be whitelisted, instructions for this can be found in the
[FAQ](https://github.com/vippsas/vipps-psp-api/blob/master/vipps-psp-api-faq.md#can-i-skip-the-landing-page).

### Payment confirmation

After the payment initiation, Vipps sends a push notification or redirects the user to
the Vipps app. The user logs in, selects payment source, and confirms the payment.

### makePaymentUrl

Once the end user has confirmed the payment, Vipps shares the network token
with the PSP by `POST`-ing to the `makePaymentUrl`:
[`POST:makePaymentUrl`](https://vippsas.github.io/vipps-psp-api/#/Endpoints%20required%20by%20Vipps%20from%20the%20PSP/makePaymentV3UsingPOST).

The PSP uses the card token to process the payment through the acquirer.
This is the PSP's responsibility.
Vipps is not involved in the actual payment, Vipps only provides the
PSP with the card token.

The PSP then sends Vipps an update on the status of the payment:
[`POST:/v3/psppayments/updatestatus`](https://vippsas.github.io/vipps-psp-api/#/Vipps%20PSP%20API/updatestatusUsingPOST)

It's important that the PSP sends this update, so the user can see the
correct status of the payment in Vipps.
Without a status update from the PSP, the user will see an incorrect status.

Payment updates are processed in batches.
See: [Batch processing of status updates](#batch-processing-of-status-updates).

The user receives confirmation of the payment in Vipps.
Vipps redirects the end user to the `redirectUrl` provided during payment initiation.

**Please note:**
* The `makePaymentUrl` has a timeout of 15 seconds. If no response is received within this period
  Vipps will mark the transaction as failed. This is a known issue with the current API.
  Future improvements to address this issue are planned.
* Some users may close Vipps immediately after seeing the payment confirmation,
  therefore not being "redirected" back to the merchant. Because of this, it is important for the
  merchant and the PSP to _not_ base their transaction logic on the user reaching the `pspRedirectUrl`.

## EMVCo Token processing

In order to give the best possible payment experience, the Vipps PSP API uses
EMVCO token based processing.

The solution requires the PSP to have support for EMVCO token processing.

The `makePaymentUrl` call to the PSP sends the network token
and cryptogram in the following format. This is referred to as the `makePayment` request.

```
Authorization: makePaymentToken
{
  "pspTransactionId": "7686f7788898767977",
  "merchantSerialNumber": "123456",
  "paymentState": "ACCEPTED/TIMEOUT/USER_CANCEL",
  "paymentInstrument": "TOKEN",
  "binNumber": "492556",
  "cardData": null,
  "networkToken": {
    "number": "12345678901234",
    "expiryMonth": "12",
    "cryptogram": "aFgdgjdkfgjdFDF=",
    "tokenType": "VISA",
    "expiryYear": "2025",
    "eci": "07"
  }
}
```

In certain cases, a legacy encrypted card request is sent in the following format.

**Important:** This is only available for existing PSPs that are not able to handle
tokens, and there are _very_ few left.

```
Authorization: makePaymentToken
{
  "pspTransactionId": "7686f7788898767977",
  "merchantSerialNumber": "123456",
  "paymentState": "ACCEPTED/TIMEOUT/USER_CANCEL",
  "paymentInstrument" : "ENCRYPTEDCARD",
  "binNumber": "492556",
  "cardData": "f0a29801b4#d4ff30e221fa2980ff30e2",
  "networkToken": null
}
```

Where:
* `paymentInstrument` is as an indicator that can be used to differentiate the two alternatives.
* `networkToken` is the Network token of the card, up to 16-19 digits. A full replacement of the PAN.

### Scheme specific details

#### Visa

Visa tokens must be processed with the acquirer submitting the TAVV cryptogram
in field 126.8. The cryptogram received with the Network Token will contain the
information for Delegated Authentication (DA) and the SCA factors used.

The Visa Token Service will during detokenization populate a flag for DA in field
34 to issuers and the Vipps TRID in field 123 Usage 2 Tag 03. In this way,
Issuers recognise Vipps originated transactions and will not soft decline for
DS step-up unless the issuing bank has opted out of the Visa D-SCA program.
The expected ECI value for VISA requests is ECI-07.

#### Mastercard

A MasterCard transaction should be processed as an ecom-token in accordance with
the acquirers instructions from Mastercard. Mastercard adds the Token Requestor ID
(TRID) to the authorization message. It will always be available in DE48, SE33, SF6.

Vipps is working on providing delegated authentication with MasterCard.
As the time of writing, you should expect soft declines on all MasterCard transactions.
Vipps initially returns ECI-06 for Mastercard, this should be handled as "no 3DS,
no challenge requested" in accordance with your acquirer.

### Token Requestor Ids

For Visa/Mastercard, the Token Requestor ID (TRID) is an eleven digit number.
It is added by the scheme in the processing of the payments.

### 3DSecure and Network tokens

In order to start a 3DS session, simply use the Network Token Number instead of
the regular PAN. The scheme Directory Server maps the Network Token to the
underlying PAN before it requests the challenge session from the Issuing Bank's ACS.
CVC is not required in order to perform the 3DS session with a network token.

Once the 3DS CAVV cryptogram is acquired from the 3DS session, both the CAVV and
the token cryptogram must be submitted in the authorization request in the fields
specified by the acquirer. This is necessary to perform a valid authorization.

### Magic Numbers for EMVCo Tokens

Any request to the Vipps PSP API will return a Visa Token. However, this can be changed
by setting the amount in the init request. No matter what is selected in the app,
the Token returned in the MakePayment request will be:

| Amount Value | Token Number | Expiry | Cryptogram
|-----------|-----------|-----|-----|
| 22.00 | 5226603115488031 | 05/25 | AlhlvxmN2ZKuAAESNFZ4GoABFA==
| 31.00 | Emulates Card not eligible |
| 32.00 | 4111111111111111 | 03/30 | uxToh3Ep6gsR8AAkvZALN19Iz34=
| 42.00 | 4895370013193500 | 03/30 | AlhlvxmN2ZKuAAESNFZ4GoABFA==
| 43.00 | 5226603115488031 | 03/30 | AlhlvxmN2ZKuAAESNFZ4GoABFA==
| 44.00 | 4895370012792682 | 12/22 | AgAAAAAAAIR8CQrXSohbQAAAAAA=
| 51.00 | 4268270087302871 | 09/24 | AgAAAAAAAIR8CQrXSohbQAAAAAA=
| 52.00 | 5413330089010442 | 12/25 | AgAAAAAAAIR8CQrXSohbQAAAAAA=
| All other amounts | 4895370013193500 | 05/25 | AlhlvxmN2ZKuAAESNFZ4GoABFA==

## Status Updates

To provide a consistent end user experience, it is important that Vipps is
notified by changes to the payment status when it is captured, cancelled, or refunded:
[`POST:/v3/psppayments/updatestatus`](https://vippsas.github.io/vipps-psp-api/#/Vipps_PSP_API/updatestatusUsingPOST)

Vipps also provides an endpoint allowing you to check the payment status:
[`GET:/v3/psppayments/{pspTransactionId}/details`](https://vippsas.github.io/vipps-psp-api/#/Vipps_PSP_API/getPSPPaymentDetailsUsingGET)

For customers upgrading from the PSP API v1: It is ok to call `updateStatus`
with the v3 API on payments done with the v1 API.

### Batch processing of status updates

Requests to
[`POST:/v3/psppayments/updatestatus`](https://vippsas.github.io/vipps-psp-api/#/Vipps_PSP_API/updatestatusUsingPOST)
receive a `HTTP 200 OK` response if the JSON payload was valid.
The actual _processing_ of the data is done as a batch.

The reason for this is that some PSPs send Vipps all their status updates once
a day. Based on this, we created a service that can handle thousands of
transaction updates in one operation.

To avoid all of these updates being written to our database at once, with the
resulting performance hit, we chose to defer these into a nightly batch job.

Because of this, the updated status is only visible in Vipps the next day.

**Please note:** The batch update is _not_ run in the test environment, as
there are some technical details preventing it from being run in the same way
as in the production environment. The status of PSP payments will therefore
always be in the initiated state in the test environment.

## Cancelling pending transactions

A user might return to the PSP's checkout without logging in to the Vipps App,
or they might abort the transaction in the Vipps App.
Vipps's recommendation is that the PSP then cancels the transaction in their
backend and returns an error code.

```json
| 85      | Response received too late            |
```

See: [Error codes](#error-codes).

A typical flow would be:

1. The user selects Vipps as payment method.
2. The user returns without completing in the app (no `makePaymentUrl` request has been received by the PSP).
3. The PSP cancels the payment on their end and restarts the checkout.
4. The user might end up going back to the Vipps app. If that happens and a `makePaymentUrl` request is sent,
   the PSP responds with error code 85.

# Example request

```json
Authorization: makePaymentToken
{
  "pspTransactionId": "7686f7788898767977",
  "merchantSerialNumber": "123456",
  "cardData": "f0a29801b4#d4ff30e221fa2980ff30e2",
  "paymentState": "ACCEPTED/TIMEOUT/USER_CANCEL"
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

Many API requests to Vipps APIs can be retried without any side effects
by providing `Request-Id`(or `Idempotency-key`) in the header of the
request. For example, in case the request fails because of network error, it can
safely be retried with the same `Request-Id` key. The `Request-Id` key must be
generated by the merchant.

For example:

```
Request-Id: slvnwdcweofjwefweklfwelf
```

# PSP API implementation checklist

See: [Vipps PSP API Checklist](vipps-psp-api-checklist.md).

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
| 85      | Response received too late            | A third party didn't respond in time for the makePayment or [Cancelling pending transactions](#Cancelling-pending-transactions).                                                      |
| 86      | Expired card                         | Expired card.                                                                                                                     |
| 87      | Invalid card number (no such number) | The card provided is invalid.                                                                                                     |
| 88      | Merchant does not allow credit cards | If the PSP supports disallowing credit cards.                                                                                     |
| 89      | Insufficient funds                   | Insufficient funds.                                                                                                               |
| 91      | Internal error                       | Unhandled or unknown exception.                                                                                                   |
| 92      | Unable to decrypt                    | The PSP was unable to decrypt `cardData` from the makePayment request.                                                            |
| 93      | Status from Vipps:CANCEL             | Response to Vipps sending `CANCEL`. Caused by customer cancelling the payment from the Vipps App, or in the Vipps landing page.    |
| 93      | Status from Vipps:TIMEOUT            | Response to Vipps sending `TIMEOUT`. Caused by customer not acting on the payment.                                                |
| 93      | Status from Vipps:NO                 | Response to Vipps sending `NO`. Something failed during payment approval. Often caused by failure to retrieve and send `cardData`.|
| 94      | Unhandled soft decline               | Response to a Soft Decline from the bank that could not be processed or forwarded to the Vipps App.                               |

# PSD2 Compliance and Secure Customer Authentication (SCA)

## 3DSecure Fallback

In case of a soft decline (when the issuer requires 3DS), the PSP must host a
3DSecure session and must provide the URL to Vipps.

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

The Vipps App will then open the URL in a web view, letting the user complete
the 3DSecure flow. The PSP must host and retrieve any necessary data
from the session. Once the session is completed, the PSP must finish with a
redirect according to the `Operations.url` object sent in the initial
`makePayment` request, where upon the app will close the iframe. Vipps will
then resend the `makePayment` request.

For example, if a 3DSecure session succeeds, you should redirect to the
`Operations.url` with the operation `3dssuccess`.

```
operations\":[{\"url\":\"https://www.example.com/?transactionId=xxxxx&responsecode=OK\",\"operation\":\"3dssuccess\"}"
```

Note that the `responseCode` query parameter is critical.

When Vipps sends the status in the response to the second `makePayment`
request, it should never be `SOFT_DECLINE`, only `FAIL` or `OK`.
Once the status is returned, it will be displayed to the user as normal in the app.

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

**Please note:** We are currently not onboarding new PSP partners to
Passthrough Recurring Payments due to challenges with user experience under PSD2.
We will update this section when there is new information.

The PSP API supports recurring payments, allowing the PSP to
perform recurring payments through Vipps, while retaining full transactional
control. This has been built as an extension to the existing PSP v3 API, and no
existing integrations will be affected, other than the possibility to initialize
and preform recurring payments.

## Scope

As of now, there is one possible way to perform a recurring payment: `psp_subscription`.
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

At that time, the `scope` can be set to  `psp_subscription`.
The `agreementURL` should be a link to where the user can click to manage the agreement.

To start the initialization, create a standard /init call with the addition of
the required fields. If you want to initialize a recurring `psp_subscription`,
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
  "scope": "psp_subscription",
  "agreementURL": "https://example.com/linkToTheAgreement"
}
```

In the same way as a normal, non-recurring PSP v3 payment, the PSP will receive a
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
to the [`POST:/v3/psppayments/payments`](https://vippsas.github.io/vipps-psp-api/#/Vipps%20PSP%20API/processPaymentOnToken)
endpoint as a header with the name `User-Token`.

```json
HEADER: "
        Authorization: Bearer eyJhbGciOiJSUzI1NiIsInR5c<snip>
        Ocp-Apim-Subscription-Key: c1b1a8846ec56d09db39bd3b5403bf9
        PSP-ID: C948DFD1546347568874C4DDC93A2E3C
        Merchant-Serial-Number: 123456
        User-Token: eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9
        "
{
  "pspTransactionId": "7686f7788898767977",
  "merchantOrderId": "8874C4DDC93A2E3C",
  "amount": 39900,
  "currency": "NOK",
  "paymentText": "Description of payment"
}
```

Response with network token:
```json
{
  "pspTransactionId": "7686f7788898767977",
  "merchantOrderId": "8874C4DDC93A2E3C",
  "paymentInstrument": "TOKEN",
  "networkToken": {
    "number": "12345678901234",
    "expiryMonth": "12",
    "cryptogram": "aFgdgjdkfgjdFDF=",
    "tokenType": "MASTERCARD",
    "expiryYear": "2025",
    "eci": "07"
  }
}
```

Response with encryptedCard:
```json
{
  "pspTransactionId": "7686f7788898767977",
  "merchantOrderId": "8874C4DDC93A2E3C",
  "paymentInstrument": "ENCRYPTEDCARD",
  "cardData": "f0a29801b4#d4ff30e221fa2980ff30e2",
}
```

Once the card data is received from
[`POST:/v3/psppayments/payments`](https://vippsas.github.io/vipps-psp-api/#/Vipps%20PSP%20API/processPaymentOnToken)
and the payment has been processed, the PSP must call
[`POST:/v3/psppayments/updatestatus`](https://vippsas.github.io/vipps-psp-api/#/Vipps_PSP_API/updatestatusUsingPOST)
to notify Vipps of the status. In this context, updateStatus accepts a `RESERVED` status.
If the status for the previous payment has not been received, the agreement will be locked from processing future payments until the update is received.
If a recurring payment fails, you should call
[`POST:/v3/psppayments/updatestatus`](https://vippsas.github.io/vipps-psp-api/#/Vipps_PSP_API/updatestatusUsingPOST)
with `operationStatus: FAILED` set in the body.

# URL Validation

All URLs in Vipps eCommerce API are validated with the
[Apache Commons UrlValidator](https://commons.apache.org/proper/commons-validator/apidocs/org/apache/commons/validator/routines/UrlValidator.html).

If `isApp` is true, any `pspRedirectUrl` is not validated with Apache Commons UrlValidator,
as the app-switch URL may be something like `vipps://`, which is not a valid URL.

The endpoints required by Vipps must be publicly available.

URLs that start with `https://localhost` will be rejected. If you want to use
localhost as fallback, please use `http://127.0.0.1`.
It is not possible to use `https://localhost` or
`http://127.0.0.1` for the callback, as the Vipps backend would then call itself.

Here is a simple Java class suitable for testing URLs,
using the dummy URL `https://example.com/vipps/fallback-result-page/acme-shop-123-order123abc`:

```java
import org.apache.commons.validator.routines.UrlValidator;

public class UrlValidate {
 public static void main(String[] args) {
  UrlValidator urlValidator = new UrlValidator();

  if (urlValidator.isValid("https://example.com/vipps/fallback-result-page/acme-shop-123-order123abc")) {
   System.out.println("URL is valid");
  } else {
   System.out.println("URL is invalid");
  }
 }
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
| `500 Server Error`  | An internal Vipps problem                  |

## Error codes

| errorCode        | errorMessage                               | Description |
| ---------------- | ------------------------------------------ | ---|
| `21`             | Merchant not available or active           | |
| `42`             | Invalid payment model type                 | |
| `44`             | PSP Transaction id already exists          | |
| `51`             | Invalid request                            | |
| `51`             | Invalid pspRedirectUrl                     | |
| `99`             | OrderId already exists                     | |
| `amount`         | amount.less.than.one                       | |
| `currency`       | transaction.currency.invalid               | |
| `makePaymentUrl` | Invalid makePaymentUrl                     | |
| `121`        | Agreement not found | The agreement does not exist |
| `122`        | Agreement not active | The agreement has been deactivated via the Delete agreement endpoint |
| `123`        | Connected payment source not active for Agreement | The connected card is no longer active or valid |
| `124`        | No Network token available for this Agreement | The connected card does not have a active network token available |

# Recommendations regarding handling redirects

Since Vipps is a mobile entity, the amount of control Vipps has over the redirect back to the merchant after the purchase is completed is limited. A merchant must not assume that Vipps will redirect to the exact same session and for example rely entirely on cookies in order to handle the redirect event. For example the redirect could happen to another browser.

Examples of some, but not all, factors outside of Vipps control.
- Configurations set by the OS itself, for example the default browser.
- User configurations of browsers.
- Users closing app immediately upon purchase.

Therefore, Vipps recommends having a stateless approach in the site that is supposed to be the end session. An example would be a polling-based result handling from a value in the redirect url.

For demonstration purposes, an example that should be handled is:

1. User starts is in web session in a Chrome Browser.
1. A Vipps purchase is started, a redirect URL is defined by the Merchant.
1. The user completes the purchase.
1. The Vipps app redirects the user.
1. The OS defaults to a Safari Browser for the redirect.
1. The merchant handles the redirect without the customer noticing any discrepancies from the browser switch.

# PSP Signup API

The Vipps PSP Signup API allows PSPs to onboard and manage their merchants.

This API is the only way to sign up non-Norwegian merchants.

The following are the screens in the Vipps app, where the information about the merchant that was provided by the PSP is rendered to the end user.

![Payment Screen](./docs/signup/payment.png)

![Receipt Screen](./docs/signup/receipt.png)

Some details of information shown in the screenshots:

| Item               | Description                                           |
| ------------------ | ----------------------------------------------------- |
| Merchant           | The name of the merchant's sale unit                  |
| #23412             | Merchant serial number, the sale unit's id            |
| Merchant AS        | The name of the merchant                              |
| Product name       | The name of the product being paid for                |
| Order ID / Description | The orderId, provided by the merchant             |
| Transaction ID     | The internal Vipps id for the transaction             |
| butikken.no        | Merchant website                                      |

The API specification is available here:
* [Swagger UI](https://vippsas.github.io/vipps-psp-api/signup/)
* [Swagger source](https://github.com/vippsas/vipps-psp-api/blob/master/docs/signup/swagger.yaml)
* [Postman collection](tools/vipps-psp-v3-api-postman-collection.json)
* [Postman environment](tools/vipps-psp-v3-api-postman-environment.json)

A PSP can use their existing keys to access this APIs. They can perform the following:
- List one or all merchants under them
- Create a new merchant under them
- Update an existing merchant

## Get all merchants

For a json response showing all the merchants and their information, send [`GET:/v1/merchants`](https://vippsas.github.io/vipps-psp-api/signup/#/Merchant/getMerchants).

## Get information about a specific merchant

For information about a specific merchant, send 
[`GET:/v1/merchants/:merchantSerialNumber`](https://vippsas.github.io/vipps-psp-api/signup/#/Merchant/getMerchant). Supply the MSN for a merchant in your list of merchants.

## Create a new merchant sale unit

To create new merchant sale unit, send [`POST:/v1/merchants`](https://vippsas.github.io/vipps-psp-api/signup/#/Merchant/addMerchant).

## Update an existing merchant sale unit

To update a merchant sale unit, send [`PATCH:/v1/merchants/:merchantSerialNumber`](https://vippsas.github.io/vipps-psp-api/signup/#/Merchant/patchMerchant). 
Provide the MSN for the merchant and update the details in the body section.










# Proposals

## Recurring 3DS Update Card

The following is a proposal for triggering 3DS when a user changes the card attached to a PSP recurring agreement in the Vipps app.

A new optional `updateCardUrl` property would be added to the initiate call. This callback will be triggered when the user changes card with the following request;

```json
HEADER: "
        PSP-ID: C948DFD1546347568874C4DDC93A2E3C
        Meerchant-Serial-Number: 123456
        User-Token: eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9
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

## Questions?

We're always happy to help with code or other questions you might have!
Please create an [issue](https://github.com/vippsas/vipps-psp-api/issues),
a [pull request](https://github.com/vippsas/vipps-psp-api/pulls),
or [contact us](https://github.com/vippsas/vipps-developers/blob/master/contact.md).

Sign up for our [Technical newsletter for developers](https://github.com/vippsas/vipps-developers/tree/master/newsletters).
