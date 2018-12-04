# Vipps PSP API

Vipps Netthandel (eCommerce) via PSP offers functionality for payments on
websites and apps (P2M). Vipps Netthandel offer merchants functionality to
provide a solution where the end user only enters her Norwegian mobile number
to retrieve a payment request in Vipps app.

In the Vipps app the end user
selects a payment source to complete the payment request. Vipps initiates
the payment transaction on the selected source and provide feedback to the
PSP of the payment card selected.

The PSP processes the payment transaction
and provides feedback to merchant and Vipps of the payment transaction
success or failure.

*API version: 2*\
*Document version 1.0.0*

## Differences from PSP API version 1

* Added support for redirection of user after payment completion in Vipps app
* Added support for providing the makePayment-URL in the initiate payment call
* Improved authorization of the makePayment call by adding the authorization header value
* Improved and more consistent parameter names in the API

## PSP payment sequence

![PSP API sequence diagram](images/psp-sequence-diagram.png)

### Initiate payment

Payment request is initiated by PSP to Vipps API after end user has request to pay with Vipps. Vipps creates the payment and returns a link to Vipps landing page where end user can confirm the mobile number. Once user has confirmed number the payment can be considered initiated.

### Payment confirmation

After payment initiation, Vipps sends push notification or redirects user to Vipps app. End user verifies the Vipps profil by logging in to Vipps app. In Vipps app the end user can select payment source and confirm the amount.

### MakePayment

Once the end user has confirmed the payment, Vipps shares the encrypted card details with the PSP to the makePaymentUrl. PSP tries to process the payment through the acquirer and responds to the makePayment-call with the payment request status. End user receives confirmation in Vipps app. Vipps redirects the end user to the redirectUrl provided during payment initiation.

Example request:
```http
Authorization: makePaymentToken
{
  "pspTransactionId": "7686f7788898767977",
  "merchantSerialNumber": "123456",
  "cardData": "f0a29801b4#d4ff30e221fa2980ff30e2",
  "confirmed": "YES/TIMEOUT/CANCEL"
}
```

Example response:
```http
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

**Error codes**\
| errorId | errorText                             |
| ------- | ------------------------------------- |
| 71      | Invalid request                       |
| 72      | Different texts                       |
| 81      | No such issuer                        |
| 82      | Refused by Issuer                     |
| 83      | Suspected fraud                       |
| 84      | Exceeds withdrawal amount limit       |
| 85      | Response received to late             |
| 86      | Expired card                          |
| 87      | Invalid card number (no such number)  |
| 88      |                                       |
| 91      | Internal error                        |
| 92      | Unable to decrypt                     |
| 93      | NA                                    |


### Status Updates

To provide a consistent end user experience it is important that Vipps is notified by changes to the payment status when it is captured, cancelled or refunded. Vipps also provides an endpoint to check the payment status.


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

We're always happy to help with code or other questions you might have! Please create an [issues](https://github.com/vippsas/vipps-recurring-api/issues), a [pull requests](https://github.com/vippsas/vipps-recurring-api/pulls), or contact us at `integration@vipps.no`.
