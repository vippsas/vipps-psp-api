# Postman

[Postman](https://www.getpostman.com/) is a common tool for working with REST APIs.
We offer a [Postman Collection](https://www.getpostman.com/collection) to make development easier.
See the [Postman documentation](https://www.getpostman.com/docs/) for more information about using Postman.

By following the steps below, you can make calls to all the
endpoints, and see the full `request` and `response` for each call.

We also have a short [getting started guide to Postman](https://github.com/vippsas/vipps-developers/blob/master/postman-guide.md).

## Setting up Postman

### Step 1: Get the Postman Collection

Import the collection by following the steps below:

1. Click `Import` in the upper-left corner.
2. Import the [vipps-psp-v2-api-postman-collection.json](https://raw.githubusercontent.com/vippsas/vipps-psp-api/master/tools/vipps-psp-v2-api-postman-collection.json) file.

### Step 2: Import the Postman Environment

1. Click `Import` in the upper-left corner.
2. Import the [vipps-psp-v2-api.postman_environment.json](https://raw.githubusercontent.com/vippsas/vipps-psp-api/master/tools/vipps-psp-v2-api-postman-environment.json) file.

### Step 3: Setup Postman Environment

1. Click the down arrow, next to the "eye" icon in the top-right corner, and select the environment you have imported.
2. Click the "eye" icon and, in the dropdown window, click `Edit` in the top-right corner.
3. Fill in the `Current Value` for the following fields to get started.
   - `client_id`
   - `client_secret`
   - `Ocp-Apim-Subscription-Key-PSP`
   - `PSP-ID`
   - `makePaymentUrl`
   - `customerMobileNumber`
   - `merchantSerialNumber`
   - `amount`

Note that most of the calls in this PSP API require that you use your PSP key.

### Step 4: Run PSP Payment tests

1. Send request `Get Access Token`. This provides you with access to the API.
2. Send request `Initiate a PSP Payment`. Ctrl+click on the link that appears and it will take you to the website where
   you can enter your test phone number and complete the payment authorization in the Vipps app in your mobile test environment.
   The pspTransactionId in the environment is updated automatically.
3. Send `Get Details` for details about the transaction.
4. Send `Update Status` to set information about the status.


### Step 5: Run PSP Merchant Signup tests

1. Send `Get all Merchants` for a json response showing all the merchants and their information.
2. Send `Get Merchant by MSN` for information about a specific merchant. Supply the MSN for a merchant in the list of all merchants.
3. Send `Create new Merchant Sale Unit`. Notice the merchantSerialNumber is provided back.
4. Open `Update Merchant Sale Unit`. Use MSN for the new merchant and update the body before sending the request.
   Then, you can update the `Get Merchant by MSN` request with the new MSN and see if the changes were implemented.

## Questions?

We're always happy to help with code or other questions you might have!

Please create an [issue](https://github.com/vippsas/vipps-psp-api/issues),
a [pull request](https://github.com/vippsas/vipps-psp-api/pulls),
or [contact us](https://github.com/vippsas/vipps-developers/blob/master/contact.md).

Sign up for our [Technical newsletter for developers](https://github.com/vippsas/vipps-developers/tree/master/newsletters).
