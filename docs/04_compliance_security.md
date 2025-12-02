# Compliance and Security Manual



Maintaining stringent security and audit controls is mandatory for FinTech integrations. QBank Connect adheres to PCI DSS and SOX standards.



## 1. Audit Logging Requirements (SOX/PCI)



A robust audit trail must be maintained for all financial activities. All API calls, system status changes, and mETL actions are logged in an immutable, structured format (JSON).[^3] [^21]



### 1.1. Mandatory Audit Log Fields



To ensure compliance, the logs capture:



| Field | Description | Compliance Context |
| :--- | :--- | :--- |
| `timestamp` | UTC time with millisecond precision.[^21] [^22] | SOX (Precise timing of events) |
| `actor` | The authenticated Client ID or user identity.[^22] | SOX (Who performed the action) |
| `entity_impacted` | The Account ID, Payment ID, or resource affected.[^22] | SOX (What was affected) |
| `request_id` | The unique request identifier for end-to-end tracing.[^23] [^22] | PCI DSS (Troubleshooting/Investigation) |
| `idempotency_key` | Mandatory for all transactional attempts (if applicable).[^17] | SOX (Proof of exactly-once execution) |



### 1.2. Non-Repudiation Header



For all transactional operations (`POST`, `PUT`, `PATCH`), the following header is highly recommended for audit compliance, especially if the API call is user-initiated within the ERP:



| Header Name | Value Format | Description |
| :--- | :--- | :--- |
| **`x-fapi-auth-date`** | HTTP-date (e.g., `Tue, 11 Sep 2012 19:43:31 GMT`) | Records the time the end-user (PSU) last authenticated with the client application.[^24] |



## 2. Transport and Application Security

The following mandatory controls govern how partners connect to QBank Connect, securing the data transmission pipeline and ensuring proper access.

### 2.1. TLS Enforcement



All traffic to QBank Connect must use **HTTPS (TLS 1.2 or higher)**. Connections using older, vulnerable TLS versions will be immediately rejected at the API Gateway level to mitigate man-in-the-middle attacks.[^25] X.509 certificates are utilized to validate identity and enforce encryption.[^25]



### 2.2. Role-Based Access Control (RBAC)



Access is strictly governed by the OAuth 2.0 scopes granted to your `client_id` upon registration (e.g., `transactions.read` vs. `payments.ach.write`). Requests attempting to access unauthorized resources will be met with a `403 Forbidden` response.[^6] [^7]



## 3. API Versioning Strategy



QBank Connect uses URL-based versioning (e.g., `/v1/`).



*   **Major Version Change (`v1` to `v2`):** Only occurs when introducing **breaking changes**—such as modifying existing payload structures, removing critical fields, or altering resource routing.[^26]

*   **Minor Updates:** All non-breaking additions (new endpoints, new optional fields) are rolled out within the existing version and guarantee backward compatibility.[^27] [^26]


## References

[^3]: QBank Internal Systems Manual, "Immutable Log Architecture."
[^6]: QBank API Security Guide, "OAuth 2.0 Scope and RBAC Policies."
[^7]: OAuth 2.0 Authorization Framework, RFC 6749 (Conceptual Reference).
[^17]: Idempotency Key Specification, RFC 8142 (Conceptual Reference).
[^21]: PCI DSS Requirement 10: Track and Monitor All Access to Network Resources and Cardholder Data.
[^22]: SOX Audit Control Requirements, Section 404 (Conceptual Reference).
[^23]: Distributed Tracing Standard, W3C Trace Context (Conceptual Reference).
[^24]: Open Banking (UK) Security Profile: FAPI Compliance and Authentication.
[^25]: NIST SP 800-52 Rev. 2: Guidelines for TLS Server Configuration.
[^26]: QBank API Design Standards, "Versioning Policy."
[^27]: Semantic Versioning 2.0.0 (Conceptual Reference).