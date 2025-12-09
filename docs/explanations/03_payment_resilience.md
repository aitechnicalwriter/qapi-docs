# Payment Resilience

If a connection error causes a payment request to fail, your Platform can safely repeat the request. QBank Connect API supports **idempotency** to prevent the risk of duplicate payments.[^16]

## 1. Transactional Idempotence Protocol

All transaction requests require the **`Idempotency-Key`** header containing a 36-character key value (Value Type: UUID V4). In the event that the request needs to be retried, this key provides a database constraint that prevents duplicate records.[^17][^18] 

**Sample Header:**
```
    -H "Idempotency-Key: f47ac10b-58cc-4372-a567-0e02b2c3d479" \
```

### 1.2. Conflict Resolution (HTTP 409)

When a request is received successfully, the confirmation response can still fail. Your Platform **mistakes the non-response as a sign of request failure**, so it tries the request again. When QBank Connect finds that the first request is still processing, the second request is rejected with a **409 Conflict** error.

The response body conforms to the standard **`application/problem+json`** structure:

```json
HTTP/1.1 409 Conflict
Link: https://docs.qbankconnect.com/03_payment_workflow.md#transactional-idempotency-protocol; rel="describedby"
Content-Type: application/problem+json

{
"type": "https://api.qbankconnect.com/errors/idempotency_conflict",
"title": "A request is outstanding for this Idempotency-Key",
"status": 409,
"detail": "The payment submission with this key is already in the P_VALIDATED or P_SUBMITTED_TO_CORE state.",
"instance": "a2b3c4d5-6e7f-8g9h-0i1j-2k3l4m5n6o7p"
}
```

## 2. Payment Orchestration Finite State Machine

All payment requests are tracked by the **Payment Orchestration Service (POS)** using a finite state machine. [^19] [^20]

| State | Transition Event | Actor/System | Latency |
| --- | --- | --- | --- |
| **`P_CREATED`** | A `POST /payments/ach` request is submitted. | ERP Platform / API | Instant |
| **`P_VALIDATED`** | Validations succeed (schema, funds). | QBank Connect Service | Sub-Second |
| **`P_SUBMITTED_TO_CORE`** | The mETL Engine job successfully places the batch file into the Fiserv SFTP Location. | mETL Engine | 15 Minutes (Max) |
| **`P_PROCESSING`** | The Fiserv CoreAdvance confirms receipt and begins processing the transaction. | Fiserv CoreAdvance | EOD (Variable) |
| **`P_COMPLETED`** | Final Settlement Confirmation Received. | mETL / POS | Next Business Day |
| **`P_FAILED`** | Either rejected by Core Advance, or validation failed. | Core Advance / POS | Immediate or Batch |

## 3. Payable Status Notifications

For payable transactions, QBank Connect uses webhooks to send status updates to your Platform. [^16]

### 3.1. Webhook Events

When a significant state transition occurs (i.e. from `P_PROCESSING` to `P_COMPLETED`) the **Webhook Notification Service (WNS)** sends an event payload to your registered callback URL. The headers include:

- **Webhook-Event-ID:**  An identifier for this webhook delivery attempt.

- **Webhook-Signature:** The HMAC signature that verifies that a). The payload originated from QBank Connect, b). The payload contents have not been tampered with.

### 3.2. Signature Verification

In order to prevent impersonation and ensure non-repudiation of events received, your Platform must verify the `Webhook-Signature` header against the payload using the shared secret provided by QBank Connect. **Any events received without a valid signature should be discarded**.

## 4. Documents Cited

[^16]: QBank Connect API Reference, "Payment Status and Webhook Integration."
[^17]: Idempotency Key Specification, RFC 8142.
[^18]: Payments API Integration Guide, "Handling Duplicate Submissions."
[^19]: Payment Orchestration Service (POS) State Diagram, Internal Document.
[^20]: Fiserv CoreAdvance Batch Processing Guide, "Payment Lifecycle States."
