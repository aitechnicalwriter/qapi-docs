# API Overview

This overview describes the **core components** of QBank Connect API.

> For the comprehensive machine-readable OpenAPI Specification, see the [**API Console**](../openapi-spec/index.html)

## 1. Service Details & Authentication

### 1.1. Base URL
All API requests must be prefixed with the versioned base URL:
`https://api.qbankconnect.com/v1`

### 1.2. Security
QBank Connect uses the **OAuth 2.0 Client Credentials Grant** for B2B server-to-server authentication.

The `access_token` grants permissions defined by the scopes below. You will receive a **`403 Forbidden`** error if the token lacks the necessary scope for an endpoint.

| Scope | Description | Resource Access |
| :--- | :--- | :--- |
| `accounts.read` | Retrieve general account information | Read-Only |
| `balances.read` | Retrieve real-time cash balances | Read-Only |
| `transactions.read` | Retrieve historical transaction data | Read-Only |
| `payments.read` | Retrieve payment status and history | Read-Only |
| `payments.ach.write` | Initiate high-volume ACH payments | Write |
| `payments.wire.write` | Initiate real-time Wire payments | Write |
| `transfers.write` | Execute internal fund transfers | Write |
| `positivepay.issues.write` | Submit check issue records (Positive Pay) | Write |
| `positivepay.exceptions.read` | Retrieve outstanding check exceptions | Read-Only |


## 2. Endpoints

The QBank architecture is a hybrid of real-time and batch systems. Developers must be aware of the latency profile for each resource.

| Resource | Primary Endpoints | Latency | Description |
| :--- | :--- | :--- | :--- |
| **Authentication** | `POST /token` | Real-Time | OAuth 2.0 Client Credentials flow. |
| **Account** | `GET /accounts` | **Batch-Latent (Daily)** | General account info. Refreshed nightly via mETL. |
| **Balance** | `GET /balances/{accountId}` | **Real-Time** | Synchronous call to core system for immediate cash position. |
| **Transaction** | `GET /transactions` | **Batch-Latent (Daily)** | Historical records with three reconciliation date fields. |
| **ACH Payment** | `POST /payments/ach` | **Asynchronous (15 min batch)** | High-volume file transfers. **Requires `Idempotency-Key`**. |
| **Wire Payment** | `POST /payments/wire` | **Real-Time** | Immediate fund transfer. **Requires `Idempotency-Key`**. |
| **Transfer** | `POST /transfers` | **Real-Time** | Internal account fund movement. **Requires `Idempotency-Key`**. |
| **Payment Status** | `GET /payments/{paymentId}` | Real-Time | Allows polling for the final outcome of any payment. |
| **Positive Pay** | `POST /positivepay/issues` | Asynchronous/Batch | Submit a new check issue record. **Requires `Idempotency-Key`**. |
| | `GET /positivepay/exceptions` | **Batch-Latent (Daily)** | Retrieve outstanding non-matching checks for decisioning. |

## 3. Request and Response Headers

### 3.1. Authorization Header
All calls require the **`Authorization`** header, containing the `Bearer` token received from the `/token` endpoint.

| Header | Value Format |
| :--- | :--- |
| `Authorization` | `Bearer <access_token>` |

### 3.2. Idempotency Header (Mandatory for Transactions)
All transactional `POST` requests (e.g., `/payments/ach`, `/transfers`) **must** include the `Idempotency-Key` header to ensure transactional integrity and prevent duplicate submissions.

| Header | Value Format | Description |
| :--- | :--- | :--- |
| **`Idempotency-Key`** | **UUID V4 (36-character)** | A unique, client-generated key that prevents duplicate processing. |

## 4. Webhooks

QBank Connect uses webhooks to notify your platform of significant state changes in asynchronous payments.

### 4.1. Event: `paymentStatusUpdate`
Sent when a payment transitions to a final state (e.g., `P_COMPLETED` or `P_FAILED`).

