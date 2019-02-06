# Vipps PSP API Checklist

Document version 0.1.0

# Overall flow for PSP integrations

1. The PSP completes the direct integration with the [Vipps PSP API v2](https://github.com/vippsas/vipps-psp-api)
  - This includes _all_ the [API endpoints](https://vippsas.github.io/vipps-psp-api/)
  - The PSP provides information of every `capture` and `refund` to Vipps (not just `reserve`)
  - The Vipps branding must be according to the [Vipps design guidelines](https://github.com/vippsas/vipps-design-guidelines)
  - The PSP must provide specific error information to Vipps, such as `82 - Refused by issuer` and `89 Not sufficient funds` (see [Error codes](https://github.com/vippsas/vipps-psp-api#makepayment))
  - See the Postman collection in [tools](tools/)
2. The PSP completes testing in the [Vipps test environment](https://github.com/vippsas/vipps-developers#the-vipps-test-environment-mt)
3. The PSP completes one or more test transactions in the production environment
4. The PSP [contacts Vipps](https://github.com/vippsas/vipps-developers/blob/master/contact.md) to verify the integration in the production environment
5. Vipps completes one or more transactions in the production environment
6. The PSP goes live 🎉
