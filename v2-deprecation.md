# PSP V2 API Deprecation

In order to achieve compliance with PSD2’s SCA requirements Vipps will be processing network tokens instead of PANs going forward. Network Tokens is Vipps' strategy to achieve Delegated Authentication and will enable Vipps to leverage existing solutions to authenticate users - maintaining the current user experience with PIN or biometric identification to approve payments.

This will affect Vipps' PSP/PassThrough services and we are therefore required to launch an updated version of our PSP-API. The new version, that we will call V3, has the exact same workflow as our current V2 API, but the messages exchanged in the API calls contain different information. V3 API messages will contain a Network token, a cryptogram (aka. TAVV) and Vipps' Token Requestor ID (TRID).  V3 will require that you have the capabilities to process Network Tokens instead of raw PANs.This is described in technical detail in 

https://github.com/vippsas/vipps-psp-api/blob/master/vipps-psp-api.md#differences-between-v2-and-v3

 https://github.com/vippsas/vipps-psp-api/blob/master/vipps-psp-api.md#emvco-token-processing

and is live in our test environment.

As EBA and national authorities have mandated, SCA will come into effect January 1st 2021, and Vipps is working towards this deadline. We expect all raw PAN-processing to cause 3D Secure step ups beyond this date. Please contact us if you have any questions or want a meeting, and we will find a time to meet up.

## Scheme specific details

### Visa:

Visa tokens must be processed with the acquirer submitting the TAVV  cryptogram in field 126.8. The cryptogram received with the Network Token will contain the information for Delegated Authentication (DA) and the SCA factors used. Visa Token Service will during detokenization populate a flag for DA in field 34 to issuers and the Vipps TRID in field 123 Usage 2 Tag 03. In this way Issuers recognise Vipps originated transactions and not soft decline for 3DS step-up unless the issuing bank has opted out of the Visa D-SCA program.

### MasterCard:

Vipps is in dialogue with Mastercard regarding flagging delegated authentication. Although will be supporting Mastercard tokens. Mastercard’s delegated authentication programs are not directly tied to being a Token Requestor. We are currently in process with them to achieve Delegation of SCA. More info coming shortlySoft decline:

While Vipps is doing it’s utmost to achieve delegated SCA in all scenarios we are not in full control of all  the factors. Therefore it is critical that you support soft decline step ups for all payment flows. <this is described in our documentation here.https://github.com/alfmags/vipps-psp-api/blob/master/vipps-psp-api.md#psd2-compliance-and-secure-customer-authentication-sca

### Exemptions:

We encourage any exemptions to be applied when marking the transaction by the PSP/Acquirer that is applicable, for example low value exemptions.

