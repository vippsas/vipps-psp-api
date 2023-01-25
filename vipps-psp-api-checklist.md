<!-- START_METADATA
---
title: Checklist
sidebar_position: 35
---
END_METADATA -->

# Vipps PSP API Checklist

For examples of requests and responses, see the [Postman collection](./tools/vipps-psp-v3-api-postman-collection.json)
and [environment](https://github.com/vippsas/vipps-developers/blob/master/tools/vipps-api-global-postman-environment.json).

## Checklist for full integration

- [ ] Integrate _all_ the [API endpoints](https://vippsas.github.io/vipps-developer-docs/api/psp)
    - [ ] [`POST:/v3/psppayments/init`](https://vippsas.github.io/vipps-developer-docs/api/psp#tag/Vipps-PSP-API/operation/initiatePaymentV3UsingPOST)
    - [ ] [`POST:/v3/psppayments/updatestatus`](https://vippsas.github.io/vipps-developer-docs/api/psp#tag/Vipps-PSP-API/operation/updatestatusUsingPOST)
    - [ ] [`GET:/v3/psppayments/{pspTransactionId}/details`](https://vippsas.github.io/vipps-developer-docs/api/psp#tag/Vipps-PSP-API/operation/getPSPPaymentDetailsUsingGET)
    - [ ] On the merchant side, be able to process EMVCo tokens: [`POST:makePaymentUrl`](https://vippsas.github.io/vipps-developer-docs/api/psp#tag/Endpoints-required-by-Vipps-from-the-PSP/operation/makePaymentV3UsingPOST)
    - [ ] For recurring only(Currently on freeze pending PSD2 migration): [`POST:/v3/psppayments/payments`](https://vippsas.github.io/vipps-developer-docs/api/psp#tag/Vipps-PSP-API/operation/processPaymentOnTokenV3)
    - [ ] For recurring only (Currently on freeze pending PSD2 migration): [`DELETE:/v3/psppayments/agreements`](https://vippsas.github.io/vipps-developer-docs/api/psp#tag/Vipps-PSP-API/operation/deletePSPPaymenAgreementUsingDELETE)
- [ ] Respond with correct error information to [`POST:makePaymentUrl`](https://vippsas.github.io/vipps-developer-docs/api/psp#tag/Endpoints-required-by-Vipps-from-the-PSP/operation/makePaymentV3UsingPOST). See [error codes list](vipps-psp-api.md#error-codes) for possible responses.
- [ ] Support soft decline, or handle step ups according to our [3DS specifications](vipps-psp-api.md#psd2-compliance-and-secure-customer-authentication-sca)
- [ ] Avoid Integration pitfalls
    - [ ] The PSP _must not_ rely on `pspRedirectUrl` alone, see [PSP Payment Sequence](vipps-psp-api.md#summary)
    - [ ] The PSP provides information of every `capture` and `refund` to Vipps (not just `reserve`)
    - [ ] The Vipps branding must be according to the [Vipps design guidelines](https://github.com/vippsas/vipps-design-guidelines)

## Live flow

1. The PSP completes all checklist items.
2. The PSP [contacts Vipps](https://vippsas.github.io/vipps-developer-docs/docs/vipps-developers/contact) with test IDs (`pspTransactionId`, `merchantOrderId`) in the [Vipps test environment](https://vippsas.github.io/vipps-developer-docs/docs/vipps-developers/test-environment), showing that all checklist items have been fulfilled.
    - A complete order including `Reserve`, `Capture` and `Refund`, that has been updated with [`POST:/v3/psppayments/updatestatus`](https://vippsas.github.io/vipps-developer-docs/api/psp#tag/Vipps-PSP-API/operation/updatestatusUsingPOST).
    - A complete order including `Cancel`, that has been updated with [`POST:/v3/psppayments/updatestatus`](https://vippsas.github.io/vipps-developer-docs/api/psp#tag/Vipps-PSP-API/operation/updatestatusUsingPOST).
    - One ID for each of the [error codes](vipps-psp-api.md#errors).
        - Some codes like 85 aren't applicable for all systems, please provide short description for why each code does not apply.
3. The PSP [contacts Vipps](https://vippsas.github.io/vipps-developer-docs/docs/vipps-developers/contact) to verify the integration in the production environment:
    - At least one IDs for orders with each of the following statuses: `Capture`, `Refund`, `Cancel`.
    - At least 3 IDs for orders with different [error codes](vipps-psp-api.md#errors).
4. The PSP goes live 🎉
