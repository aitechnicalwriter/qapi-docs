# Compliance & Security Manual

Maintaining the highest level of security & auditing for FinTech integrations is required. QBank Connect is compliant with both SOX & PCI DSS standards.

## 1. Audit Logging Requirements (SOX / PCI)

A comprehensive audit log must exist for each financial activity. Each API call, system status change & mETL action is captured in an immutable, structured format (JSON).[^3] [^21]

## 1.1. Mandatory Audit Log Fields

To provide auditable evidence, the logs contain:

| Field | Description | Compliance Context |
| :--- | :--- | :--- |
| `timestamp` | Time recorded in UTC with millisecond accuracy.[^21] [^22]. | SOX (Audit Trail) |
| `actor` | The authenticated Client ID or User Identity.[^22]. | SOX (User Accountability) |
| `entity_impacted` | The impacted Account ID, Payment ID, etc.[^22]. | SOX (Data Accountability) |
| `request_id` | Unique request ID for end-to-end tracing purposes.[^23] [^22]. | PCI DSS (Troubleshooting / Investigation) |
| `idempotency_key` | Required for all transactional attempts (if applicable).[^17]. | SOX (Audit Trail) |

### 1.2. Non-Repudiation Header

For all transactional requests (POST, PUT, PATCH), a highly recommended audit compliance header should be used for each request that initiates from a user interface within the ERP.

| Header Name | Value Format | Description |
| :--- | :--- | :--- |
| **`x-fapi-auth-date`** | HTTP-date (e.g., Tue, 11 Sep 2012 19:43:31 GMT) | The date/time the end-user (PSU) last authenticated with the client application.[^24]. |



## 2. Secure Transport and Applications

The following mandatory security policies apply to how you connect to QBank Connect to secure the data transport pipeline, and provide appropriate access to your environment.

### 2.1. TLS Enforcement

All connections to QBank Connect MUST use HTTPS (TLS 1.2 or higher) to provide secure communication over the web. If a connection is made using a less secure version of TLS, it will be denied immediately at the API Gateway layer to prevent man-in-the-middle attacks.[^25]. X.509 certificates are used to verify the identity of the client, and provide encryption.[^25]

## 2.2. Role-Based Access Control (RBAC)

Access to the services provided by QBank Connect are based on the OAuth 2.0 scopes that have been assigned to your client_id upon registration (i.e. transactions.read vs payments.ach.write). If you attempt to make a request against a resource for which you do not have permission, you will receive a 403 Forbidden response.[^6] [^7]

## 3. API Versioning Strategy

QBank Connect employs a URL-based versioning strategy (i.e. /v1/).

**Major Version Changes (v1 to v2):** These occur when breaking changes are introduced into the API (e.g. changing the structure of the payload, removing important fields, changing the way resources are routed through the API)[^26].

**Minor Versions:** Any changes that are non-breaking (adding new endpoints, adding optional fields, etc.) will be added to the current version of the API and will remain backwards compatible with previous versions.[^27] [^26]


## 4. Documents Cited

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