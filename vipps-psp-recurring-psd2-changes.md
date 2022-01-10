# Changes to PSP Recurring to support PSD2 requirements

### Preamble

PSD2 requires a `traceId` to be stored by the merchant and send on all subsequent transactions when processing subscription payments.

A `traceId` is the schemes reference value of the original CIT transaction authenticated with SCA, also know as Network Transaction Reference.

Vipps PSP Recurring allows the user to freely change which card they have assosiated to a subscription agreement with a merchant. This flexibility is positive for merchants with regards to anti-churn but has complexities with PSD2. From an issuers perspective, a subscription is between a specific card and merchant. This means Vipps PSP Recuring must have functionality to get a new `traceId` when a customer chooses to change card assosiated with a subscription.

### Changes to MIT Processing

Based on these requirements the following changes are proposed:

The `makePayment` response when performing the CIT to instantiate a subscription agreement shall include a new property, `traceId`. This is the transaction reference from the scheme (Visa/Mastercard). Vipps will store this value for future payments.

When a MIT payment is requested via `/v3/psppayments/payments` Vipps will return the current `traceId` value which should be sent with the payment to the processing acquirer. This value will assert with the Issuer that the subscription has a valid SCA according to PSD2.


### Handling user initiated card change

When a user changes the card assosiated with a PSP Recurring agreement in the Vipps app Vipps will perform a `makePayment` callback to the PSP. This will use the `pspTransactionId` of the original CIT transaction that initiated the agreement. The callback with use a new `paymentState` value `VERIFY`. Included in the callback will be a network token generated for use on a 0-auth transaction. 
The PSP shall process this as a 0-auth CIT transaction with their acquirer. 

The PSP should send a response to this callback inline with all other `makePayment` callback processing.

If successful the PSP shall respond with the `traceId` from this CIT transaction which Vipps will store for future MIT payments.


If a `SOFT-DECLINE` is received from the issuer then the PSP should respond with `SOFT-DECLINE` state. A normal soft decline flow will be initiated, the follow-up callback will then use state `VERIFY_COMPLETED`. This is to allow differentiation between 3DS completion and user drop-off where they then start a new `VERIFY` session.