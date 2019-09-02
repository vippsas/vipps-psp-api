# Vipps PSP API Checklist

Document version 1.0.0

**Important:** V1 will be phased out shortly, and it is therefore important that all merchants are migrated over to V2 as soon as the integration is verified by Vipps and you are ready to go live.

# Overall flow for PSP integrations

1. The PSP completes the direct integration with the [Vipps PSP API v2](https://github.com/vippsas/vipps-psp-api)
  - This includes _all_ the [API endpoints](https://vippsas.github.io/vipps-psp-api/)
    - [`POST:/v2/psppayments/init`](https://vippsas.github.io/vipps-psp-api/#/Vipps_PSP_API/initiatePaymentV2UsingPOST)
    - [`POST:/v2/psppayments/updatestatus`](https://vippsas.github.io/vipps-psp-api/#/Vipps_PSP_API/updatestatusUsingPOST)
    - [`GET:/v2/psppayments/{pspTransactionId}/details`](https://vippsas.github.io/vipps-psp-api/#/Vipps_PSP_API/getPSPPaymentDetailsUsingGET)
    - On the merchant side: [`POST:makePaymentUrl`](https://vippsas.github.io/vipps-psp-api/#/Endpoints_required_by_Vipps_from_the_PSP/makePaymentSwaggerUsingPOST)
    - For recurring only: [`POST:/v2/psppayments/payments`](https://vippsas.github.io/vipps-psp-api/#/Vipps_PSP_API/processPaymentOnToken)
    - For recurring only: [`DELETE:/v2/psppayments/payments`](https://vippsas.github.io/vipps-psp-api/#/Vipps_PSP_API/deletePSPPaymenAgreementUsingDELETE)
  - The PSP _must not_ rely on "redirect" alone, see [PSP Payment Sequence](vipps-psp-api.md#summary)
  - The PSP provides information of every `capture` and `refund` to Vipps (not just `reserve`)
  - The Vipps branding must be according to the [Vipps design guidelines](https://github.com/vippsas/vipps-design-guidelines)
  - The PSP must provide specific error information to Vipps, such as:
    - `82 Refused by issuer`
    - `89 Not sufficient funds`
    - See [Error codes](https://github.com/vippsas/vipps-psp-api/blob/master/vipps-psp-api.md#error-codes)
    - See [Test data](https://github.com/vippsas/vipps-developers/tree/master/testdata)
  - For examples of requests and responses, see the Postman collection in [tools](tools/)
2. The PSP completes testing in the [Vipps test environment](https://github.com/vippsas/vipps-developers#the-vipps-test-environment-mt)
  - "Happy day": Valid card, confirmed in the Vipps app, completed payment
    - Initiate
    - Receive `makePaymentRequest`
    - Reserve
    - Capture and updateStatus
    - Refund and updateStatus
  - "Happy day", but with Cancel instead of Capture.
  - Insufficient funds
  - Card expired
  - Card rejected by issuer
3. The PSP completes one or more test transactions in the production environment
4. The PSP [contacts Vipps](https://github.com/vippsas/vipps-developers/blob/master/contact.md) to verify the integration in the production environment:
  - At least one order id for orders with each of the following statuses: capture, refund, cancel.
  - At least one order id for orders with each of the [error codes](https://github.com/vippsas/vipps-psp-api/blob/master/vipps-psp-api.md#error-codes) 
5. Vipps completes one or more transactions in the production environment
6. The PSP goes live 🎉

# Questions?

We're always happy to help with code or other questions you might have!
Please create an [issue](https://github.com/vippsas/vipps-ecom-api/issues),
a [pull request](https://github.com/vippsas/vipps-ecom-api/pulls),
or [contact us](https://github.com/vippsas/vipps-developers/blob/master/contact.md).
