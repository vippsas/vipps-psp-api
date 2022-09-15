<!-- START_METADATA
---
title: Checklist
sidebar_position: 35
---
END_METADATA -->

# Vipps PSP API Checklist

Document version 2.1.0

For examples of requests and responses, see the [Postman collection](./tools/vipps-psp-v3-api-postman-collection.json)
and [environment](./tools/vipps-psp-v3-api-postman-environment.json).

## Checklist for Full integration

- [ ] Integrate _all_ the [API endpoints](https://vippsas.github.io/vipps-psp-api/)
    - [ ] [`POST:/v3/psppayments/init`](https://vippsas.github.io/vipps-psp-api/#/Vipps_PSP_API/initiatePaymentV2UsingPOST)
    - [ ] [`POST:/v3/psppayments/updatestatus`](https://vippsas.github.io/vipps-psp-api/#/Vipps_PSP_API/updatestatusUsingPOST)
    - [ ] [`GET:/v3/psppayments/{pspTransactionId}/details`](https://vippsas.github.io/vipps-psp-api/#/Vipps_PSP_API/getPSPPaymentDetailsUsingGET)
    - [ ] On the merchant side, be able to process EMVCo tokens: [`POST:makePaymentUrl`](https://vippsas.github.io/vipps-psp-api/#/Endpoints_required_by_Vipps_from_the_PSP/makePaymentSwaggerUsingPOST)
    - [ ] For recurring only(Currently on freeze pending PSD2 migration): [`POST:/v3/psppayments/payments`](https://vippsas.github.io/vipps-psp-api/#/Vipps_PSP_API/processPaymentOnToken)
    - [ ] For recurring only (Currently on freeze pending PSD2 migration): [`DELETE:/v3/psppayments/agreements`](https://vippsas.github.io/vipps-psp-api/#/Vipps_PSP_API/deletePSPPaymenAgreementUsingDELETE)
- [ ] Respond with correct error information to [`POST:makePaymentUrl`](https://vippsas.github.io/vipps-psp-api/#/Endpoints_required_by_Vipps_from_the_PSP/makePaymentSwaggerUsingPOST). See [error codes list](https://github.com/vippsas/vipps-psp-api/blob/master/vipps-psp-api.md#error-codes) for possible responses.

## Live flow

1. The PSP completes all checklist items.
2. The PSP [contacts Vipps](https://github.com/vippsas/vipps-developers/blob/master/contact.md) with test IDs (`pspTransactionId`, `merchantOrderId`) in the [Vipps test environment](https://github.com/vippsas/vipps-developers#the-vipps-test-environment-mt), showing that all checklist items have been fulfilled.
    - A complete order including `Reserve`, `Capture` and `Refund`, that has been updated with [`POST:/v3/psppayments/updatestatus`](https://vippsas.github.io/vipps-psp-api/#/Vipps_PSP_API/updatestatusUsingPOST).
    - A complete order including `Cancel`, that has been updated with [`POST:/v3/psppayments/updatestatus`](https://vippsas.github.io/vipps-psp-api/#/Vipps_PSP_API/updatestatusUsingPOST).
    - One ID for each of the [error codes](https://github.com/vippsas/vipps-psp-api/blob/master/vipps-psp-api.md#errors).
        - Some codes like 85 aren't applicable for all systems, please provide short description for why each code does not apply.
3. The PSP [contacts Vipps](https://github.com/vippsas/vipps-developers/blob/master/contact.md) to verify the integration in the production environment:
    - At least one IDs for orders with each of the following statuses: `Capture`, `Refund`, `Cancel`.
    - At least 3 IDs for orders with different [error codes](https://github.com/vippsas/vipps-psp-api/blob/master/vipps-psp-api.md#errors).
4. The PSP goes live 🎉

## Questions?

We're always happy to help with code or other questions you might have!
Please create an [issue](https://github.com/vippsas/vipps-psp-api/issues),
a [pull request](https://github.com/vippsas/vipps-psp-api/pulls),
or [contact us](https://github.com/vippsas/vipps-developers/blob/master/contact.md).

Sign up for our [Technical newsletter for developers](https://github.com/vippsas/vipps-developers/tree/master/newsletters).
