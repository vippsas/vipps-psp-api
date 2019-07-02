# Vipps PSP API version 2

[Vipps på Nett](https://www.vipps.no/bedrift/vipps-pa-nett)
(eCommerce) via PSP offers functionality for payments on
websites and apps (P2B). "Vipps på Nett" provides merchants a solution where
the end user only enters a Norwegian mobile number to make a payment with a
card in the Vipps app.

In the Vipps app the end user selects a payment source to complete the payment
request. Vipps initiates the payment transaction on the selected source and
provide feedback to the PSP of the payment card selected.

The PSP processes the payment transaction and provides feedback to merchant
and Vipps of the payment transaction success or failure.

API version: 2.0

Document version 1.0.4

API details: [Swagger UI](https://vippsas.github.io/vipps-psp-api/#/),
[swagger.yaml](https://raw.githubusercontent.com/vippsas/vipps-psp-api/master/docs/swagger.yaml),
[swagger.json](https://raw.githubusercontent.com/vippsas/vipps-psp-api/master/docs/swagger.json).

## Differences from PSP API version 1

* Added support for redirection of user after payment completion in the Vipps app
* Added support for providing the `makePayment` URL in the initiate payment call
* Improved authorization of the `makePayment` call by adding the authorization header value
* Improved and more consistent parameter names in the API

## PSP payment sequence

### Summary

1. PSP initiates payment: [`POST:/v2/psppayments/init`](https://vippsas.github.io/vipps-psp-api/#/Vipps_PSP_API/initiatePaymentV2UsingPOST)
2. User confirms payment in Vipps
3. Vipps sends encrypted card details to PSP: [`POST` to the PSP-specified `makePaymentUrl`](https://vippsas.github.io/vipps-psp-api/#/Endpoints_required_by_Vipps_from_the_PSP/makePaymentSwaggerUsingPOST)
4. PSP uses the encrypted card details to perform the payment.
5. PSP informs Vipps about the payment status: [`POST:/v2/psppayments/updatestatus`](https://vippsas.github.io/vipps-psp-api/#/Vipps_PSP_API/updatestatusUsingPOST)

Please note that Vipps is _not_ involved in the actual payment. 
Vipps provides the encrypted card details to the PSP, and it is the PSP that performs the payment.

**Important:** Some users may close Vipps immediately after seeing the payment confirmation, 
therefore not being "redirected" back to the merchant. Because of this it is important for the
merchant and the PSP to _not_ base their transaction logic on the redirect alone. 
For example: Check for "reserved" status with the PSP's API (not Vipps' API), 
then do "capture" when the goods have been delivered. 
See the [Vipps PSP API Checklist](vipps-psp-api-checklist.md).  

### Sequence diagram

![PSP API sequence diagram](images/psp-sequence-diagram.png)

### PSP implementation checklist

See the [Vipps PSP API Checklist](vipps-psp-api-checklist.md) for details.

### Initiate payment

A payment request is initiated by the PSP to the Vipps API after end user has
request to pay with Vipps. Vipps creates the payment and returns a link to
the Vipps landing page where end user can confirm the mobile number.
Once user has confirmed number the payment can be considered initiated.

### Payment confirmation

After payment initiation, Vipps sends push notification or redirects user to
the Vipps app. End user verifies the Vipps profile by logging in to the Vipps
app. In the Vipps app the end user can select payment source and confirm the amount.

### MakePayment

Once the end user has confirmed the payment on the Vipps landing page, Vipps
shares the encrypted card details with the PSP with
[`POST:makePaymentUrl`](https://vippsas.github.io/vipps-psp-api/#/Endpoints_required_by_Vipps_from_the_PSP/makePaymentSwaggerUsingPOST).

Vipps does _not_ call the `makePaymentUrl` if the order is not activated by the user. The user activates the order by either accepting/cancelling in the landing page or logging in to the app from a deeplink/landing page redirect. Vipps may also provide additional `makePayment` calls if the user retries a already paid order. In this case the second `makePayment` call should not override the result from the first.

The PSP tries to process the payment through the acquirer and responds to the
`makePayment` call with the payment request status. The end user receives
confirmation in the Vipps app. Vipps redirects the end user to the `redirectUrl`
provided during payment initiation.

#### MakePayment status
| Enum value    | Description                                   |
| --------------| --------------------------------------------- |
| YES           | User successfully approved the payment        |
| NO            | Something failed during approval              |
| TIMEOUT       | User didn't act on the payment                |
| CANCEL        | User cancelled the payment                    |

## Example request

```json
Authorization: makePaymentToken
{
  "pspTransactionId": "7686f7788898767977",
  "merchantSerialNumber": "123456",
  "cardData": "f0a29801b4#d4ff30e221fa2980ff30e2",
  "confirmed": "YES"
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

## Update status

Since Vipps doesn’t process transactions, updates on payment status are required
in order to deliver expected customer experience. That means that PSP has to inform
Vipps about any PSP payment status change with
[`POST:/v2/psppayments/updatestatus`](https://vippsas.github.io/vipps-psp-api/#/Vipps_PSP_API/updatestatusUsingPOST).

### Example request

```json
{
  "transactions": [
    {
      "pspTransactionId": "7686f7788898767977",
      "status": "CAPTURED",
      "amount": 20000,
      "currency": "NOK",
      "paymentText": "One pair of Vipps socks"
    }
  ]
}
```

### Example response

```json
{
  "responseInfo": {
    "responseCode": "9000",
    "responseMessage": "SUCCESS"
  }
}
```

## Idempotency

All API requests can be retried without any side effects
by providing the `Request-Id` idempotency key in the header of the request.

In case of network issues or other problems, the API call can be safely retried
with the same idempotency key. The idempotency key is a unique alphanumeric id
generated by the merchant.

```
Request-Id: slvnwdcweofjwefweklfwelf
```

# Authentication

Every API call is authenticated and authorized based on the application
Authorization token (JWT) and APIM subscription key
(`Ocp-Apim-Subscription-Key`).

The following headers are required in every API request
to successfully authenticate every API call:

| Header Name | Header Value | Description |
| :-------------------------- | :--------------------------- | :--------------------------------------------- |
| Authorization | Bearer 'jwt_access_token'' | type: Authorization token. Value: Access token is obtained from accessToken/get |
| Ocp-Apim-Subscription-Key | Base 64 encoded string | Subscription key for the product.<br>Can be found in Vipps developer portal |


## Error codes

| errorId | errorText                            |
| ------- | ------------------------------------ |
| 71      | Invalid request                      |
| 72      | Different texts                      |
| 81      | No such issuer                       |
| 82      | Refused by Issuer                    |
| 83      | Suspected fraud                      |
| 84      | Exceeds withdrawal amount limit      |
| 85      | Response received to late            |
| 86      | Expired card                         |
| 87      | Invalid card number (no such number) |
| 88      | Merchant does not allow credit cards |
| 89      | Insufficient funds                   |
| 91      | Internal error                       |
| 92      | Unable to decrypt                    |
| 93      | Status from Vipps:CANCEL or Status from Vipps:TIMEOUT |

Note 93 is for when the makePayment request from Vipps contains the statuses CANCEL or TIMEOUT. Cancel is when the user cancels in the Vipps app, and TimeOut is when the user does not act on the payment.

### Status Updates

To provide a consistent end user experience it is important that Vipps is notified by changes to the payment status when it is captured, cancelled or refunded. Vipps also provides an endpoint to check the payment status.

# PSD2 Compliance

In order to be allow for delegated SCA through the PSD2 directive every transaction needs to be marked with Vipps's WalletId. Vipps's WalletId is A62.

## 3DSecure Fallback.

In case of a soft decline (issuer demanded 3DS) the PSP will need to provide the 3DSecure to Vipps.

![PSP API sequence diagram](diagrams/3DSFallbackFlow.png)

Format of MakePaymentRequest response provided by the PSP in case of a soft decline

```json
{
  "errorMessage": {
    "errorId": 1036,
    "errorText": "Soft Decline"
  },
  "paymentInfo": {
    "pspTransactionId": "7686f7788898767977",
    "status": "Challenge",
    "3DSUrl": "https://terminal.com/example"
  }
}
```

The Vipps App will then host the URL in an iframe, letting the user complete the 3DSecure flow. Vipps will then forward the data from the 3DSecure session. The data will be sent in a an otherwise identical version of the MakePaymentRequest that got the soft decline.

```json
Authorization: makePaymentToken
{
  "pspTransactionId": "7686f7788898767977",
  "merchantSerialNumber": "123456",
  "cardData": "f0a29801b4#d4ff30e221fa2980ff30e2",
  "confirmed": "YES/TIMEOUT/CANCEL",
  "3DSecureData": {
    "pares" : "eNpdU8tymzAU3ecrvMumYz1AgD2yZnDsTpMZx06TRdudLK5skhiIgNjust/Tr+qXVMIm4DDDoHvuOdKZexB/2hqA2SOo2oC4Ggz4AspSbmCQJpPrSELAZKApSOyFkWYaY+brJAq0pzTQa6ewmlX8Hd4aRZDoKMIs8WXiK2/NvIAFDPwwoRrCkV6fFVbzDqZM80yQIR5SjtqybRcyE4xFPmUhoZiGhPkRRw5tGQswaiuzqgUsJNXb9PZeMBpYFUfnsuvvwNzOBKa4fTg6QR0lkzsQ84PcFa/AUVN1TZXXWWWOgmLrpS26dm1exbaqijFC+/1+CKddhrnZII5cs7WOPnvnq9oBZf+wQ5qIxSzed+/8uPj9QO6flb98iiccOUbHT2QF1hlhmODRgARj6o0Z46jBezPaOd/i35+/hH7x7ATOQMconJf4hBLqKH2kN43aGMjUUYxCN4626ghwKPIMrMbm+7HuGYZSCevPfT4m83kQ/ObbRcCqsnHls7V++fVAHoPlnb/9eVMuVz+m8fzrNN5MXOwN6cJHaoMiETkZSbvUOGr3t0e7v7i5A+h8CcQVR5cX5D/FxN2u"
  }
}
```

# Recurring PSP payments

The PSP API supports recurring payments out of the box. This enables the PSP to perform recurring payments through Vipps, while retaining full transactional control. This has been built as an extension to the existing PSPv2 API, and no existing integrations will be affected, other than the possibility to initialize and preform recurring payments.

## Subscription and oneclick scopes

As of now, there are two possible ways to perform a recurring payment - *subscription* and *oneclick*. These are referred to as the *scope* of the recurring agreement. Only one *scope* can be used at a time, and it's not possible to change the scope of an agreement.

*Subscription* based payments are created as a consent to an agreement that allows the PSP to withdraw money on unknown intervals. This implies that the user won't have to accept the payment on each occasion, only the first one when consenting to the agreement. An example of this could be a subscription to a music streaming service.

*Oneclick* payments are created as agreements where the user will be prompted on each payment. An example of a oneclick payment might be a ticketing service where the user will get prompted to buy a new ticket when the old one expires. Instead of switching to the app to accept the payment, the user already has a oneclick recurring agreement with Vipps as the payment source, and the ticketing app won´t need to involve the Vipps app at all.

## Initialize a recurring payment

Initializing a recurring payment works in the same way as a non-recurring payment, but with the inclusion of a *scope* and *agreementURL* in the init call.

The *scope* can at the time be set to either *subscription* or *oneclick*. The *agreementURL* should be a link to where the user can click to administer the agreement.

To start the initialization, create a standard /init call with the addition of the required fields. If you want to initialize a recurring *subscription*, the init request body could look like this.

```json
{
  "pspRedirectUrl": "yourRedirectUrl",
  "amount": 1337,
  "makePaymentToken": "tokenGoesHere",
  "makePaymentUrl": "yourCallbackEndpointUrl",
  "currency": "NOK",
  "merchantOrderId": "123123123",
  "isApp": false,
  "pspTransactionId": "{{psptransactionid}}",
  "paymentText": "Order id: 213213",
  "scope": "subscription",
  "agreementURL": "linkToTheAgreement"
}
```

In the same way as a normal, non-recurring PSPv2 payment, the PSP will receive a *makePayment* callback. In the body of this callback, you will now also find a *userToken*.

## The userToken

The user token is a token generated when the user has given a consent. This token is provided to the PSP in the makePayment callback when initializing or preforming a recurring payment.

A *userToken* contains the *scope* of the consent, in the claim named *scope*. The token also includes various useful information

```
{
  "sub" // agreement_id
  "aud" // audience (usually something like "https://vipps.no")
  "iat" // issue_day_time
  "scope"
}
```

## Do the next recurring payment

After initialisation, the next payment can be made by passing your *userToken* to the */payments* endpoint as a header with the name *userToken*.

```json
HEADER: "
        Authorization: asd123
        Ocp-Apim-Subscription-Key: asdf234
        PSP-ID: sdf421
        Meerchant-Serial-Number: dsdf322
        UserToken: 123abc
        "
{
  "pspTransactionId": "123",
  "merchantOrderId": "123ABCD",
  "amount": 100,
  "currency": "NOK",
  "makePaymentUrl": "http://yourPaymentUrl",
  "makePaymentToken": "token123",
  "paymentText": "Description of payment"
}
```


## HTTP responses

This API returns the following HTTP statuses in the responses:

| HTTP status         | Description                                 |
| ------------------- | ------------------------------------------- |
| `200 OK`            | Request successful                          |
| `400 Bad Request`   | Invalid request, see the error for details  |
| `401 Unauthorized`  | Invalid credentials                         |
| `403 Forbidden`     | Authentication ok, but credentials lacks authorization  |
| `404 Not Found`     | The resource was not found  |
| `500 Server Error`  | An internal Vipps problem.                  |

### Error codes

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

# Questions?

We're always happy to help with code or other questions you might have!
Please create an [issue](https://github.com/vippsas/vipps-psp-api/issues),
a [pull request](https://github.com/vippsas/vipps-psp-api/pulls),
or [contact us](https://github.com/vippsas/vipps-developers/blob/master/contact.md).
