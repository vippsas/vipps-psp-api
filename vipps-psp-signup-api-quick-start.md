<!-- START_METADATA
---
title: Quick start for the PSP Merchant Sign up
sidebar_label: Signup Quick start
sidebar_position: 50
description: Quick steps for getting started with the PSP Merchant Signup.
pagination_next: null
pagination_prev: null
---

import ApiSchema from '@theme/ApiSchema';
import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';

END_METADATA -->

# Quick start for PSP Merchant Signup

## Before you begin

The provided example values in this guide must be changed with the values for your sales unit and user.
This applies for API keys, HTTP headers, reference, phone number, etc.

## Sign up your first merchant

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
* `makePaymentUrl` - PSP URL used by Vipps to send the card data, needed only for `Initiate a PSP Payment`.
* `pspRedirectUrl` - Redirect URL which the user is redirected to after approving/rejecting the payment.
* `PSP-ID` - PSP ID provided by Vipps, needed for `Initiate a PSP Payment`, `Update Status`, and `Get Details`.
* `idempotency_key` - Unique request ID, needed only for `Create new Merchant Sales Unit`.


<Tabs
defaultValue="curl"
groupId="sdk-choice"
values={[
{label: 'curl', value: 'curl'},
{label: 'Postman', value: 'postman'},
]}>
<TabItem value="postman">

In Postman, import the following files:

* [PSP API Postman collection](/tools/vipps-psp-v3-api-postman-collection.json)
* [Global Postman environment](https://github.com/vippsas/vipps-developers/blob/master/tools/vipps-api-global-postman-environment.json)

🔥 **To reduce risk of exposure, never store production keys in Postman or any similar tools.** 🔥

Update the *Current Value* field in your Postman environment with your **Merchant Test** keys.
Use *Current Value* field for added security, as these values are not synced to the cloud.

</TabItem>
<TabItem value="curl">

No additional setup needed :)

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

### Step 3 - Get all merchants

Send
[`GET:/v1/merchants`](https://developer.vippsmobilepay.com/api/psp-signup#tag/Merchant/operation/getMerchants)
for a JSON response showing all the merchants and their information.

<Tabs
defaultValue="curl"
groupId="sdk-choice"
values={[
{label: 'curl', value: 'curl'},
{label: 'Postman', value: 'postman'},
]}>
<TabItem value="postman">

```bash
Send request Get all Merchants
```

</TabItem>
<TabItem value="curl">

```bash
curl https://apitest.vipps.no/merchant-management/psp/v1/merchants \
-H "Content-Type: application/json" \
-H "Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1Ni <truncated>" \
-H "Ocp-Apim-Subscription-Key: <This is the PSP's key, and is the same for all the PSP's merchants. Keep it secret.>" \
-X GET
```

</TabItem>
</Tabs>

### Step 4 - Get Merchant by MSN

Use
[`GET:/v1/merchants/{merchantSerialNumber}`](https://developer.vippsmobilepay.com/api/psp-signup#tag/Merchant/operation/getMerchant)
for information about a specific merchant. Supply the MSN.

<Tabs
defaultValue="curl"
groupId="sdk-choice"
values={[
{label: 'curl', value: 'curl'},
{label: 'Postman', value: 'postman'},
]}>
<TabItem value="postman">

```bash
Send request Get Merchant by MSN
```

</TabItem>
<TabItem value="curl">

```bash
curl https://apitest.vipps.no/merchant-management/psp/v1/merchants/{merchantSerialNumber} \
-H "Content-Type: application/json" \
-H "Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1Ni <truncated>" \
-H "Ocp-Apim-Subscription-Key: <This is the PSP's key, and is the same for all the PSP's merchants. Keep it secret.>" \
-X GET
```

</TabItem>
</Tabs>

### Step 5 - Create new Merchant Sales Unit

Send [`POST:`](https://developer.vippsmobilepay.com/api/psp-signup#tag/Merchant/operation/addMerchant). Notice the `merchantSerialNumber` is returned.

<Tabs
defaultValue="curl"
groupId="sdk-choice"
values={[
{label: 'curl', value: 'curl'},
{label: 'Postman', value: 'postman'},
]}>
<TabItem value="postman">

```bash
Send request Create new Merchant Sales Unit
```

</TabItem>
<TabItem value="curl">

```bash
curl https://apitest.vipps.no/merchant-management/psp/v1/merchants \
-H "Content-Type: application/json" \
-H "Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1Ni <truncated>" \
-H "Ocp-Apim-Subscription-Key: <This is the PSP's key, and is the same for all the PSP's merchants. Keep it secret.>" \
-H 'Idempotency-Key: cac4fd6c-6020-4570-aec9-0a249d1ac64a' \
-X POST \
--data '{
    "name": "Fry Merch Shop",
    "address": {
        "addressLine1": "Robert Levins gate 5",
        "city": "Oslo",
        "country": "NO",
        "postCode": "0154",
        "addressLine2": ""
    },
    "logo": "",
    "organizationNumber": "123456789",
    "companyName": "Fry Teknologi AS",
    "companyEmail": "developer@example.com",
    "mccCode": "5411",
    "email": "developer@example.com",
    "website": "https://example.com"
}'
```

</TabItem>
</Tabs>

### Step 6 - (Optional) Update Merchant Sales Unit

To update the merchant's sales unit, send the
[`PATCH:/v1/merchants/:merchantSerialNumber`](https://developer.vippsmobilepay.com/api/psp-signup#tag/Merchant/operation/patchMerchant)
request. Provide the Merchant Serial Number and the properties to be updated.

<Tabs
defaultValue="curl"
groupId="sdk-choice"
values={[
{label: 'curl', value: 'curl'},
{label: 'Postman', value: 'postman'},
]}>
<TabItem value="postman">

Use MSN for the new merchant and update the body in `Update Merchant Sales Unit` before sending the request.

```bash
Send request Update Merchant Sales Unit
```

Then, you can update the `Get Merchant by MSN` request with the new MSN and see if the changes were implemented.

</TabItem>
<TabItem value="curl">

```bash
curl https://apitest.vipps.no/merchant-management/psp/v1/merchants/{merchantSerialNumber} \
-H "Content-Type: application/json" \
-H "Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1Ni <truncated>" \
-H "Ocp-Apim-Subscription-Key: <This is the PSP's key, and is the same for all the PSP's merchants. Keep it secret.>" \
-H 'Idempotency-Key: cac4fd6c-6020-4570-aec9-0a249d1ac64a' \
-X PATCH \
--data '{
    "email": "developer.new@example.new.com",
    "website": "https://example.new.com"
    }'
```

</TabItem>
</Tabs>

## Next steps

See the [PSP Signup API guide](.) to read about the concepts and details.