| Component | Detail |
| :--- | :--- |
| **Inbound Method** | `POST` |
| **Payload Schema** | `WebhookEvent` (containing the updated `Payment` object) |
| **Required Client Response** | `HTTP 200 OK` (to acknowledge receipt) |

### 4.2. Webhook Security Verification
For non-repudiation, all webhook deliveries include the **`Webhook-Signature`** header. Your application **must** verify this HMAC signature against the payload using the shared secret provided by QBank Connect. **Events without a valid signature must be discarded**.

| Header Name | Purpose |
| :--- | :--- |
| `Webhook-Event-ID` | Unique identifier for this delivery attempt. |
| **`Webhook-Signature`** | HMAC signature for verifying the payload's origin and integrity. |


## 5. Error Handling

All error responses fall into one of two categories, ensuring client SDKs can programmatically handle failures (e.g., in Python/TypeScript `try/catch` blocks).

### 5.1. Standard Error Response (4xx, 5xx)

Used for all general failures (Authentication, Authorization, Validation, etc.) except for idempotency conflicts.

**Content-Type:** `application/json`

| Status Code | Example Use Case |
| :--- | :--- |
| **`401 Unauthorized`** | Invalid/expired access token. |
| **`403 Forbidden`** | Valid token, but missing required scope (e.g., `payments.write`). |
| **`404 Not Found`** | Resource ID in the path does not exist. |
| **`503 Service Unavailable`** | Temporary system maintenance. |

#### Standard Error Response Body

| Field | Type | Description |
| :--- | :--- | :--- |
| `message` | `string` | User-friendly explanation. |
| **`error_code`** | `string` | QBank-specific code (e.g., `ERR_ACC_001`) for SDK programmatic translation. |
| **`request_id`** | `UUID` | Unique identifier for end-to-end tracing and QBank Support lookup. |

### 5.2. Idempotency Conflict Response (HTTP 409)

This specialized response is used when a transaction request is duplicated while the original request is still processing.

**Content-Type:** `application/problem+json` (IETF RFC 7807)

| Status Code | Error Type | Recovery Instruction |
| :--- | :--- | :--- |
| **`409 Conflict`** | Idempotency Violation | **DO NOT RETRY**. Use the `transaction_id` to poll the `GET /payments/{id}` endpoint. |

#### Conflict Response Body (`application/problem+json`)

| Field | Type | Description |
| :--- | :--- | :--- |
| `type` | `URI` | Reference URI for the problem type (e.g., `.../problems/idempotency-conflict`). |
| `title` | `string` | Human-readable error summary. |
| `detail` | `string` | Specific context (e.g., "A request with this key is already processing"). |
| **`transaction_id`** | `UUID` | **The ID of the transaction that is currently processing**, enabling safe client recovery. |

***

## 6. Data Models

All resource schemas adhere to an inheritance pattern to ensure regulatory compliance across all financial objects.

### 6.1. AuditFields (Compliance Inheritance)
All auditable resources (`Account`, `Transaction`, `Payment`, etc.) automatically inherit the following mandatory fields for **SOX** and **PCI DSS** compliance.

| Field | Type | Description | Compliance Context |
| :--- | :--- | :--- | :--- |
| **`timestamp`** | `string` (`date-time`) | Time recorded in UTC with millisecond accuracy. | SOX (Audit Trail) |
| **`actor`** | `string` | The authenticated Client ID who initiated the call. | SOX (User Accountability) |
| **`request_id`** | `string` (`uuid`) | Unique request ID for end-to-end tracing purposes. | PCI DSS (Troubleshooting) |

### 6.2. Transaction Resource Schema
The `Transaction` schema includes three distinct date fields crucial for financial reconciliation and regulatory requirements.

| Field | Type | Purpose |
| :--- | :--- | :--- |
| `transaction_date` | `string` (`date-time`) | **Initiated:** When the customer/ERP initiated the transaction. |
| `value_date` | `string` (`date`) | **Interest/Funds:** The date funds are actually credited/debited for interest accrual. |
| `posting_date` | `string` (`date-time`) | **Available/Posted:** The date the record was inserted into the QBank Connect DB and made available via API (EOD sync). |

