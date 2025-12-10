# API Introduction

In this section we describe the **key elements** of the QBank Connect API.

> For the full machine-readable OpenAPI Specification, visit the [**API Console**](../openapi-spec/index.html)

## 1. Service Information & Login

### 1.1. The Base URL

You must use the versioned base URL as your prefix for all API calls:

```bash

https://api.qbankconnect.com/v1

```

### 1.2. Security

QBank Connect uses the **OAuth 2.0 Client Credentials Grant** mechanism for B2B server-to-server login.

The `access_token` represents the authorization you have to perform operations defined in the scopes listed below. If the `access_token` does not contain the required scope for an operation you will receive a **403 Forbidden** response.

| Scope | Description | Resource Access |
| :--- | :--- | :--- |
| `accounts.read` | Get general information about an account | Read-Only |
| `balances.read` | Get the current cash balance on an account | Read-Only |
| `transactions.read` | Get historical transaction data for an account | Read-Only |
| `payments.read` | Get the current status and history of payments | Read-Only |
| `payments.ach.write` | Initiate high volume ACH payments | Write |
| `payments.wire.write` | Initiate wire payments immediately | Write |
| `transfers.write` | Transfer funds between accounts | Write |
| `positivepay.issues.write` | Submit check issues (Positive Pay) | Write |
| `positivepay.exceptions.read` | Get outstanding check exceptions (Positive Pay) | Read-Only |

## 2. Endpoints

The QBank architecture uses both real-time and batch architectures. All developers must be aware of the delay associated with each resource.

| Resource | Primary Endpoints | Latency | Description |
| :--- | :--- | :--- | :--- |
| **Authentication** | `POST /token` | Real-Time | OAuth 2.0 Client Credentials flow. |
| **Account** | `GET /accounts` | **Batch-Latent (Daily)** | General account details. Updated nightly via mETL. |
| **Balance** | `GET /balances/{accountId}` | **Real-Time** | Calls the core system synchronously for the current cash position on an account. |
| **Transactions** | `GET /transactions` | **Batch-Latent (Daily)** | Historical transactions for an account with three reconciliation date fields. |
| **ACH Payments** | `POST /payments/ach` | **Asynchronous (15 min batch)** | Bulk file uploads for ACH payments. **Must include `Idempotency-Key`.** |
| **Wire Payments** | `POST /payments/wire` | **Real-Time** | Immediately sends funds to another bank account. **Must include `Idempotency-Key`.** |
| **Transfers** | `POST /transfers` | **Real-Time** | Moves funds internally between accounts. **Must include `Idempotency-Key`.** |
| **Payment Status** | `GET /payments/{paymentId}` | Real-Time | Enables the client to poll for the final status of a payment. |
| **Positive Pay** | `POST /positivepay/issues` | Asynchronous/Batch | Creates a new check issue record. **Must include `Idempotency-Key`.** |
| | `GET /positivepay/exceptions` | **Batch-Latent (Daily)** | Retrieves non-matching checks pending decisioning. |

## 3. Request and Response Headers

### 3.1. Authorization Header

A valid **`Authorization`** header containing the `Bearer` access token obtained from the `/token` endpoint is required for every call.

| Header | Value Format |
| :--- | :--- |
| `Authorization` | `Bearer ` |

### 3.2. Idempotency Header (Required for Transaction Endpoints)

Every `POST` request for transactional endpoints (e.g., `/payments/ach`, `/transfers`) **requires** an `Idempotency-Key` header so the system can enforce the atomicity of transactions and eliminate duplicate submissions.

| Header | Value Format | Description |
| :--- | :--- | :--- |
| **`Idempotency-Key`** | **UUID V4 (36-character)** | A client-provided UUID that uniquely identifies a transaction to avoid duplication. |

## 4. WebHooks

QBank Connect notifies your platform of the final status of asynchronous payments using WebHooks.

### 4.1. Event: `paymentStatusUpdate`

The event is triggered whenever there is a final transition in the payment status (i.e., `P_COMPLETED` or `P_FAILED`).

| **Component** | **Detail** |
| :--- | :--- |
| **Incoming Method** | `POST` |
| **Payload Schema** | `WebhookEvent` (including the updated `Payment` object) |
| **Required Client Response** | `HTTP 200 OK` (acknowledge receipt) |

