<!-- START_METADATA
---
title: eCom API checklist
sidebar_label: Checklist
sidebar_position: 35
description: Checklist for full integration with the PSP API.
pagination_next: null
pagination_prev: null
---
END_METADATA -->

# Checklist

## Endpoints to integrate

Integrate _all_ the [API endpoints](https://developer.vippsmobilepay.com/api/psp). For examples of requests and responses, see the [Postman collection](/tools/vipps-psp-v3-api-postman-collection.json)
and [environment](https://github.com/vippsas/vipps-developers/blob/master/tools/vipps-api-global-postman-environment.json).

| Endpoint | Comment |
|----------|---------|
| Initiate a PSP payment | [`POST:/v3/psppayments/init`](https://developer.vippsmobilepay.com/api/psp#tag/Vipps-PSP-API/operation/initiatePaymentV3UsingPOST) |
| Update the status of the PSP transaction | [`POST:/v3/psppayments/updatestatus`](https://developer.vippsmobilepay.com/api/psp#tag/Vipps-PSP-API/operation/updatestatusUsingPOST) |
| Get the details of the PSP payment | [`GET:/v3/psppayments/{pspTransactionId}/details`](https://developer.vippsmobilepay.com/api/psp#tag/Vipps-PSP-API/operation/getPSPPaymentDetailsUsingGET) |
| On the merchant side, be able to process EMVCo tokens | [`POST:makePaymentUrl`](https://developer.vippsmobilepay.com/api/psp#tag/Endpoints-required-by-Vipps-from-the-PSP/operation/makePaymentV3UsingPOST) |
| Delete a payment agreement. For recurring only(Currently on freeze pending PSD2 migration) | [`POST:/v3/psppayments/payments`](https://developer.vippsmobilepay.com/api/psp#tag/Vipps-PSP-API/operation/processPaymentOnTokenV3) |
| Get card data to process a payment with token. For recurring only (Currently on freeze pending PSD2 migration) | [`DELETE:/v3/psppayments/agreements`](https://developer.vippsmobilepay.com/api/psp#tag/Vipps-PSP-API/operation/deletePSPPaymenAgreementUsingDELETE) |
| Respond with correct error information  | [`POST:makePaymentUrl`](https://developer.vippsmobilepay.com/api/psp#tag/Endpoints-required-by-Vipps-from-the-PSP/operation/makePaymentV3UsingPOST).  See [error codes list](vipps-psp-api.md#error-codes) for possible responses.|

## Avoid integration pitfalls

| Action | Comment   |
|--------|-----------|
| Support soft decline step-ups according to [3DS specifications](vipps-psp-api.md#psd2-compliance-and-secure-customer-authentication-sca). |  In case of a soft decline (when the issuer requires 3DS), the PSP must host a 3DSecure session and must provide the URL to Vipps. See [PSD2 Compliance and Secure Customer Authentication (SCA)](./vipps-psp-api.md#psd2-compliance-and-secure-customer-authentication-sca). |
| Do not rely on `pspRedirectUrl`  |  Some users may close Vipps immediately after seeing the payment confirmation, therefore not being "redirected" back to the merchant. Because of this, it is important for the merchant and the PSP to _not_ base their transaction logic on the user reaching the `pspRedirectUrl`. See [PSP Payment Sequence](vipps-psp-api.md#psp-payment-sequence). |
| Provide reserve, capture, and refund information to Vipps | The PSP provides information of every `capture` and `refund` to Vipps (not just `reserve`) |
| Follow design guidelines     | The Vipps branding must be according to the [Vipps design guidelines](https://developer.vippsmobilepay.com/docs/design-guidelines).|

## Live flow

1. The PSP completes all checklist items.
2. The PSP [contacts Vipps](https://developer.vippsmobilepay.com/docs/contact) with test IDs (`pspTransactionId`, `merchantOrderId`) in the [Vipps test environment](https://developer.vippsmobilepay.com/docs/test-environment), showing that all checklist items have been fulfilled.
   * A complete order including `Reserve`, `Capture` and `Refund`, that has been updated with [`POST:/v3/psppayments/updatestatus`](https://developer.vippsmobilepay.com/api/psp#tag/Vipps-PSP-API/operation/updatestatusUsingPOST).
   * A complete order including `Cancel`, that has been updated with [`POST:/v3/psppayments/updatestatus`](https://developer.vippsmobilepay.com/api/psp#tag/Vipps-PSP-API/operation/updatestatusUsingPOST).
   * One ID for each of the [error codes](vipps-psp-api.md#errors).
       * Some codes like 85 aren't applicable for all systems, please provide short description for why each code does not apply.
3. The PSP [contacts Vipps](https://developer.vippsmobilepay.com/docs/contact) to verify the integration in the production environment:
   * At least one IDs for orders with each of the following statuses: `Capture`, `Refund`, `Cancel`.
   * At least 3 IDs for orders with different [error codes](vipps-psp-api.md#errors).
4. The PSP goes live 🎉
