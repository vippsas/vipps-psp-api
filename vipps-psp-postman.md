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
   - `Ocp-Apim-Subscription-Key`
   - `makePaymentUrl`
   - `customerMobileNumber`
   - `PSP-ID`
   - `merchantSerialNumber` - optional

### Step 4: Run tests

1. Send request `Get Access Token`.


## Questions?

We're always happy to help with code or other questions you might have!

Please create an [issue](https://github.com/vippsas/vipps-psp-api/issues),
a [pull request](https://github.com/vippsas/vipps-psp-api/pulls),
or [contact us](https://github.com/vippsas/vipps-developers/blob/master/contact.md).

Sign up for our [Technical newsletter for developers](https://github.com/vippsas/vipps-developers/tree/master/newsletters).
