# Python SDK Integration Guide

The QBank Python SDK is designed to be both type-safe and idiomatic in nature and abstracts away the OAuth 2.0 complexities and manages the transactional integrity automatically.[^30] [^31]


## 1. Installation & Initialization

SDK is available for installation via pip:

```bash

pip install qbank-connect

```

You can initialize the client with your `client_id` and `client_secret` using the following method. The SDK will automatically begin the OAuth 2.0 Client Credential flow and manage token caching and refreshing for you.[^4]



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



## 2. Idempotence Layer (Important)

All transactional methods (e.g., `client.payments.create_ach()`) generate a new UUID for the `Idempotency-Key` header for each call by default unless you explicitly pass one for a retry use case.[^32]

### 2.1. Safe ACH Submission (Automated Idempotence)

```python

# The SDK will create an automatic Idempotency-Key for this call.
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
# If a 409 Conflict is returned, this exception will be raised.
# Instead of attempting to retry immediately we check the status instead.
    print(f"Conflict detected. Checking transaction status: {e.detail}")
    # Logic to call client.payments.get_payment_status(e.transaction_id)
except QBankPermissionDeniedError:
    print("Authentication error: Check scopes and token.")
```



## 3. Type-Safe Data Structures and Named Exceptions

The Python SDK uses Pydantic Models to define data objects.[^30] This means that you receive the advantage of type checking in your IDE, which reduces the number of times you get a `400 Bad Request` due to incorrect data format.[^30]

In addition, the SDK converts all generic HTTP status codes (e.g., 403, 409, 500) into specific, named exceptions (e.g., `QBankIdempotencyConflict` for a 409).[^30] Developers are able to use idiomatic Python `try/except` blocks to address failures without having to parse strings to determine the cause of failure.[^31] See the Canonical Error Map in the API Reference for a complete list of translations between HTTP status codes and SDK exceptions.


## 4. Documents Cited

[^4]: PCI DSS Requirement 10: Track and Monitor All Access to Network Resources and Cardholder Data.
[^30]: QBank Developer Documentation, "Error Code Directory."
[^31]: QBank Python SDK Readme, "Core Features and Design Principles."
[^32]: Python SDK Source Code, `qbank_connect/payments.py` (Idempotency Logic).