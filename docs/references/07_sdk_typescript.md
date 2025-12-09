# TypeScript SDK Integration Guide
QBank's TypeScript SDK is used to integrate QBank with a modern Node.js application or with a web browser in a **type-safe** manner that abstracts away the underlying complexity of the API and creates an environment where **errors can be identified during development instead of runtime**. [^1], [^2]

## 1. Installation and Configuration

You may use npm or yarn to install the SDK:



```bash
npm install @qbank/connect

# or

yarn add @qbank/connect
```



You will need to initialize the client using your `clientId` and `clientSecret`. The SDK handles the complete **OAuth 2.0 Client Credential token lifecycle**, including caching and renewing the Bearer token before it has expired. [^3]



```typescript
import { QBankConnectClient, IdempotencyKeyConflictError, PermissionDeniedError } from '@qbank/connect';
import { AchPaymentRequest } from './models'; // Auto-generated model interfaces

// Client initializes and manages the token automatically
const client = new QBankConnectClient({
  clientId: 'YOUR_CLIENT_ID',
  clientSecret: 'YOUR_CLIENT_SECRET',
  scopes: ['payments.ach.write', 'balances.read'],
});
```



## 2. Idiomatic Idempotence Management

For all transactional endpoints (`POST`, `PUT`) the SDK provides **transactional integrity** management of the `Idempotency-Key` header.

* **First Try**: Generates a new UUID as the value of the `Idempotency-Key` header when the key is not supplied manually.

* **Retrying**: If a connection issue occurs, you must retry the payment with **the same** `Idempotency-Key` so that the payment is processed **only once**. [^4], [^5]

### 2.1. Securely Submitting Payments (Manual Retry Example)



```typescript
const requestKey = crypto.randomUUID();
const achRequest: AchPaymentRequest = {
  sourceAccountId: '...',
  recipientDetails: { /*... */ },
  amount: 500.25,
  effectiveDate: '2025-11-05',
};

async function createPaymentWithRetry(key: string, request: AchPaymentRequest) {
  try {
    const response = await client.payments.createAch(request, {
      idempotencyKey: key, // Provide key for idempotency guarantee
    });
    console.log('Payment accepted. Status:', response.status);
    return response;
  } catch (error) {
    if (error instanceof IdempotencyKeyConflictError) {
      // The API returned a 409 Conflict. **DO NOT** retry the submission.
      console.warn(`Idempotency conflict detected. Reason: ${error.detail}`);
      // Recommended: Check status using GET /payments/{id}
    } else if (error instanceof PermissionDeniedError) {
      // The API returned a 403 Forbidden.
      throw new Error('Unauthorized API call. Check scopes.');
    } else {
      // Catch other errors (e.g., QBankServiceUnavailableError for 5xx)
      throw error;
    }
  }
}
createPaymentWithRetry(requestKey, achRequest);
```



## 3. Type Safety and Structured Errors (Developer Experience)

Using strong type checking (e.g. Zod Schemas), the SDK makes the inputs and outputs of APIs explicit, thus **enabling errors to be caught while developing** (in the IDE) instead of at runtime. [^1]

In addition to translating all status code based API errors (e.g. 400, 403, 409) to **Named Error Classes** (`IdempotencyKeyConflictError`, `PermissionDeniedError`), this abstraction enables developers to **programmatically handle failures** using `try/catch` blocks without having to parse raw HTTP status codes or error messages. [^1]


## Documents Cited

[^1]: QBank TypeScript SDK Readme, "Core Features and Design Principles (Developer Experience)."
[^2]: TypeScript SDK Source Code, `src/client.ts` (Client Initialization).
[^3]: OAuth 2.0 Client Credentials Flow, RFC 6749 (Token Lifecycle).
[^4]: API Design Guidelines, "Idempotency and Network Failure Handling."
[^5]: QBank Connect API Reference, "Idempotency-Key Requirements."