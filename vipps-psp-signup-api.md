---
title: PSP Signup API guide
sidebar_label: PSP Signup API guide
sidebar_position: 25
description: Find technical details about integrating with the PSP Signup API.
pagination_prev: Null
pagination_next: Null
---

# PSP Signup API guide

The PSP Signup API allows PSPs to onboard and manage their merchants.

This API is the only way to sign up non-Norwegian merchants.

The API specification is available at:
[PSP Signup API Reference](https://developer.vippsmobilepay.com/api/psp-signup).

API version: 3.0

## Introduction

A PSP can use its existing API keys to access this API, and perform the following:

* List one or all merchants under them
* Create a new merchant under them
* Update an existing merchant

See the
[PSP API Guide](https://developer.vippsmobilepay.com/docs/APIs/psp-api/vipps-psp-api)
for more.

## Information shown in Vipps

The following are the screens in the Vipps app, where the information about
the merchant that was provided by the PSP is rendered to the end user.

| Item               | Description                                           |
| ------------------ | ----------------------------------------------------- |
| Merchant           | The name of the merchant's sales unit                 |
| #23412             | Merchant serial number, the sales unit's ID           |
| Merchant AS        | The name of the merchant                              |
| Product name       | The name of the product being paid for                |
| Order ID / Description | The orderId, provided by the merchant             |
| Transaction ID     | The internal Vipps ID for the transaction             |
| butikken.no        | The merchant's website                                |

### The payment screen

![Payment Screen](./docs/signup/payment.png)

### The receipt screen

![Receipt Screen](./docs/signup/receipt.png)

## Get all merchants

For a JSON response showing all the merchants and their information, send
[`GET:/v1/merchants`](https://developer.vippsmobilepay.com/api/psp-signup#tag/Merchant/operation/getMerchants).

Example response (see the API specification for details):

```json
{
   "merchants":[
      {
         "organizationNumber":"123456789",
         "companyName":"Vipps AS",
         "companyEmail":"user@example.com",
         "merchantSerialNumber":"123456",
         "name":"Example AS",
         "mccCode":"5411",
         "status":"ACTIVE",
         "logoUrl":"https://product-logos.vipps.no/product-logos/896bed0e-fc0d-4edf-952c-aeb710581c4e.png",
         "email":"user@example.com",
         "website":"https://example.com",
         "createdAt":"2019-01-01T00:00:00Z",
         "updatedAt":"2019-01-01T00:00:00Z",
         "address":{

         }
      }
   ]
}
```

## Get information about a specific merchant

For information about a specific merchant, send
[`GET:/v1/merchants/:merchantSerialNumber`](https://developer.vippsmobilepay.com/api/psp-signup#tag/Merchant/operation/getMerchant).
Supply the MSN for a merchant in your list of merchants.

Example response (see the API specification for details):

```json
{
   "organizationNumber":"123456789",
   "companyName":"Vipps AS",
   "companyEmail":"user@example.com",
   "merchantSerialNumber":"123456",
   "name":"Example AS",
   "mccCode":"5411",
   "status":"ACTIVE",
   "logoUrl":"https://product-logos.vipps.no/product-logos/896bed0e-fc0d-4edf-952c-aeb710581c4e.png",
   "email":"user@example.com",
   "website":"https://example.com",
   "createdAt":"2019-01-01T00:00:00Z",
   "updatedAt":"2019-01-01T00:00:00Z",
   "address":{
      "addressLine1":"Robert Levins gate 5",
      "addressLine2":"Att: Rune Garborg",
      "city":"Oslo",
      "postCode":"0154",
      "country":"NO"
   }
}
```

**Please note:** The `mccCode` may be returned as `null`, this is expected.

## Create a new merchant sales unit

To create new merchant sales unit, send
[`POST:/v1/merchants`](https://developer.vippsmobilepay.com/api/psp-signup#tag/Merchant/operation/addMerchant).

For Norway: The orgno must be 9 digits without spaces, the merchant
must be active in [Brønnøysundregistrene](https://www.brreg.no)
and the orgno must be for the main entity ("hovedenhet"),
not a sub entity ("underenhet").
For other countries: The orgno, address, etc. is validated as much
as practically possible.

Example request (see the API specification for details):

```json
{
   "organizationNumber":"123456789",
   "name":"Example AS",
   "mccCode":"5411",
   "logo":"VGhlIGltYWdlIGdvZXMgaGVyZQ== (truncated)",
   "email":"user@example.com",
   "website":"https://example.com",
   "address":{
      "addressLine1":"Robert Levins gate 5",
      "addressLine2":"Att: Rune Garborg",
      "city":"Oslo",
      "postCode":"0154",
      "country":"NO"
   },
   "companyName":"Vipps AS",
   "companyEmail":"user@example.com"
}
```

## Update an existing merchant sales unit

To update a merchant sales unit, send
[`PATCH:/v1/merchants/:merchantSerialNumber`](https://developer.vippsmobilepay.com/api/psp-signup#tag/Merchant/operation/patchMerchant).
Provide the MSN for the merchant and update the details in the body section.
