# API outline

Base path `/api`. JSON over HTTPS. All endpoints except `/auth/*` need
`Authorization: Bearer <JWT>`. Money in the API is a decimal string. The
database stores integers (minor units and micro-rates), see the ERD.

## Endpoints

| Method | Path | Purpose | Idempotency-Key |
|---|---|---|---|
| POST | /auth/register | Create account | - |
| POST | /auth/login | Get JWT | - |
| GET | /rates | Current bid/ask rates | - |
| GET | /rates/{code}/history?days=30 | Rate history | - |
| GET | /wallet | Wallet balances | - |
| POST | /wallet/fund | Simulated funding | Required |
| POST | /trades | Buy or sell | Required |
| GET | /trades | Transaction history | - |
| GET | /trades/by-key/{key} | Outcome of an earlier request | - |

## Representative requests and responses

### POST /auth/login
```json
{ "email": "user@example.com", "password": "********" }
```
200:
```json
{ "token": "<jwt>", "expiresInSeconds": 3600 }
```

### GET /rates
200:
```json
{
  "effectiveDate": "2026-10-30",
  "fetchedAt": "2026-10-30T09:15:02Z",
  "rates": [
    { "currency": "USD", "bid": "3.5812", "ask": "3.6534" },
    { "currency": "EUR", "bid": "4.1920", "ask": "4.2764" }
  ]
}
```

### POST /wallet/fund (simulated)
Header: `Idempotency-Key: 6b1f0c1e-7d52-4a5e-9a39-2a3f4b0c9d11`
```json
{ "amount": "1000.00", "simulated": true }
```
201:
```json
{
  "transactionId": "a2c4...",
  "type": "FUNDING",
  "amount": "1000.00",
  "currency": "PLN",
  "simulated": true,
  "balances": { "PLN": "1000.00" }
}
```

### POST /trades
Header: `Idempotency-Key: 0f8e2c4a-91b3-4c6d-8d0a-5e7b1c2d3f44`
```json
{ "side": "BUY", "currency": "USD", "amount": "100.00" }
```
201 (the server chooses the rate at execution time):
```json
{
  "transactionId": "c91d...",
  "side": "BUY",
  "currency": "USD",
  "amount": "100.00",
  "rate": "3.6534",
  "plnAmount": "365.34",
  "rateDate": "2026-10-30",
  "createdAt": "2026-10-30T09:20:11Z",
  "balances": { "PLN": "634.66", "USD": "100.00" }
}
```
Repeated request with the same key and body: same response body, header
`Idempotent-Replayed: true`, no new transaction.

### GET /trades/by-key/{key}
- 200: the stored result (same body as the original 201). The server committed.
- 404: `{ "error": "not_found" }`. The server has no record of this key, so
  the same request can be safely resent with the same key.

### GET /trades
200:
```json
{
  "items": [
    { "transactionId": "c91d...", "type": "BUY", "currency": "USD",
      "amount": "100.00", "rate": "3.6534", "plnAmount": "365.34",
      "createdAt": "2026-10-30T09:20:11Z" }
  ]
}
```

## Errors

Format: `{ "error": "<code>", "message": "<text>" }`

| Status | Code | When |
|---|---|---|
| 400 | invalid_request | Missing or malformed field, missing Idempotency-Key |
| 401 | unauthorized | Missing, invalid or expired JWT |
| 403 | forbidden | Access to another user's data |
| 404 | not_found | Unknown key in status query |
| 422 | insufficient_balance | Not enough funds for the operation |
| 422 | idempotency_key_reuse | Same key with a different request body |
| 503 | rate_unavailable | No usable rate (NBP unavailable and no stored rate) |

## Duplicate protection rules

1. The client creates a UUID per data-changing operation and keeps it until
   the outcome is known.
2. The service stores key, request hash and first response in the same
   database transaction as the balance change.
3. Same key and same body returns the stored response. Same key and a
   different body returns 422.
4. After a lost response the client first calls `GET /trades/by-key/{key}`.
   200 means committed, 404 means resend with the same key.
