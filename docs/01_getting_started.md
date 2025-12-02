# Getting Started: Onboarding and Authentication
QBank Connect uses the **OAuth 2.0 Client Credentials Grant** for server-to-server B2B authentication, which is ideal for ERP systems that require automated, non-interactive access to resources.

## 1. Partner Onboarding Flow

1.1. **Registration:** Contact your QBank Account Manager to register your ERP platform. You will receive a unique **`client_id`** and a secure **`client_secret`**.

1.2. **Scope Approval:** During registration, you must specify the exact permissions (scopes) your application requires (e.g., `transactions.read`, `payments.ach.write`). This adheres to the principle of least privilege.

1.3. **Credential Storage:** Store the `client_id` and `client_secret` securely in an environment vault (e.g., AWS Secrets Manager, Azure Key Vault).

## 2. OAuth 2.0 Client Credentials Flow
Your application uses the `client_id` and `client_secret` to obtain a short-lived **Bearer Token**.

### 2.1. Request an Access Token
Send a `POST` request to the token endpoint:

| Field | Value |
| :--- | :--- |
| **Endpoint** | `POST https://auth.qbankconnect.com/token` |
| **Content-Type** | `application/x-www-form-urlencoded` |

**Request Body (Form URL Encoded)**

| Parameter | Description |
| :--- | :--- |
| `grant_type` | Must be `client_credentials`. |
| `client_id` | The unique identifier provided during registration. |
| `client_secret` | The secret key provided during registration. |
| `scope` | A space-separated list of required permissions (e.g., `accounts.read payments.ach.write`). |

### 2.2. Handle the Response
A successful response returns the access token, which is valid for a limited time (typically 3600 seconds/1 hour).

```json
{
"access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
"token_type": "Bearer",
"expires_in": 3600,
"scope": "accounts.read payments.ach.write"
}

```



### 2.3. API Call with the Token



Use the `access_token` in the `Authorization` header for all subsequent API calls:



| Header | Value |
| :--- | :--- |
| `Authorization` | `Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...` |



The QBank SDKs automatically manage this entire token lifecycle, including renewal and expiration handling (see [Python SDK](sdk_integration/sdk_python.md)).



## 3. Quickstart: Retrieving a Real-Time Balance



To retrieve the current balance for an account, use the `/balances` endpoint, which is a **real-time synchronous** call to the core system.



| Field | Value |
| :--- | :--- |
| **Endpoint** | `GET /v1/balances/{accountId}` |
| **Scopes Required** | `balances.read` |



```bash
curl --location '[https://api.qbankconnect.com/v1/balances/a1b2c3d4-e5f6-7890-1234-567890abcde](https://api.qbankconnect.com/v1/balances/a1b2c3d4-e5f6-7890-1234-567890abcde)' \
--header 'Authorization: Bearer <YOUR_ACCESS_TOKEN>'
```



This request will return a `200 OK` response with the `available_balance` and `ledger_balance`.

