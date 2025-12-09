# API Reference

The QBank Connect API Reference was created from our **OpenAPI Specification (OAS 3.1)**; it outlines all of the required parameters, security headers, and schema models to call each endpoint in the API.

## 1. Reference Source

Our full machine readable OpenAPI Specification is located here: [openapi.yaml](https://www.google.com/search?q=openapi.yaml)

## 2. Endpoint Overview

Each endpoint is categorized by the type of financial resource accessed.

| Resource | Primary Endpoints | Latency | Use Cases |

| :--- | :--- | :--- | :--- |

| **Account** | `GET /accounts` | Batch-Latent (Daily) | Retrieve general account information. |

| **Transaction** | `GET /transactions` | Batch-Latent (Daily) | Financial reconciliations can be accomplished using the three date fields (transaction_date, value_date, posting_date). |

| **Balance** | `GET /balances/{id}` | Real-Time | Cash position at any point during the day and check available funds. [^10][^12] |

| **ACH** | `POST /payments/ach` | Asynchronous (15 minute batch) | High volume file transfers.[^28] **Requires Idempotency-Key**. |

| **PositivePay** | `POST /positivepay/issues`, `GET /positivepay/exceptions` | Asynchronous/Batch | Check fraud prevention management.[^15][^29] |

| **Wire/Transfer** | `POST /payments/wire`, `POST /transfers` | Real-Time | Fund transfer immediately. **Requires Idempotency-Key**. |

## 3. Standardized Error Response

For all non-conflict status codes (4xx and 5xx), any response that does not return a `409 Conflict` will return a standard JSON error response:

| Field | Type | Description |

| :--- | :--- | :--- |

| `message` | String | User friendly description of the problem encountered. |

| `error_code` | String | QBank specific code for automatic programmatic handling.[^30] |

| `request_id` | UUID | A unique identifier used by QBank Support (for internal logging).[^22] |


## References

[^10]: QBank Connect SLA, "API Latency and Data Freshness Profiles."
[^12]: Fiserv CoreAdvance API Integration Guide, "Direct Connect Latency."
[^15]: QBank Accounting Compliance Memo, "Posting Date Audit Requirement."
[^22]: SOX Audit Control Requirements, Section 404 (Conceptual Reference).
[^28]: ACH Operating Rules and Guidelines (Conceptual Reference).
[^29]: Positive Pay System Requirements, ABA Standards (Conceptual Reference).
[^30]: QBank Developer Documentation, "Error Code Directory."
