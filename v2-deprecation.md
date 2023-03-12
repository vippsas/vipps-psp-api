<!-- START_METADATA
---
title: PSP API v2 deprecation
sidebar_label: PSP API v2 deprecation
sidebar_position: 100
hide_table_of_contents: true
pagination_next: null
pagination_prev: null
---
END_METADATA -->

# PSP API v2 Deprecation

In order to achieve compliance with PSD2’s SCA requirements, we will be processing network tokens instead of PANs going forward. The use of network tokens is Vipps' strategy to achieve Delegated Authentication. This will enable Vipps to leverage existing solutions to authenticate users; thus, maintaining the current user experience with PIN or biometric identification in payment approvals.

This will affect Vipps' PSP/PassThrough services and we are therefore required to launch an updated version of our PSP API. The new version, that we will call v3, has the exact same workflow as our current v2 API, but the messages exchanged in the API calls contain different information. 

The v3 API messages will contain a network token, a cryptogram (aka. TAVV) and Vipps' Token Requestor ID (TRID). The PSP API v3 will require that you have the capabilities to process Network Tokens instead of raw PANs. This is described in technical detail below and is live in our test environment:

* [Differences between v2 and v3](vipps-psp-api.md#differences-from-psp-api-v2-to-v3)
* [EMVco token processing](vipps-psp-api.md#emvco-token-processing)

As EBA and national authorities have mandated, SCA will come into effect January 1st 2021, and Vipps is working towards this deadline. We expect all raw PAN-processing to cause 3D Secure step-ups from this date. Please
[contact us](https://vippsas.github.io/vipps-developer-docs/docs/vipps-developers/contact)
if you have any questions or want a meeting, and we will find a time to meet up.

## Scheme specific details

### Visa

Visa tokens must be processed with the acquirer submitting the TAVV cryptogram in field 126.8. The cryptogram received with the network token will contain the information for Delegated Authentication (DA) and the SCA factors used. The Visa Token Service will during detokenization populate a flag for DA in field 34 to issuers and the Vipps TRID in field 123 Usage 2 Tag 03. In this way, issuers recognize Vipps-originated transactions and not soft decline for 3DS step-up unless the issuing bank has opted out of the Visa D-SCA program.

### MasterCard

Vipps is in dialogue with Mastercard regarding flagging delegated authentication. Although we will be supporting MasterCard tokens, MasterCard’s delegated authentication programs is not directly tied to being a Token Requestor. We are currently in process with MasterCard to achieve Delegation of SCA. More info coming shortly.

## Soft decline

While Vipps is doing it’s utmost to achieve delegated SCA in all scenarios we are not in full control of all  the factors. Therefore it is critical that you support soft decline step-ups for all payment flows.
This is described in [PSD2 Compliance and Secure Customer Authentication (SCA)](vipps-psp-api.md#psd2-compliance-and-secure-customer-authentication-sca).

### Exemptions

We encourage any exemptions to be applied when marking the transaction by the PSP/Acquirer that is applicable, for example low value exemptions.
