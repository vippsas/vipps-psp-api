<!-- START_METADATA
---
title: Quick start for the PSP API
sidebar_label: Quick start
sidebar_position: 15
description: Quick steps for getting started with the PSP API.
pagination_next: null
pagination_prev: null
---

END_METADATA -->

# Quick start

Use the PSP API to initiate a PSP payment and get details or update the status of this payment.

## Before you begin

The provided example values in this guide must be changed with the values for your sales unit and user.
This applies for API keys, HTTP headers, reference, phone number, etc.
Note that any currency amount must be an Integer value minimum 100 in øre.

## Your first Payment

### Step 1 - Setup

You must have already signed up as an organization with Vipps MobilePay and have
your test credentials from the merchant portal.

You will need the following values, as described in the
[Getting started guide](https://developer.vippsmobilepay.com/docs/getting-started):

* `client_id` - Merchant key is required for getting the access token.
* `client_secret` - Merchant key is required for getting the access token.
* `Ocp-Apim-Subscription-Key-PSP` - PSP subscription key is required for all requests.
* `mobileNumber` - Test mobile number, needed only for `Initiate a PSP Payment`.
* `merchantSerialNumber` - Required only for `Get Merchant by MSN`.
* `makePaymentUrl` - PSP URL used by Vipps to send the card data.
* `pspRedirectUrl` - Redirect URL which the user is redirected to after approving/rejecting the payment.
* `PSP-ID` - PSP ID provided by Vipps, needed for `Initiate a PSP Payment`, `Update Status`, and `Get Details`.

Note that the requests in this PSP API require your PSP subscription key.

### Step 2 - Authentication

For all the following, you will need an `access_token` from:
[`POST:/accesstoken/get`](https://developer.vippsmobilepay.com/api/access-token#tag/Authorization-Service/operation/fetchAuthorizationTokenUsingPost).

```bash
curl https://apitest.vipps.no/accessToken/get \
-H "client_id: YOUR-CLIENT-ID" \
-H "client_secret: YOUR-CLIENT-SECRET" \
-H "Ocp-Apim-Subscription-Key: YOUR-SUBSCRIPTION-KEY" \
-X POST \
--data ''
```

The property `access_token` should be used for all other API requests in the `Authorization` header as the Bearer token.

### Step 3 - A simple payment

Initiate a payment with:
[`POST:/psp/v3/psppayments/init/`](https://developer.vippsmobilepay.com/api/psp#tag/PSP-API/operation/initiatePaymentV3UsingPOST).

```bash
curl --location 'https://apitest.vipps.no/psp/v3/psppayments/init' \
-H 'Content-Type: application/json' \
-H "Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1Ni <truncated>" \
-H "Ocp-Apim-Subscription-Key: <This is the PSP's key, and is the same for all the PSP's merchants. Keep it secret.>" \
-H 'PSP-ID: <Provided by Vipps>' \
-H "Merchant-Serial-Number: YOUR-MSN" \
-X POST \
-d '{
  "pspTransactionId": "428FY3Zmeyc",
  "merchantOrderId": "8WG39VI9hywg",
  "customerMobileNumber": "96574209",
  "amount": "49900",
  "currency": "NOK",
  "pspRedirectUrl": "https://example.com",
  "makePaymentUrl": "https://example.com",
  "makePaymentToken": "WCAG6W5HYWGg4",
  "paymentText": "Transaction initiated.",
  "isApp": false,
  "skipLandingPage": false
}'
```

### Step 4 - Completing the payment

*Ctrl+click* (*Command-click* on macOS) on the link that appears, and it will take you to the
[landing page](https://developer.vippsmobilepay.com/docs/common-topics/landing-page/).
The phone number of your test user should already be filled in, so you only have to click *Next*.
You will be presented with the payment in the app, where you can complete or reject the payment.
Once you have acted upon the payment, you will be redirected back to the specified `pspRedirectUrl` URL under a "best effort" policy.

:::note
We cannot guarantee the user will be redirected back to the same browser or session, or that they will at all be redirected back. User interaction can be unpredictable, and the user may choose to fully close the app or browser.
:::

### Step 5 - Getting the status of the payment

To see the details about the transaction, run the
[`GET:/psp/v3/psppayments/{pspTransactionId}/details`](https://developer.vippsmobilepay.com/api/psp#tag/PSP-API/operation/getPSPPaymentDetailsUsingGET) request.

```bash
curl https://apitest.vipps.no/psp/v3/psppayments/{pspTransactionId}/details \
-H "Content-Type: application/json" \
-H "Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1Ni <truncated>" \
-H "Ocp-Apim-Subscription-Key: <This is the PSP's key, and is the same for all the PSP's merchants. Keep it secret.>" \
-H 'PSP-ID: <Provided by Vipps>' \
-H "Merchant-Serial-Number: YOUR-MSN" \
-X GET
```

### Step 6 (Optional): Update the payment

You can update the payment with the
[`POST:/psp/v3/psppayments/updatestatus`](https://developer.vippsmobilepay.com/api/psp#tag/PSP-API/operation/updatestatusUsingPOST)
endpoint.

```bash
curl https://apitest.vipps.no/psp/v3/psppayments/updateStatus \
-H "Content-Type: application/json" \
-H "Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1Ni <truncated>" \
-H "Ocp-Apim-Subscription-Key: <This is the PSP's key, and is the same for all the PSP's merchants. Keep it secret.>" \
-H 'PSP-ID: <Provided by Vipps>' \
-H "Merchant-Serial-Number: YOUR-MSN" \
-X POST \
-d '{
  "transactions": [
   {
      "pspTransactionId": "428FY3Zmeyc",
      "status": "CAPTURED",
      "amount": "49900",
      "currency": "NOK",
      "paymentText": "Transaction updated"
   }
  ]
}'
```

## Next steps

See the [PSP API guide](vipps-psp-api.md) to read about the concepts and details.
