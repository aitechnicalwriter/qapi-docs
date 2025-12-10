> *NOTE: This document showcases John Stonecypher's skills in API Documentation and Docs-as-Code documentation methods. The products, business entities, and data used in this document are completely fictional. The products, and their associated code, were designed with assistance from Google Gemini.

# Welcome to QBank Connect API
This product provides third-party Treasury Management platforms with secure real-time access to QBank's suite of financial services.

> ### Developer QuickStart (<5 Minutes)
> Take our API for a test drive! 

> Use the [**Getting Started**](tutorials/01_getting_started.md) guide to onboard, authenticate, and make your first call, all in less than 5 minutes.

## 1. Functionality
When you integrate your TM Platform ("Platform") with QBank Connect, the Platform gains the following functionality:

* **Real-Time Cash Positioning:** Real-time access to cash balances.

* **Secure Payment Initiation:** Secure submission of ACH, Wire payments.

* **Fraud Prevention:** Two-way Positive Pay integration for managing check issues and exceptions.

* **Compliance:** Detailed logs of all activity and security headers for SOX and PCI DSS compliance.


## 2. Documentation Structure

This documentation set is organized around three distinct developer needs: Learning, Understanding, and Information.

### 2.1. Tutorials
Use tutorials to _learn_ new skills:

- [**Getting Started**](tutorials/01_getting_started.md): A quickstart tutorial for developers.

### 2.2. Explanations
Read explanations to _understand_ structures, connections, and processes:

- [**Architecture**](explanations/02_architecture.md): An explanation of the API's unique blend of real-time and batch processing.

- [**Payment Resilience**](explanations/03_payment_resilience.md): A discussion of the API's support for transactional idempotency.

### 2.3. References
Consult the reference materials for _information_ lookup when you need it:

- [**API Overview**](references/05_api_overview.md): Overview of the core components.

- [**API Console**](openapi-spec/index.html): Comprehensive, machine-readable OpenAPI Specification.

- [**Python SDK**](references/06_sdk_python.md): Python libraries to assist development.

- [**TypeScript SDK**](references/07_sdk_typescript.md): TypeScript libraries to assist development.

- [**Compliance and Security**](references/08_compliance_security.md): Regulatory and technical constraints.

- [**Glossary**](references/09_glossary.md): Definitions for terms and acronyms used in this document.
