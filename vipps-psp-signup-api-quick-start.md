<!-- START_METADATA
---
title: Quick start for the PSP Merchant Sign up
sidebar_label: Signup Quick start
sidebar_position: 50
description: Quick steps for getting started with the PSP Merchant Signup.
pagination_next: null
pagination_prev: null
---

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

Note that the requests in this PSP API require your PSP subscription key.

### Step 2 - Authentication

For all the following, you will need an `access_token` from:
[`POST:/accesstoken/get`](https://developer.vippsmobilepay.com/api/access-token#tag/Authorization-Service/operation/fetchAuthorizationTokenUsingPost).
This provides you with access to the API.

```bash
curl https://apitest.vipps.no/accessToken/get \
-H "client_id: YOUR-CLIENT-ID" \
-H "client_secret: YOUR-CLIENT-SECRET" \
-H "Ocp-Apim-Subscription-Key: YOUR-SUBSCRIPTION-KEY" \
-X POST \
--data ''
```

The property `access_token` should be used for all other API requests in the `Authorization` header as the Bearer token.

### Step 3 - Get all merchants

Send
[`GET:/merchant-management/psp/v1/merchants`](https://developer.vippsmobilepay.com/api/psp-signup#tag/Merchant/operation/getMerchants)
for a JSON response showing all the merchants and their information.

```bash
curl https://apitest.vipps.no/merchant-management/psp/v1/merchants \
-H "Content-Type: application/json" \
-H "Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1Ni <truncated>" \
-H "Ocp-Apim-Subscription-Key: <This is the PSP's key, and is the same for all the PSP's merchants. Keep it secret.>" \
-X GET
```

### Step 4 - Get Merchant by MSN

Use
[`GET:/merchant-management/psp/v1/merchants/{merchantSerialNumber}`](https://developer.vippsmobilepay.com/api/psp-signup#tag/Merchant/operation/getMerchant)
for information about a specific merchant. Supply the MSN.

```bash
curl https://apitest.vipps.no/merchant-management/psp/v1/merchants/{merchantSerialNumber} \
-H "Content-Type: application/json" \
-H "Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1Ni <truncated>" \
-H "Ocp-Apim-Subscription-Key: <This is the PSP's key, and is the same for all the PSP's merchants. Keep it secret.>" \
-X GET
```

### Step 5 - Create new Merchant Sales Unit

Send
[`POST:`](https://developer.vippsmobilepay.com/api/psp-signup#tag/Merchant/operation/addMerchant).
Notice the `merchantSerialNumber` is returned.

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

### Step 6 - (Optional) Update Merchant Sales Unit

To update the merchant's sales unit, send the
[`PATCH:/v1/merchants/:merchantSerialNumber`](https://developer.vippsmobilepay.com/api/psp-signup#tag/Merchant/operation/patchMerchant)
request. Provide the Merchant Serial Number and the properties to be updated.

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

## Next steps

See the [PSP Signup API guide](vipps-psp-signup-api.md) to read about the concepts and details.
