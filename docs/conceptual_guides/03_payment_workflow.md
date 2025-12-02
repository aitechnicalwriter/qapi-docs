# Payment Workflows: Integrity and Webhooks



This guide covers the non-negotiable requirements for submitting financial transactions, ensuring **exactly-once execution** and tracking payment status via our finite state machine.[^16]



## 1. Transactional Idempotency Protocol



For all mutable operations (`POST`, `PUT`, `PATCH`), the **`Idempotency-Key`** header is mandatory. This protects your system from network failures, allowing safe request retries without creating duplicate payments.[^17][^18]



### 1.1. Header Requirement



| Header | Value Format | Mandatory | Purpose |
| :--- | :--- | :--- | :--- |
| `Idempotency-Key` | UUID v4 (36 characters) | YES | Ensures exactly-once execution, acting as a unique database constraint.[^17] |



### 1.2. Conflict Resolution (HTTP 409)



If you submit a payment request, but the network connection times out, your system may automatically retry. If the first request was received and is currently being processed by the mETL engine, the API will reject the retry attempt with a `409 Conflict`.



**Client Action:** When receiving a `409 Conflict`, **DO NOT** retry the submission immediately. Instead, use the related status endpoint (`GET /payments/{id}`) to check the status of the original request.[^16]



**Mandatory 409 Response Structure**



The response body adheres to the standardized `application/problem+json` format:



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



## 2. Payment Orchestration State Machine



All asynchronous payment requests (ACH, Positive Pay issues) are tracked by the Payment Orchestration Service (POS) through a defined finite-state machine. [^19] [^20]



| State | Transition Event | Actor/System | Latency |
| :--- | :--- | :--- | :--- |
| **`P_CREATED`** | `POST /payments/ach` is received. | ERP Platform / API | Instant |
| **`P_VALIDATED`** | Input validation (schema, funds check) succeeds. | QBank Connect Service | Sub-second |
| **`P_SUBMITTED_TO_CORE`** | **mETL Job** successfully places the batch file in the Fiserv SFTP location. | mETL Engine | 15 minutes (Max) |
| **`P_PROCESSING`** | Fiserv CoreAdvance acknowledges receipt and begins processing. | Fiserv CoreAdvance | Variable (EOD) |
| **`P_COMPLETED`** | Final settlement confirmation received. | mETL / POS | Next Business Day |
| **`P_FAILED`** | Core Rejection or validation failure. | CoreAdvance / POS | Immediate or Batch |



## 3. Payable Confirmation Webhooks



For time-sensitive payables, QBank Connect provides guaranteed delivery of status updates via webhooks. [^16]



### 3.1. Webhook Event



When a key state transition occurs (e.g., from `P_PROCESSING` to `P_COMPLETED`), the **Webhook Notification Service (WNS)** sends an event payload to your registered callback URL.



| Header | Purpose |
| :--- | :--- |
| `Webhook-Event-ID` | Unique identifier for this webhook delivery attempt. |
| `Webhook-Signature` | HMAC signature used to verify the payload's origin and integrity. |



### 3.2. Security Mandate: Signature Verification



To prevent spoofing and ensure non-repudiation, you **must** verify the `Webhook-Signature` header against the payload using your QBank-provided shared secret. **Events received without a valid signature must be discarded**.

## References

[^16]: QBank Connect API Reference, "Payment Status and Webhook Integration."
[^17]: Idempotency Key Specification, RFC 8142 (Conceptual Reference).
[^18]: Payments API Integration Guide, "Handling Duplicate Submissions."
[^19]: Payment Orchestration Service (POS) State Diagram, Internal Document.
[^20]: Fiserv CoreAdvance Batch Processing Guide, "Payment Lifecycle States."
