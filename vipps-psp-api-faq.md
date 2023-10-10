<!-- START_METADATA
---
title: PSP API Frequently Asked Questions
sidebar_label: FAQ
sidebar_position: 40
description: Frequently asked questions for the PSP API.
pagination_next: null
pagination_prev: null
---
END_METADATA -->

# Frequently asked questions

Vipps Netthandel (eCommerce) via PSP offers functionality for payments on
websites and apps (P2M).

See the [PSP API guide](vipps-psp-api.md) for all the technical details.

For more common questions, see:

* [Common API General FAQ](https://developer.vippsmobilepay.com/docs/faqs)

API version: 3.0.0.

## How can I check if a merchant or sales unit is active?

You can try to initiate payment, specifying the `Merchant-Serial-Number` header:
[`POST:/v3/psppayments/init/`](https://developer.vippsmobilepay.com/api/psp#tag/Vipps-PSP-API/operation/initiatePaymentV3UsingPOST).

If the merchant or sales unit is not active, you will get an error:
"Merchant not available or active".

## Why do I get `errorCode 37 "Merchant not available or deactivated or blocked"`?

Or: `Merchant not available or active`.

Please check that the merchant's organization number is still active in
[Brønnøysundregistrene](https://www.brreg.no). We automatically deactivate
merchants (companies) when they are deleted from Brønnøysundregistrene.
This can also happen if a merchant changes organization type, for instance
from *ENK* to *AS*.

Merchants can log in on
[portal.vipps.no](https://portal.vipps.no)
and deactivate their sales units. This is sometimes done "by accident", without being
aware of the consequences. If a sales unit has been incorrectly deactivated,
the merchant can reactivate it again.

**Please note:** We require BankID for deactivation and reactivation,
and cannot help with this based on email requests.

Deactivation can also happen if the test merchant is not being used for a
*very long* time. Please
[contact customer service](https://vipps.no/kontakt-oss/),
and we will reactivate the merchant.

Partners that use
[partner keys](https://developer.vippsmobilepay.com/docs/partner/partner-keys)
can also get this error if the partner itself is deactivated, even though
the sales unit (that it is acting on behalf of) is active.

**Please note:** We no longer automatically deactivate test merchants.
Merchants can [create new test sales units](https://developer.vippsmobilepay.com/docs/developer-resources/portal#how-to-create-a-test-sales-unit).

## Why do I get an `Invalid MSN` error?

You may get an error like this:

```text
Invalid MSN: 123456. This MSN is not valid for the provided PSP id.
Check that you are using the correct credentials for the right environment.
```

This happens if the MSN was created with one PSP ID and you attempt to initiate a
payment with a different PSP ID. Although Vipps can help figure this out, it is
a time-consuming manual task, and we must ask the PSP to try to solve this
on its own.

## Why do I get `Invalid MSN: 654321. This MSN is not valid for the provided PSP id.`?

The full error message text is:

"Invalid MSN: 654321. This MSN is not valid for the provided PSP ID. Check that you are
using the correct credentials for the right environment."

In addition to what the error message says, this error can occur if a PSP attempts to initiate
payments for an MSN that was created by a different PSP.
PSP's can only initiate payments for MSNs that are connected to them.

The solution is to create a new MSN with the
[PSP Signup API](https://developer.vippsmobilepay.com/docs/APIs/psp-api/vipps-psp-signup-api/).

## How can I update the status of a payment?

When the PSP has new information about a payment, it is important to send the
new information to Vipps, so the user sees the correct status in Vipps.

The PSP must use the
[`POST:/v3/psppayments/updatestatus`](https://developer.vippsmobilepay.com/api/psp#tag/Vipps-PSP-API/operation/updatestatusUsingPOST)
endpoint to notify Vipps of changes to the payment,

**Please note:** Vipps processes the information in daily batches. See:
[Status updates](vipps-psp-api.md#status-updates).

## How can I get details for a payment?

With the PSP API, the PSP gets network tokens from Vipps
(see [What is a network token?](https://developer.vippsmobilepay.com/docs/APIs/psp-api/vipps-psp-api-faq/#what-is-a-network-token)).

The PSP uses the token to process the payment, and then sends Vipps information about
how it went with
[`POST:/psppayments/updatestatus`](https://developer.vippsmobilepay.com/api/psp/#tag/Vipps-PSP-API/operation/updatestatusUsingPOST).

Vipps does not have other information about the payment than what the PSP has sent.

The PSP can use
[`GET:/psppayments/{pspTransactionId}/details`](https://developer.vippsmobilepay.com/api/psp/#tag/Vipps-PSP-API/operation/getPSPPaymentDetailsUsingGET)
to get the details of a payment, but this will of course be the same
information that the PSP has sent earlier.

**Please note:** Vipps processes the information in daily batches. See:
[Status updates](vipps-psp-api.md#status-updates).

## What is a network token?

Network tokens are payment credentials that replace the card details for online
payments.

Every merchant gets a unique token for the card, so a network token can not be
shared between businesses. This also means that it's possible to invalidate
the token used by one merchant without invalidating the other tokens.

The EMVco token is not considered
[PCI DSS sensitive](https://www.pcisecuritystandards.org/document_library/?document=pci_dss).
See the
[EMVco documentation](https://www.emvco.com/emv-technologies/payment-tokenisation/)
for more.

## What is a PAN?

The PAN (Primary account numbers) is the 15- or 16-digit number found on all
credit or debit cards. Since PANs are accepted for online payments, anyone with
access to the PAN may be able to make payments without the physical card.

A PAN is unique per card, but may be shared between businesses.
This means that if a PAN is invalidated, it is invalidated for everyone.

## Why do I get `No network token available for this Agreement` or similar?

The most common reason for this problem is that the user has a card issuer that
does not support network tokens.

Banks can block tokens, for instance if the card is blocked, and are not always
great at reactivating them.

It could also be that the user has not added his/her new card in Vipps.

To find out more: Contact the card issuer.

## Is there a unique PSP ID for all merchants?

No, the PSP ID is unique for the PSP and used for all the PSP's merchants.

## Does Vipps have a test environment?

Yes, please see: [The Vipps test environment (MT)](https://developer.vippsmobilepay.com/docs/test-environment).

## What will be sent in the `makePayment()` request if the customer declines the payment, or it times out?

From the
[`makePayment`](https://developer.vippsmobilepay.com/api/psp#tag/Endpoints-required-by-Vipps-from-the-PSP/operation/makePaymentV3UsingPOST)
specification:

| Name      | Type   | Size | Optional | Values             |
| --------- | ------ | ---- | -------- | ------------------ |
| confirmed | String |  7   | No       | YES/TIMEOUT/CANCEL |

## What is the correct response to the `MakePayment` request?

Continued: "What should be the response when `Confirmed` contains `TimeOut` or `Cancel` -
should `paymentInfo.status` be `OK` or `FAIL`? If it is `FAIL", what error code should be returned?`"

It is suggested that you send `FAIL` in case of failure either at your end or -
as in this case - at our end.

If `Confirmed` contains `TimeOut` or `Cancel`: Set `paymentInfo.status` to `FAIL`?

## Can you help merchants that have problems with the PSP's use of the PSP API?

Merchants that use Vipps through a PSP must contact the PSP if there are
problems. This is because we only provides the payment card token to the PSP,
and the PSP is responsible for performing the transaction and then to notify
Vipps about how it went.

Simplified flow when using Vipps through a PSP:

1. The user selects to pay with Vipps at the merchant.
2. The PSP uses the PSP API to initiate a payment.
3. The user confirms the payment in Vipps.
4. We send the user's payment card token to the PSP.
5. The PSP uses the payment card token to perform the payment.
6. The PSP uses the PSP API to inform us about how the payment went.
7. The user gets a confirmation in their Vipps app.

It is important to understand that we only provide the payment card token
to the PSP. We are not involved in the payment process. If there are any
problems with the payment, it is the PSP that has all the information about that.

## Is it possible to skip the landing page?

Skipping the landing page is reserved for special cases, where displaying it is not possible.

For more details, see
[Knowledge base: Skip the landing page](https://developer.vippsmobilepay.com/docs/common-topics/landing-page/#skip-landing-page)

## What functionality is included in the eCom API, but not the PSP API?

See [Knowledge base: Benefits of direct integration](https://developer.vippsmobilepay.com/docs/common-topics/direct-vs-psp#benefits-of-direct-integration).
