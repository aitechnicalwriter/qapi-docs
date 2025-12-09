# Getting Started
We can begin by using the **OAuth 2.0 Client Credentials Grant** to do B2B server-to-server authentication on behalf of our customers. This is perfect for our ERP clients because we need to allow them to have automatic, unattended access to their resources.

## 1. Onboarding Process

1.1. **Registering**: In order to get started, contact your QBank Account Manager to register your Platform. Your Account Manager will give you a unique **`client_id`** and a safe **`client_secret`**.

1.2. **Scope Approval**: When registering, you will need to enter the specific **permissions (scopes)** that your application needs (i.e., `transactions.read`, `payments.ach.write`) as well as who has approval to make those requests.

1.3. **Storing Credentials**: Securely store your `client_id` and `client_secret` in an **environment vault** (AWS Secrets Manager, Azure Key Vault).

## 2. Using OAuth 2.0 Client Credential Flow

You are going to use your `client_id` and `client_secret` to get a short-term **Bearer Token** from us.

### 2.1. Get an Access Token

You send a `POST` to the token endpoint:

| Field | Value |
| :--- | :--- |
| **Endpoint** | `POST https://auth.qbankconnect.com/token` |
| **Content-Type** | `application/x-www-form-urlencoded` |

**Request Body (Form URL Encoded)**

| Parameter | Description |
| :--- | :--- |
| **grant_type** | It needs to be `client_credentials`. |
| **client_id** | The unique ID provided when registering. |
| **client_secret** | The secret key provided when registering. |
| **scope** | The scope parameter needs to contain a space separated list of requested permissions (e.g., `accounts.read payments.ach.write`). |

### 2.2. Processing the Response

When sending a successful response, it includes the access token and the length of time (usually 3600 seconds/1 hour) the token remains valid.

```json
{
"access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
"token_type": "Bearer",
"expires_in": 3600,
"scope": "accounts.read payments.ach.write"
}

```



### 2.3. API Call with the Token

You will need to include the **`access_token`** in the **`Authorization`** header of each API call after obtaining the access token.

| Header | Value |
| :--- | :--- |
| `Authorization` | `Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...` |

All of these steps are included in the QBank SDKs (including renewal and token expiration), so you don't need to handle the token lifecycle yourself (see [Python SDK](sdk_integration/sdk_python.md)).

## 3. Quick Start: Retrieving a Current Balance

You can retrieve the most recent balance of an account by calling the `/balances` endpoint; however, the call to the `/balances` endpoint is a **real-time synchronous** call to the core system.

| Field | Value |
| :--- | :--- |
| **Endpoint** | `GET /v1/balances/{accountId}` |
| **Scopes Required** | `balances.read` |


```bash
curl --location '[https://api.qbankconnect.com/v1/balances/a1b2c3d4-e5f6-7890-1234-567890abcde](https://api.qbankconnect.com/v1/balances/a1b2c3d4-e5f6-7890-1234-567890abcde)' \
--header 'Authorization: Bearer <YOUR_ACCESS_TOKEN>'
```


When making this request, you will receive a `200 OK` response and the available balance and ledger balance for the account.

