<!-- START_METADATA
---
title: Quick start for the PSP Merchant Sign up
sidebar_label: Quick start
sidebar_position: 15
description: Quick steps for getting started with the PSP Merchant Sign up.
pagination_next: null
pagination_prev: null
---
END_METADATA -->

# Quick start for PSP Merchant Sign up

Use the PSP Merchant Signup API to get information about merchants, create a new sales unit, or update the sales unit details.

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
* `makePaymentUrl` - PSP URL used by Vipps to send the card data, needed only for `Initiate a PSP Payment`.
* `pspRedirectUrl` - Redirect URL which the user is redirected to after approving/rejecting the payment.
* `PSP-ID` - PSP ID provided by Vipps, needed for `Initiate a PSP Payment`, `Update Status`, and `Get Details`.
* `idempotency_key` - Unique request ID, needed only for `Create new Merchant Sales Unit`.

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
-H "Merchant-Serial-Number: 123456" \
-H "Vipps-System-Name: acme" \
-H "Vipps-System-Version: 3.1.2" \
-H "Vipps-System-Plugin-Name: acme-webshop" \
-H "Vipps-System-Plugin-Version: 4.5.6" \
-X POST \
--data ''
```

</TabItem>
</Tabs>

The property `access_token` should be used for all other API requests in the `Authorization` header as the Bearer token.

### Step 3 - Get all merchants

Send `Get all Merchants` for a JSON response showing all the merchants and their information. See [`GET:/v1/merchants`](https://developer.vippsmobilepay.com/api/psp-signup#tag/Merchant/operation/getMerchants).

### Step 4 - Get Merchant by MSN

Use [`GET:/v1/merchants/:merchantSerialNumber`](https://developer.vippsmobilepay.com/api/psp-signup#tag/Merchant/operation/getMerchant)
for information about a specific merchant. Supply the MSN.

### Step 5 - Create new Merchant Sales Unit

Send [`POST:/v1/merchants`](https://developer.vippsmobilepay.com/api/psp-signup#tag/Merchant/operation/addMerchant). Notice the `merchantSerialNumber` is returned.

### Step 6 - Update Merchant Sales Unit

Run [`PATCH:/v1/merchants/:merchantSerialNumber`](https://developer.vippsmobilepay.com/api/psp-signup#tag/Merchant/operation/patchMerchant) with the Merchant Serial Number.

## Next steps

See the [PSP API guide](vipps-psp-api.md) to read about the concepts and details.
