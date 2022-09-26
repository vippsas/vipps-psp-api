<!-- START_METADATA
---
title: Quick start
sidebar_position: 15
---
END_METADATA -->

# Quick start

<!-- START_TOC -->

## Table of Contents

* [Postman](#postman)
  * [Prerequisites](#prerequisites)
  * [Step 1: Get the Postman collection](#step-1-get-the-postman-collection)
  * [Step 2: Import the Postman environment](#step-2-import-the-postman-environment)
  * [Step 3: Set up Postman environment](#step-3-set-up-postman-environment)
* [Make API calls](#make-api-calls)
  * [PSP Payments and details](#psp-payments-and-details)
  * [PSP Merchant Sign up](#psp-merchant-sign-up)
* [Questions?](#questions)

<!-- END_TOC -->

Document version 1.0.3.

## Postman

### Prerequisites

* If you are not familiar with Postman, review
[Getting started with Postman](https://vippsas.github.io/vipps-developer-docs/docs/vipps-developers/vipps-quick-start-guides#getting-started-with-postman).

* You will need a merchant test account.
If you haven't already set this up, see
[Test Merchants](https://vippsas.github.io/vipps-developer-docs/docs/vipps-developers/vipps-test-environment#test-merchants).

### Step 1: Get the Postman collection

Import the collection by following the steps below:

1. Click `Import` in the upper-left corner.
2. Import the [vipps-psp-v3-api-postman-collection.json](./tools/vipps-psp-v3-api-postman-collection.json) file.

### Step 2: Import the Postman environment

1. Click `Import` in the upper-left corner.
2. Import the [vipps-psp-v3-api.postman_environment.json](./tools/vipps-psp-v3-api-postman-environment.json) file.

### Step 3: Set up Postman environment

1. Click the down arrow, next to the "eye" icon in the top-right corner, and select the environment you have imported.
2. Click the "eye" icon and, in the dropdown window, click `Edit` in the top-right corner.
3. Fill in the `Current Value` for the following fields to get started.
   - `client_id` - Merchant key is required for getting the access token.
   - `client_secret` - Merchant key is required for getting the access token.
   - `Ocp-Apim-Subscription-Key-PSP` - PSP subscription key is required for all requests.
   - `customerMobileNumber` - Test mobile number, needed only for `Initiate a PSP Payment`.
   - `makePaymentUrl` - PSP URL used by Vipps to send the card data, needed only for `Initiate a PSP Payment`.
   - `pspRedirectUrl` - Redirect URL which the user is redirected to after approving/rejecting the payment.
   - `merchantSerialNumber` - Required for `Get Merchant by MSN`.
   - `PSP-ID` - PSP ID provided by Vipps, needed for `Initiate a PSP Payment`, `Update Status`, and `Get Details`.
   - `idempotency_key` - Unique request id, needed only for `Create new Merchant Sale Unit`.

Note that the calls in this PSP API require your PSP subscription key.

## Make API calls

For all of the following, you will start by sending request `Get Access Token`.
This provides you with access to the API.

   The access token is valid for 1 hour in the test environment
   and 24 hours in the production environment.

See the
[API reference](https://vippsas.github.io/vipps-developer-docs/api/psp)
for details about the calls.

### PSP Payments and details

1. Send request `Initiate a PSP Payment`. Ctrl+click on the link that appears and it will take you to the website where you can enter your test phone number and complete the payment authorization in the Vipps app in your mobile test environment. The `pspTransactionId` in the environment is updated automatically. See [`POST:/v3/psppayments/init/`](https://vippsas.github.io/vipps-developer-docs/api/psp#tag/Vipps-PSP-API/operation/initiatePaymentV3UsingPOST).
2. Send request  `Get Details` for details about the transaction. See [`GET:/v3/psppayments/{pspTransactionId}/details`](https://vippsas.github.io/vipps-developer-docs/api/psp#tag/Vipps-PSP-API/operation/getPSPPaymentDetailsUsingGET).
3. Send request `Update Status`. This is an example of what the PSP sends Vipps to update on the status of the payment. See [`POST:/v3/psppayments/updatestatus`](https://vippsas.github.io/vipps-developer-docs/api/psp#tag/Vipps-PSP-API/operation/updatestatusUsingPOST).

### PSP Merchant Sign up

1. Send `Get all Merchants` for a json response showing all the merchants and their information. See [`GET:/v1/merchants`](https://vippsas.github.io/vipps-developer-docs/api/psp-signup#tag/Merchant/operation/getMerchants).
2. Send `Get Merchant by MSN` for information about a specific merchant. Supply the MSN for a merchant in the list of all merchants. See [`GET:/v1/merchants/:merchantSerialNumber`](https://vippsas.github.io/vipps-developer-docs/api/psp-signup#tag/Merchant/operation/getMerchant).
3. Send `Create new Merchant Sale Unit`. Notice the merchantSerialNumber is provided back.  See [`POST:/v1/merchants`](https://vippsas.github.io/vipps-developer-docs/api/psp-signup#tag/Merchant/operation/addMerchant).
4. Open `Update Merchant Sale Unit`. Use MSN for the new merchant and update the body before sending the request.
   Then, you can update the `Get Merchant by MSN` request with the new MSN and see if the changes were implemented.
    See [`PATCH:/v1/merchants/:merchantSerialNumber`](https://vippsas.github.io/vipps-developer-docs/api/psp-signup#tag/Merchant/operation/patchMerchant).

## Questions?

We're always happy to help with code or other questions you might have!

Please create an [issue](https://github.com/vippsas/vipps-psp-api/issues),
a [pull request](https://github.com/vippsas/vipps-psp-api/pulls),
or [contact us](https://github.com/vippsas/vipps-developers/blob/master/contact.md).

Sign up for our [Technical newsletter for developers](https://github.com/vippsas/vipps-developers/tree/master/newsletters).
