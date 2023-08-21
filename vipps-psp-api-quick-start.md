<!-- START_METADATA
---
title: Quick start for the PSP API
sidebar_label: Quick start
sidebar_position: 15
description: Quick steps for getting started with the PSP API.
pagination_next: null
pagination_prev: null
---

import ApiSchema from '@theme/ApiSchema';
import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';

END_METADATA -->

# Quick start

Use the PSP API to initiate a PSP payment and get details or update the status of this payment.

## Before you begin

This document covers the quick steps for getting started with the PSP API.
You must have already signed up as an organization with Vipps MobilePay and have
your test credentials from the merchant portal, as described in the
[Getting started guide](https://developer.vippsmobilepay.com/docs/getting-started).

**Important:** The examples use standard example values that you must change to
use *your* values. This includes API keys, HTTP headers, reference, etc.

## Your first Payment

### Step 1 - Setup

<Tabs
defaultValue="curl"
groupId="sdk-choice"
values={[
{label: 'curl', value: 'curl'},
{label: 'Postman', value: 'postman'},
]}>
<TabItem value="postman">

**Please note:** Postman is discontinuing their offline version. Use only your test keys and delete them after testing. Ensure that your company allows for cloud use before continuing.

If you wish to use Postman, import the following files:

* [PSP API Postman collection](/tools/vipps-psp-v3-api-postman-collection.json)
* [Global Postman environment](https://github.com/vippsas/vipps-developers/blob/master/tools/vipps-api-global-postman-environment.json)

In Postman, tweak the environment with your own values (see
[API keys](https://developer.vippsmobilepay.com/docs/common-topics/api-keys/)):

* `client_id` - Merchant key is required for getting the access token.
* `client_secret` - Merchant key is required for getting the access token.
* `Ocp-Apim-Subscription-Key-PSP` - PSP subscription key is required for all requests.
* `mobileNumber` - Test mobile number, needed only for `Initiate a PSP Payment`.
* `merchantSerialNumber` - Required only for `Get Merchant by MSN`.
* `makePaymentUrl` - PSP URL used by Vipps to send the card data.
* `pspRedirectUrl` - Redirect URL which the user is redirected to after approving/rejecting the payment.
* `PSP-ID` - PSP ID provided by Vipps, needed for `Initiate a PSP Payment`, `Update Status`, and `Get Details`.

</TabItem>
<TabItem value="curl">

No setup needed :)

</TabItem>
</Tabs>

Note that the requests in this PSP API require your PSP subscription key.

### Step 2 - Authentication

For all the following, you will need an `access_token` from the
[Access token API](https://developer.vippsmobilepay.com/docs/APIs/access-token-api):
[`POST:/accesstoken/get`](https://developer.vippsmobilepay.com/api/access-token#tag/Authorization-Service/operation/fetchAuthorizationTokenUsingPost).
This provides you with access to the API.

<Tabs
defaultValue="curl"
groupId="sdk-choice"
values={[
{label: 'curl', value: 'curl'},
{label: 'Postman', value: 'postman'},
]}>
<TabItem value="postman">

```bash
Send request Get Access Token
```

</TabItem>
<TabItem value="curl">

```bash
curl https://apitest.vipps.no/accessToken/get \
-H "client_id: YOUR-CLIENT-ID" \
-H "client_secret: YOUR-CLIENT-SECRET" \
-H "Ocp-Apim-Subscription-Key: YOUR-SUBSCRIPTION-KEY" \
-X POST \
--data ''
```

</TabItem>
</Tabs>

The property `access_token` should be used for all other API requests in the `Authorization` header as the Bearer token.

### Step 3 - A simple payment

Initiate a payment with: [`POST:/v3/psppayments/init/`](https://developer.vippsmobilepay.com/api/psp#tag/Vipps-PSP-API/operation/initiatePaymentV3UsingPOST).

<Tabs
defaultValue="curl"
groupId="sdk-choice"
values={[
{label: 'curl', value: 'curl'},
{label: 'Postman', value: 'postman'},
]}>
<TabItem value="postman">

```bash
Send request Initiate a PSP Payment
```

The `pspTransactionId` should be added to the environment automatically.

</TabItem>
<TabItem value="curl">

```bash
curl --location 'https://apitest.vipps.no/psp/v3/psppayments/init' \
-H 'Content-Type: application/json' \
-H "Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1Ni <truncated>" \
-H "Ocp-Apim-Subscription-Key: <This is the PSP's key, and is the same for all the PSP's merchants. Keep it secret.>" \
-H 'PSP-ID: <Provided by Vipps>' \
-H "Merchant-Serial-Number: 123456" \
-X POST \
-d '{
  "pspTransactionId": "428FY3JTWGZW",
  "merchantOrderId": "8WG39VI9GSAN",
  "customerMobileNumber": "96574209",
  "amount": "49900",
  "currency": "NOK",
  "pspRedirectUrl": "https://example.com",
  "makePaymentUrl": "https://example.com",
  "makePaymentToken": "OHRB6W5AFSX4",
  "paymentText": "Transaction initiated through Postman",
  "isApp": false,
  "skipLandingPage": false
}'
```

</TabItem>
</Tabs>

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
[`GET:/v3/psppayments/{pspTransactionId}/details`](https://developer.vippsmobilepay.com/api/psp#tag/Vipps-PSP-API/operation/getPSPPaymentDetailsUsingGET) request.

<Tabs
defaultValue="curl"
groupId="sdk-choice"
values={[
{label: 'curl', value: 'curl'},
{label: 'Postman', value: 'postman'},
]}>
<TabItem value="postman">

```bash
Send request Get Details
```

</TabItem>
<TabItem value="curl">

```bash
curl https://apitest.vipps.no/psp/v3/psppayments/{pspTransactionId}/details \
-H "Content-Type: application/json" \
-H "Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1Ni <truncated>" \
-H "Ocp-Apim-Subscription-Key: <This is the PSP's key, and is the same for all the PSP's merchants. Keep it secret.>" \
-H 'PSP-ID: <Provided by Vipps>' \
-H "Merchant-Serial-Number: 123456" \
-X GET
```

</TabItem>
</Tabs>

### Step 6 (Optional): Update the payment

You can update the payment with the
[`POST:/v3/psppayments/updatestatus`](https://developer.vippsmobilepay.com/api/psp#tag/Vipps-PSP-API/operation/updatestatusUsingPOST) endpoint.

<Tabs
defaultValue="curl"
groupId="sdk-choice"
values={[
{label: 'curl', value: 'curl'},
{label: 'Postman', value: 'postman'},
]}>
<TabItem value="postman">

```bash
Send request Update Status
```

</TabItem>
<TabItem value="curl">

```bash
curl https://apitest.vipps.no/psp/v3/psppayments/updateStatus \
-H "Content-Type: application/json" \
-H "Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1Ni <truncated>" \
-H "Ocp-Apim-Subscription-Key: <This is the PSP's key, and is the same for all the PSP's merchants. Keep it secret.>" \
-H 'PSP-ID: <Provided by Vipps>' \
-H "Merchant-Serial-Number: 123456" \
-X POST \
-d '{
  "transactions": [
   {
      "pspTransactionId": "428FY3JTWGZW",
      "status": "CAPTURED",
      "amount": "49900",
      "currency": "NOK",
      "paymentText": "Transaction updated through Postman"
   }
  ]
}'
```

</TabItem>
</Tabs>


## Next steps

See the [PSP API guide](vipps-psp-api.md) to read about the concepts and details.
