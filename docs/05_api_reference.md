# API Reference



The QBank Connect API Reference is generated directly from our **OpenAPI Specification (OAS 3.1)**, which defines all mandatory parameters, security headers, and schema models.



## 1. Reference Source



The complete machine-readable specification is available here: [openapi.yaml](https://www.google.com/search?q=openapi.yaml)



## 2. Endpoint Overview



Endpoints are logically grouped by financial resource.



| Resource | Primary Endpoints | Latency | Key Use Case |
| :--- | :--- | :--- | :--- |
| **Account** | `GET /accounts` | Batch-Latent (Daily) | Retrieving general account metadata. |
| **Transaction** | `GET /transactions` | Batch-Latent (Daily) | Financial reconciliation using the three date fields (`transaction_date`, `value_date`, `posting_date`). |
| **Balance** | `GET /balances/{id}` | **Real-Time** | Intraday cash positioning and available funds check.[^10] [^12] |
| **ACH** | `POST /payments/ach` | Asynchronous (15 min batch) | Submitting high-volume file transfers.[28] **Requires `Idempotency-Key`**. |
| **PositivePay** | `POST /positivepay/issues`, `GET /positivepay/exceptions` | Asynchronous/Batch | Check fraud prevention management.[^15] [^29] |
| **Wire/Transfer** | `POST /payments/wire`, `POST /transfers` | **Real-Time** | Immediate fund movement. **Requires `Idempotency-Key`**. |



## 3. Standardized Error Response



All 4xx and 5xx responses that are not specifically a `409 Conflict` will return a standardized JSON error object:



| Field | Type | Description |
| :--- | :--- | :--- |
| `message` | String | User-friendly explanation. |
| `error_code` | String | QBank-specific code for programmatic handling.[^30] |
| `request_id` | UUID | Unique trace ID for QBank Support (Internal logging).[^22] |


## References

[^10]: QBank Connect SLA, "API Latency and Data Freshness Profiles."
[^12]: Fiserv CoreAdvance API Integration Guide, "Direct Connect Latency."
[^15]: QBank Accounting Compliance Memo, "Posting Date Audit Requirement."
[^22]: SOX Audit Control Requirements, Section 404 (Conceptual Reference).
[^28]: ACH Operating Rules and Guidelines (Conceptual Reference).
[^29]: Positive Pay System Requirements, ABA Standards (Conceptual Reference).
[^30]: QBank Developer Documentation, "Error Code Directory."
