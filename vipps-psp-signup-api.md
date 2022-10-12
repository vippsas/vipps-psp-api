<!-- START_METADATA
---
title: PSP Signup API Guide
sidebar_position: 25
---
END_METADATA -->

# PSP Signup API

The Vipps PSP Signup API allows PSPs to onboard and manage their merchants.

This API is the only way to sign up non-Norwegian merchants.

The API specification is available at:
[PSP Signup API Reference](https://vippsas.github.io/vipps-developer-docs/api/psp-signup).

API version: 3.0

Document version 1.0.2.

<!-- START_TOC -->

## Table of Contents

- [Introduction](#introduction)
- [Get all merchants](#get-all-merchants)
- [Get information about a specific merchant](#get-information-about-a-specific-merchant)
- [Create a new merchant sale unit](#create-a-new-merchant-sale-unit)
- [Update an existing merchant sale unit](#update-an-existing-merchant-sale-unit)
- [Proposals](#proposals)
  - [Recurring 3DS Update Card](#recurring-3ds-update-card)
- [Questions?](#questions)

<!-- END_TOC -->

## Introduction

The following are the screens in the Vipps app, where the information about the merchant that was provided by the PSP is rendered to the end user.

![Payment Screen](./docs/signup/payment.png)

![Receipt Screen](./docs/signup/receipt.png)

Some details of information shown in the screenshots:

| Item               | Description                                           |
| ------------------ | ----------------------------------------------------- |
| Merchant           | The name of the merchant's sale unit                  |
| #23412             | Merchant serial number, the sale unit's id            |
| Merchant AS        | The name of the merchant                              |
| Product name       | The name of the product being paid for                |
| Order ID / Description | The orderId, provided by the merchant             |
| Transaction ID     | The internal Vipps id for the transaction             |
| butikken.no        | Merchant website                                      |

A PSP can use their existing keys to access this APIs. They can perform the following:

- List one or all merchants under them
- Create a new merchant under them
- Update an existing merchant

## Get all merchants

For a json response showing all the merchants and their information, send [`GET:/v1/merchants`](https://vippsas.github.io/vipps-developer-docs/api/psp-signup#tag/Merchant/operation/getMerchants).

## Get information about a specific merchant

For information about a specific merchant, send
[`GET:/v1/merchants/:merchantSerialNumber`](https://vippsas.github.io/vipps-developer-docs/api/psp-signup#tag/Merchant/operation/getMerchant). Supply the MSN for a merchant in your list of merchants.

## Create a new merchant sale unit

To create new merchant sale unit, send [`POST:/v1/merchants`](https://vippsas.github.io/vipps-developer-docs/api/psp-signup#tag/Merchant/operation/addMerchant).

For Norway: The orgno must be 9 digits without spaces, the merchant
must be active in [Brønnøysundregistrene](https://www.brreg.no)
and the orgno must be for the main entity ("hovedenhet"),
not a sub entity ("underenhet").
For other countries: The orgno, address, etc is validated as much
as practically possible.

## Update an existing merchant sale unit

To update a merchant sale unit, send [`PATCH:/v1/merchants/:merchantSerialNumber`](https://vippsas.github.io/vipps-developer-docs/api/psp-signup#tag/Merchant/operation/patchMerchant).
Provide the MSN for the merchant and update the details in the body section.

## Proposals

### Recurring 3DS Update Card

The following is a proposal for triggering 3DS when a user changes the card attached to a PSP recurring agreement in the Vipps app.

A new optional `updateCardUrl` property would be added to the initiate call. This callback will be triggered when the user changes card with the following request;

```json
HEADER: "
        PSP-ID: C948DFD1546347568874C4DDC93A2E3C
        Merchant-Serial-Number: 123456
        User-Token: eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9
        "
{
  "agreementId": "7686f7788898767977",
  "cardData": "f0a29801b4#d4ff30e221fa2980ff30e2"
}
```

To which the PSP should respond with a `require3DS` boolean flag which, if true, will cause a 3DS session to be hosted in the Vipps app.

```json
{
  "require3DS": true,
  "url3dSecure": "https://psp.com/3ds/123123"
}
```

The Vipps app will the host a 3DS session from the PSP, once this session is completed
the PSP should redirect to `https://www.vipps.no/mobileintercept`.

Once the user completes the 3DS session the `updateCardUrl` will be called again.
The PSP should then only approve or deny the request.

## Questions?

We're always happy to help with code or other questions you might have!
Please create an [issue](https://github.com/vippsas/vipps-psp-api/issues),
a [pull request](https://github.com/vippsas/vipps-psp-api/pulls),
or [contact us](https://github.com/vippsas/vipps-developers/blob/master/contact.md).

Sign up for our [Technical newsletter for developers](https://github.com/vippsas/vipps-developers/tree/master/newsletters).