### 4.2. WebHook Security Verification

For non-repudiation, all webhook deliveries contain the **`Webhook-Signature`** header. Your platform **must** validate this HMAC signature against the payload using the shared secret provided by QBank Connect. **You should discard events that do not have a valid signature**.

| **Header Name** | **Purpose** |
| :--- | :--- |
| `Webhook-Event-ID` | Unique identifier for this delivery attempt. |
| **`Webhook-Signature`** | HMAC signature for validating the payload's origin and integrity. |

## 5. Error Handling

All error responses are categorized into one of two types to allow client SDKs to catch errors (e.g., in Python/TypeScript `try/catch` blocks).

### 5.1. Standard Error Response (4xx, 5xx)

Use this error response for all general errors (Authentication, Authorization, Validation, etc.) except for idempotence conflicts.

**Content-Type:** `application/json`

| **Status Code** | **Example Use Case** |
| :--- | :--- |
| **`401 Unauthorized`** | Access token is invalid/has expired. |
| **`403 Forbidden`** | Token is valid, however missing required scopes (e.g., `payments.write`). |
| **`404 Not Found`** | Resource ID in the path is not present. |
| **`503 Service Unavailable`** | System is temporarily under maintenance. |

#### Standard Error Response Body

| **Field** | **Type** | **Description** |
| :--- | :--- | :--- |
| `message` | `string` | A user-friendly explanation of what occurred. |
| **`error_code`** | `string` | QBank-specific code (e.g., `ERR_ACC_001`) so SDK developers can programmatically translate the error. |
| **`request_id`** | `UUID` | Unique identifier for the entire lifecycle traceability and QBank Support lookup. |

### 5.2. Idempotency Conflict Response (HTTP 409)

This specialized response is sent when a duplicate transaction request is received while the original request is still being processed.

**Content-Type:** `application/problem+json` (IETF RFC 7807)
| **Status Code** | **Error Type** | **Recovery Instruction** |
| :--- | :--- | :--- |
| **`409 Conflict`** | **Idempotency Violation** | **DO NOT RETRY**. Use the `transaction_id` to poll the `GET /payments/{id}` endpoint. |

#### Conflict Response Body (`application/problem+json`)

| **Field** | **Type** | **Description** |
| :--- | :--- | :--- |
| `type` | `URI` | Reference URI for the problem type (e.g., `.../problems/idempotency-conflict`). |
| `title` | `string` | Brief human-readable description of the problem. |
| `detail` | `string` | Detailed context describing why the problem occurred (e.g., "There is already a request with this key being processed"). |
| **`transaction_id`** | `UUID` | **The ID of the transaction that is currently processing**, allowing safe client recovery. |

## 6. Data Models

All resource models conform to an inheritance hierarchy to ensure regulatory compliance over all financial objects.

### 6.1. Audit Fields (Compliance Inheritance)

All auditable resources (`Account`, `Transaction`, `Payment`, etc.) will automatically inherit the following required fields for **SOX** and **PCI DSS** compliance.

| **Field** | **Type** | **Description** | **Compliance Context** |
| :--- | :--- | :--- | :--- |
| **`timestamp`** | `string` (`date-time`) | The time recorded in UTC with millisecond precision. | **SOX** (Audit Trail) |
| **`actor`** | `string` | The authenticated Client ID who initiated the request. | **SOX** (User Accountability) |
| **`request_id`** | `string` (`uuid`) | Unique ID of the request for end-to-end traceability. | **PCI DSS** (Troubleshooting) |

### 6.2. Transaction Resource Model

The `Transaction` model contains three distinct date fields important for financial reconciliation and regulatory obligations.

| **Field** | **Type** | **Use** |
| :--- | :--- | :--- |
| `transaction_date` | `string` (`date-time`) | **Initiated:** Date/time the customer/ERP initiated the transaction. |
| `value_date` | `string` (`date`) | **Funds/Interest:** Date funds are actually credited/debited to calculate interest. |
| `posting_date` | `string` (`date-time`) | **Available/Posted:** Date/time record was added to the QBank Connect database and made available through the API (EOD sync). |