# Python SDK Integration Guide



The QBank Python SDK is built to be type-safe and idiomatic, abstracting away OAuth 2.0 complexity and automatically managing transactional integrity.[^30] [^31]



## 1. Installation and Setup



The SDK is available on PyPI:



```bash

pip install qbank-connect

```



Initialize the client using your `client_id` and `client_secret`. The SDK automatically initiates the OAuth 2.0 Client Credentials flow and handles token caching and refresh.[^4]



```python

from qbank_connect import QBankConnectClient
from qbank_connect.models import AchPaymentRequest
from qbank_connect.errors import QBankIdempotencyConflict, QBankPermissionDeniedError

# Client initializes and immediately fetches a Bearer Token
client = QBankConnectClient(
    client_id="YOUR_CLIENT_ID",
    client_secret="YOUR_CLIENT_SECRET",
    scopes=["payments.ach.write", "balances.read"]
)

```



## 2. Idempotency Abstraction (Crucial)



For all transactional methods (e.g., `client.payments.create_ach()`), the SDK automatically generates a new UUID for the `Idempotency-Key` header on every call, **unless you explicitly provide one** for a retry scenario.[^32]


### 2.1. Safe ACH Submission (Automatic Idempotency)

```python
# The SDK automatically generates a new Idempotency-Key for this call. 
ach_request = AchPaymentRequest(
    source_account_id="...",
    recipient_details={...},
    amount=1000.50,
    effective_date="2025-10-31"
)

try:
    response = client.payments.create_ach(request=ach_request)
    print(f"Payment request accepted. Status: {response.status}")
except QBankIdempotencyConflict as e:
    # If a 409 Conflict is received, this exception is raised.
    # We catch it and check status instead of retrying immediately.
    print(f"Conflict detected. Checking transaction status: {e.detail}")
    # Logic to call client.payments.get_payment_status(e.transaction_id)
except QBankPermissionDeniedError:
    print("Authentication error: Check scopes and token.")
```



## 3. Type Safety and Named Exceptions



The Python SDK uses Pydantic models for data objects.[^30] This ensures you benefit from type checking in your IDE, which significantly reduces the frequency of `400 Bad Request` errors.[^30]



The SDK translates all generic HTTP status codes (e.g., 403, 409, 500) into specific, actionable, named exceptions (e.g., `QBankIdempotencyConflict` for a 409).[^30] This allows developers to handle failures using idiomatic Python `try/except` blocks, focusing on application logic rather than fragile string parsing.[^31] Refer to the Canonical Error Mapping in the **API Reference** for a full list of status code translations.


## References

[^4]: PCI DSS Requirement 10: Track and Monitor All Access to Network Resources and Cardholder Data.
[^30]: QBank Developer Documentation, "Error Code Directory."
[^31]: QBank Python SDK Readme, "Core Features and Design Principles."
[^32]: Python SDK Source Code, `qbank_connect/payments.py` (Idempotency Logic).