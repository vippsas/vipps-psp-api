# Vipps PSP API Checklist

Document version 1.1.0

**Important:** V1 will be phased out shortly, and it is therefore important that all merchants are migrated over to V2 as soon as the integration is verified by Vipps and you are ready to go live.

For examples of requests and responses, see the Postman collection in [tools](tools/)

# Checklist for Full integration
- [ ] Integrate _all_ the [API endpoints](https://vippsas.github.io/vipps-psp-api/)
    - [ ] [`POST:/v2/psppayments/init`](https://vippsas.github.io/vipps-psp-api/#/Vipps_PSP_API/initiatePaymentV2UsingPOST)
    - [ ] [`POST:/v2/psppayments/updatestatus`](https://vippsas.github.io/vipps-psp-api/#/Vipps_PSP_API/updatestatusUsingPOST)
    - [ ] [`GET:/v2/psppayments/{pspTransactionId}/details`](https://vippsas.github.io/vipps-psp-api/#/Vipps_PSP_API/getPSPPaymentDetailsUsingGET)
    - [ ] On the merchant side: [`POST:makePaymentUrl`](https://vippsas.github.io/vipps-psp-api/#/Endpoints_required_by_Vipps_from_the_PSP/makePaymentSwaggerUsingPOST)
    - [ ] For recurring only: [`POST:/v2/psppayments/payments`](https://vippsas.github.io/vipps-psp-api/#/Vipps_PSP_API/processPaymentOnToken)
    - [ ] For recurring only: [`DELETE:/v2/psppayments/payments`](https://vippsas.github.io/vipps-psp-api/#/Vipps_PSP_API/deletePSPPaymenAgreementUsingDELETE)
- [ ] Respond with correct error information to [`POST:makePaymentUrl`](https://vippsas.github.io/vipps-psp-api/#/Endpoints_required_by_Vipps_from_the_PSP/makePaymentSwaggerUsingPOST). See [error codes list](https://github.com/vippsas/vipps-psp-api/blob/master/vipps-psp-api.md#error-codes) for possible responses.
- [ ] Avoid Integration pitfalls
    - [ ] The PSP _must not_ rely on `pspRedirectUrl` alone, see [PSP Payment Sequence](vipps-psp-api.md#summary)
    - [ ] The PSP provides information of every `capture` and `refund` to Vipps (not just `reserve`)
    - [ ] The Vipps branding must be according to the [Vipps design guidelines](https://github.com/vippsas/vipps-design-guidelines)


# Live flow
1. The PSP sends public keys to pspbestilling@vipps.no, see [public key](https://github.com/vippsas/vipps-psp-api/blob/master/vipps-psp-api.md#public-key).
2. The PSP completes all checklist items.
3. The PSP [contacts Vipps](https://github.com/vippsas/vipps-developers/blob/master/contact.md) with test IDs (`pspTransactionId`, `merchantOrderId`) in the [Vipps test environment](https://github.com/vippsas/vipps-developers#the-vipps-test-environment-mt), showing that all checklist items have been fulfilled.
    - A complete order including `Reserve`, `Capture` and `Refund`, that has been updated with [`POST:/v2/psppayments/updatestatus`](https://vippsas.github.io/vipps-psp-api/#/Vipps_PSP_API/updatestatusUsingPOST).
    - A complete order including `Cancel`, that has been updated with [`POST:/v2/psppayments/updatestatus`](https://vippsas.github.io/vipps-psp-api/#/Vipps_PSP_API/updatestatusUsingPOST).
    - One ID for each of the [error codes](https://github.com/vippsas/vipps-psp-api/blob/master/vipps-psp-api.md#errors).
        - Some codes like 85 aren't applicable for all systems, please provide short description for why each code does not apply.
4. The PSP [contacts Vipps](https://github.com/vippsas/vipps-developers/blob/master/contact.md) to verify the integration in the production environment:
    - At least one IDs for orders with each of the following statuses: `Capture`, `Refund`, `Cancel`.
    - At least 3 IDs for orders with different [error codes](https://github.com/vippsas/vipps-psp-api/blob/master/vipps-psp-api.md#errors).
5. The PSP goes live 🎉

## Questions?

We're always happy to help with code or other questions you might have!
Please create an [issue](https://github.com/vippsas/vipps-psp-api/issues),
a [pull request](https://github.com/vippsas/vipps-psp-api/pulls),
or [contact us](https://github.com/vippsas/vipps-developers/blob/master/contact.md).

Sign up for our [Technical newsletter for developers](https://github.com/vippsas/vipps-developers/tree/master/newsletters).
