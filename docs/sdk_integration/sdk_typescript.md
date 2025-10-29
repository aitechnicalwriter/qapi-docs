# TypeScript SDK Integration Guide



The QBank TypeScript SDK is designed for seamless integration into modern Node.js and browser environments, ensuring **Type Safety** through automatic schema generation and robust **Abstraction** of the underlying API complexities.[^1] [^2]



## 1. Installation and Setup



Install the SDK using npm or yarn:



```bash
npm install @qbank/connect

# or

yarn add @qbank/connect
```



Initialize the client using your `clientId` and `clientSecret`. The SDK manages the entire OAuth 2.0 Client Credentials token lifecycle automatically, including caching and refreshing the Bearer token before it expires.[^3]



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



## 2. Idiomatic Idempotency Handling



For all transactional endpoints (`POST`, `PUT`), the SDK ensures transactional integrity by managing the `Idempotency-Key` header automatically.



*   **First Attempt:** The SDK generates a new UUID for the header if the key is not explicitly provided.

*   **Retries:** If a network failure occurs, the client should retry using the *same* `Idempotency-Key` to ensure the payment is processed **exactly once**.[^4] [^5]



### 2.1. Safe Payment Submission (Manual Retry Example)



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
      idempotencyKey: key, // Supply key for idempotency guarantee
    });
    console.log('Payment accepted. Status:', response.status);
    return response;
  } catch (error) {
    if (error instanceof IdempotencyKeyConflictError) {
      // The API returned a 409 Conflict. DO NOT retry submission.
      console.warn(`Idempotency conflict detected. Reason: ${error.detail}`);
      // Recommended: Check status using GET /payments/{id}
    } else if (error instanceof PermissionDeniedError) {
      // The API returned a 403 Forbidden.
      throw new Error('Unauthorized API call. Check scopes.');
    } else {
      // Handle other errors (e.g., QBankServiceUnavailableError for 5xx)
      throw error;
    }
  }
}
createPaymentWithRetry(requestKey, achRequest);
```



## 3. Type Safety and Structured Errors (DX)



The SDK utilizes strong typing (e.g., Zod schemas) to make API inputs and outputs explicit, allowing errors to be caught during development (in the IDE) rather than at runtime.[^1]



All status-code-based API errors (e.g., 400, 403, 409) are translated into **Named Error Classes** (`IdempotencyKeyConflictError`, `PermissionDeniedError`). This abstraction allows developers to handle failures programmatically using `try/catch` blocks, avoiding the need to parse raw HTTP status codes or error messages.[^1]


## References

[^1]: QBank TypeScript SDK Readme, "Core Features and Design Principles (Developer Experience)."
[^2]: TypeScript SDK Source Code, `src/client.ts` (Client Initialization).
[^3]: OAuth 2.0 Client Credentials Flow, RFC 6749 (Token Lifecycle).
[^4]: API Design Guidelines, "Idempotency and Network Failure Handling."
[^5]: QBank Connect API Reference, "Idempotency-Key Requirements."