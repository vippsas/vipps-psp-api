# Vipps PSP API v3

[Vipps på Nett](https://www.vipps.no/bedrift/vipps-pa-nett)
(eCommerce) via PSP offers functionality for payments on
websites and apps (P2M). Vipps på Nett offers merchants functionality to
provide a solution where the end user only enters the Norwegian mobile number
to retrieve a payment request in the Vipps app.

In the Vipps app the end user selects a payment source to complete the payment
request. Vipps initiates the payment transaction on the selected source and
provide feedback to the PSP of the payment card selected.

The PSP processes the payment transaction and provides feedback to merchant
and Vipps of the payment transaction success or failure.

## Differences between v2 and v3

The PSP API v3 adds functionality for network tokens: PSPs use the API to
obtain tokens, not the actual card details.
Additionally `$.makePaymentRequest.confirmed` has been renamed to `$.makePaymentRequest.paymentState`
Values for this enum have changed accordingly
| Old Value | New Value |
|-----------|-----------|
| Yes | ACCEPTED |
|TIMEOUT| TIMEOUT|
|CANCEL| USER_CANCEL|
|NO| USER_CANCEL|

----

From January 1 2021 all PSPs must be able to process network tokens.

See the
[EMVco documentation](https://www.emvco.com/emv-technologies/payment-tokenisation/)
for more information.

API details: [Swagger UI](https://vippsas.github.io/vipps-psp-api/#/),
[swagger.yaml](https://raw.githubusercontent.com/vippsas/vipps-psp-api/master/docs/swagger.yaml),
[swagger.json](https://raw.githubusercontent.com/vippsas/vipps-psp-api/master/docs/swagger.json).

API version: 3.0

Document version 3.0.0.

# Table of Contents

- [Vipps PSP API v3](#vipps-psp-api-v3)
  - [Differences between v2 and v3](#differences-between-v2-and-v3)
- [Table of Contents](#table-of-contents)
- [Differences from PSP API version 1](#differences-from-psp-api-version-1)
- [PSP payment sequence](#psp-payment-sequence)
- [API overview](#api-overview)
  - [Authentication](#authentication)
  - [Initiate payment](#initiate-payment)
    - [Skip landing page](#skip-landing-page)
    - [Payment confirmation](#payment-confirmation)
    - [makePaymentUrl](#makepaymenturl)
      - [Public key](#public-key)
      - [Card Data format](#card-data-format)
  - [Emvco Token processing](#emvco-token-processing)
    - [Scheme specific details](#scheme-specific-details)
      - [Visa](#visa)
      - [Mastercard](#mastercard)
    - [Token Requestor Ids](#token-requestor-ids)
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
- [PSP Signup APIs](#psp-signup-apis)
- [Questions](#questions)
- [Proposals](#proposals)
  - [Recurring 3DS Update Card](#recurring-3ds-update-card)
  - [Questions?](#questions-1)

# Differences from PSP API version 1

* Added support for redirection of user after payment completion in the Vipps app
* Added support for providing the `makePaymentUrl` in the initiate payment call
* Improved authorization of the `makePaymentUrl` call by adding the `Authorization` header value
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

If the sale unit is not whitelisted, an error message will be returned.

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

**For testing that decryption works**  
Python example
```python
import base64
from cryptography.hazmat.backends import default_backend
from cryptography.hazmat.primitives import serialization, hashes
from cryptography.hazmat.primitives.asymmetric import padding
with open("id_rsa", "rb") as key_file:
    private_key = serialization.load_pem_private_key(
        key_file.read(),
        password=None,
        backend=default_backend()
    )
with open("enc.text") as enc_file:
    cipher_text = enc_file.read()
ciphertext = base64.b64decode(cipher_text)
plaintext = private_key.decrypt(
    ciphertext,
    padding.OAEP(
        mgf=padding.MGF1(algorithm=hashes.SHA256()),
        algorithm=hashes.SHA256(),
        label=None
    )
)
print(plaintext)
```
Alternative for C#
```cs
var bytesToDecrypt = Convert.FromBase64String(toDecrypt);
var pemReader = new PemReader(new StringReader(_privateKey));
var privateKeyObject = pemReader.ReadObject();
var keyPair = (AsymmetricCipherKeyPair)privateKeyObject;
var decryptEngine = new OaepEncoding(new RsaEngine(), new Sha256Digest());
decryptEngine.Init(false, keyPair.Private);
var decryptedBytes = decryptEngine.ProcessBlock(bytesToDecrypt, 0, bytesToDecrypt.Length);
var resultString = BytesToString(decryptedBytes);
```

#### Card Data format

The `cardData` is a string in the format
`{CardNumber:16-19},{ExpiryDate:4},{SessionId:1-32}`
that has been transformed into a 256 bytes OAEP cryptogram using the public
key provided by the PSP. The cryptogram is encoded as 344-characters base64 string.
The `ExpiryDate` is in YYMM format.

## Emvco Token processing

```
NB: Minor details subject to change, the full solution is out in test. 
```

In order to give the best possible payment experience, the Vipps PSP solution will begin supporting
EMVCO token based processing.

The ECI value will be the same for encrypted card based and token based transactions.

The solution will function on a flow level identical to it's current implementation, but the PSP
will have to support EMVCO token processing. The Vipps endpoints are, except for the version number, otherwise identical.

This will result in a new MakePayment callback where the Encrypted card details are replaced with a token
and cryptogram in the following format. This is refered to as the makePayment request.

```
Authorization: makePaymentToken
{
  "pspTransactionId": "7686f7788898767977",
  "merchantSerialNumber": "123456",
  "paymentState": "ACCEPTED/TIMEOUT/USER_CANCEL",
  "paymentInstrument": "TOKEN",
  "networkToken": {
    "number": "12345678901234",
    "expiryMonth": "12",
    "cryptogram": "aFgdgjdkfgjdFDF=",
    "tokenType": "MASTERCARD",
    "expiryYear": "2025"
  }
}
```

If Vipps can not provide a valid netWorkToken transaction, either due to downtime or missing issuer support we will provide the encrypted card exactly in the same manner as the existing solution.

```
Authorization: makePaymentToken
{
  "pspTransactionId": "7686f7788898767977",
  "merchantSerialNumber": "123456",
  "paymentState": "ACCEPTED/TIMEOUT/USER_CANCEL",
  "paymentInstrument" : "ENCRYPTEDCARD",
  "cardData": "f0a29801b4#d4ff30e221fa2980ff30e2",
}
```

Where paymentInstrument is as an indicator that can be used to differentiate the two alternatives.

Where networkToken is the Network token of the card, up to 16-19 digits. A full replacement of the PAN. 

### Scheme specific details

#### Visa

Visa tokens must be processed with the acquirer submitting the TAVV cryptogram in field 126.8. The cryptogram received with the Network Token will contain the information for Delegated Authentication (DA) and the SCA factors used. Visa Token Service will during detokenization populate a flag for DA in field 34 to issuers and the Vipps TRID in field 123 Usage 2 Tag 03. In this way Issuers recognise Vipps originated transactions and not soft decline for 3DS step-up unless the issuing bank has opted out of the Visa D-SCA program.

#### Mastercard

A Mastercard transaction should be processed as an ecom-token in accordance with the acquirers instructions from Mastercard. Mastercard adds the TRID to the authoriziation message. It will always be available in DE48, SE33, SF6.

```
NB Vipps will communicate more regarding Mastercard D-SCA shortly.
```


### Token Requestor Ids

For Visa/Mastercard the TRID is an eleven digit number. And is added by the Scheme in the processing of the Payment.

### Magic Numbers for EMVCo Tokens

Any request to the v3 API will return a Visa Token. However this can be changed by setting the amount in the init request. No matter what is selected in the app the Token returned in the MakePayment request will be:

| Amount Value | Instrument Sent |
|-----------|-----------|
| 22.00 | MasterCard |
| 31.00 | Encrypted card as chosen in app |

## Status Updates

To provide a consistent end user experience it is important that Vipps is notified by changes to the payment status when it is captured, cancelled or refunded: [`POST:/v3/psppayments/updatestatus`](https://vippsas.github.io/vipps-psp-api/#/Vipps_PSP_API/updatestatusUsingPOST)

Vipps also provides an endpoint to check the payment status: [`POST:/v3/psppayments/{pspTransactionId}/details`](https://vippsas.github.io/vipps-psp-api/#/Vipps_PSP_API/getPSPPaymentDetailsUsingGET)

For customers upgrading from the PSP API v1: It is ok to call `updateStatus`
with the v3 API on payments done with the v1 API.

### Batch processing of status updates

Requests to
[`POST:/v3/psppayments/updatestatus`](https://vippsas.github.io/vipps-psp-api/#/Vipps_PSP_API/updatestatusUsingPOST)
receive a `HTTP 200 OK` response if the JSON payload was valid.
The actual _processing_ of the data is done as a batch.

The reasoning is that in designing the API this way, was that some partners
wanted to send all status updates once a day. Based on this we created
a service that can consume thousands of transaction updates in one operation.

To avoid all of these updates being written to our database at once, with the
resulting performance hit, we chose to defer these into a nightly batch job.

Because of this, the updated status is only visible in Vipps the next day.

**Please note:** The batch update is _not_ run in the test environment, as
there are some technical details preventing it from being run in the same way
as in the production environment.

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
redirect according to the operations.url object sent in the initial makepayment request, where upon the app
will close the iframe. Vipps will then resend the `makePaymentUrl` request.

For example if a 3ds session succeeds you should redirect to the 
Operations.url with the operation `3dssuccess`.

```
operations\":[{\"url\":\"https://www.google.no/?transactionId=xxxxx&responsecode=OK\",\"operation\":\"3dssuccess\"}"
```

Note that the responseCode query paramater is critical.

The status in the response to this `makePaymentUrl` should never be
`SOFT_DECLINE`, only `FAIL` or `OK`.
Once the status is returned it will be displayed to the user as normal in the app.

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
control. This has been built as an extension to the existing PSP v3 API, and no
existing integrations will be affected, other than the possibility to initialize
and preform recurring payments.

## Scopes

As of now, there are is one possible way to perform a recurring payment: `psp_subscription`.
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

The `scope` can at the time be set to  `psp_subscription`.
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
  "cardData": "f0a29801b4#d4ff30e221fa2980ff30e2",
  "paymentText": "Description of payment"
}
```

Once the card data is received from [`POST:/v3/psppayments/payments`](https://vippsas.github.io/vipps-psp-api/#/Vipps%20PSP%20API/processPaymentOnToken) and the payment has been processed, the PSP must call
[`POST:/v3/psppayments/updatestatus`](https://vippsas.github.io/vipps-psp-api/#/Vipps_PSP_API/updatestatusUsingPOST) to notify Vipps of the status, in this context updateStatus accepts a `RESERVED` status.
If the status for the previous payment has not been recieved, the agreement will be locked from processing future payments until the update is recieved.
If a recurring payment fails you should call [`POST:/v3/psppayments/updatestatus`](https://vippsas.github.io/vipps-psp-api/#/Vipps_PSP_API/updatestatusUsingPOST) with `operationStatus: FAILED` set in the body.

# URL Validation

All URLs in Vipps eCommerce API are validated with the
[Apache Commons UrlValidator](https://commons.apache.org/proper/commons-validator/apidocs/org/apache/commons/validator/routines/UrlValidator.html).

If `isApp` is true, any `pspRedirectUrl` is not validated with Apache Commons UrlValidator,
as the app-switch URL may be something like `vipps://`, which is not a valid URL.

The endpoints required by Vipps must be publicly available.

URLs that start with `https://localhost` will be rejected. If you want to use
localhost as fallback, please use `http://127.0.0.1`.
It is, naturally, not possible to use `https://localhost` or
`http://127.0.0.1` for the callback, as the Vipps backend would then call itself.

Here is a simple Java class suitable for testing URLs,
using the dummy URL `https://example.com/vipps/fallback-result-page/order123abc`:

```java
import org.apache.commons.validator.routines.UrlValidator;

public class UrlValidate {
 public static void main(String[] args) {
  UrlValidator urlValidator = new UrlValidator();

  if (urlValidator.isValid("https://example.com/vipps/fallback-result-page/order123abc")) {
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

# PSP Signup APIs

The Vipps PSP Signup API allows PSPs to onboard and control their merchants.
The API specification can be found [here](https://github.com/vippsas/vipps-psp-api/blob/master/docs/signup/openapi.yaml)

A PSP can use their existing keys to access this APIs. They can perform the following
- List all or one merchant(s) under them
- Create a new merchant under them
- Update an existing merchant

Currently we only allow the PSPs to onboard merchants who have a Norwegian organisation number.

The following are the screens in the Vipps app, where the information about the merchant that was provided by the PSP is rendered to the end user.

![Payment Screen](./docs/signup/payment.png)

![Receipt Screen](./docs/signup/receipt.png)


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
