# Vipps PSP API Checklist

Document version 0.2.0

**Important:** V1 will be phased out shortly, and it is therefore important that all merchants are migrated over to V2 as soon as the integration is verified by Vipps and you are ready to go live.

# Overall flow for PSP integrations

1. The PSP completes the direct integration with the [Vipps PSP API v2](https://github.com/vippsas/vipps-psp-api)
  - This includes _all_ the [API endpoints](https://vippsas.github.io/vipps-psp-api/)
  - The PSP must not rely on "redirect" alone, see [PSP Payment Sequence](vipps-psp-api.md#summary)
  - The PSP provides information of every `capture` and `refund` to Vipps (not just `reserve`)
  - The Vipps branding must be according to the [Vipps design guidelines](https://github.com/vippsas/vipps-design-guidelines)
  - The PSP must provide specific error information to Vipps, such as:
    - `82 Refused by issuer`
    - `89 Not sufficient funds`
    - See [Error codes](https://github.com/vippsas/vipps-psp-api/blob/master/vipps-psp-api.md#error-codes)
    - See [Test data](https://github.com/vippsas/vipps-developers/tree/master/testdata)
  - See the Postman collection in [tools](tools/)
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
4. The PSP [contacts Vipps](https://github.com/vippsas/vipps-developers/blob/master/contact.md) to verify the integration in the production environment
5. Vipps completes one or more transactions in the production environment
6. The PSP goes live 🎉
