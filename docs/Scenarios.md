# Lab 01: Architecture, scenarios and risks

## Architecture and data owners

```
[Mobile app + local cache] --wireless, unreliable--> [Web service] ---> [SQLite database]
                                                          |
                                                          +--HTTPS--> [NBP API]
```

## Component - Owns / stores - Authoritative?
Database + service - users, wallet balances, transactions, idempotency keys - Yes (single source of truth) 

NBP API - official exchange rates - Yes, for rates (external) 

Mobile app - cached rates, last seen balances and history - No (cache only) 

Wireless segment: between the mobile app and the web service. Every
data-changing request carries a unique request ID (`Idempotency-Key`), so a
repeated request has only one effect.

## User journeys

### Normal journeys

**N1. Registration and first look at rates**
1. User registers and logs in.
2. User adds simulated funds (UI shows "SIMULATION").
3. User opens the rates screen and sees current rates with their date.

**N2. Buying a currency**
1. User selects USD and an amount, and confirms.
2. Service validates the request, updates both wallet balances and records the transaction in one database transaction.
3. App shows success only after the server confirms; wallet and history are updated.

### Journeys interrupted by connectivity loss

**I1. Connection lost after submitting a buy request**
1. User taps "Buy"; the request is sent with a request ID.
2. The connection drops before the response arrives.
3. App shows "Result uncertain" (not success, not failure).
4. When possible, the app queries the status of that request ID.
   Server committed: status found, balance already changed once.
   Server never received it: not found, the same request can be safely resent with the same ID.
5. No duplicate transaction is created.

**I2. App opened without a connection**
1. App shows an offline banner.
2. Cached rates are shown with "last updated: HH:MM".
3. Buy, sell and fund buttons are disabled.
4. When the connection returns, the app refreshes rates, wallet and history from the server.

## Initial risk list

## Risk - Mitigation 

Risk1 - Stale rate shown; user decides on old data - Show rate date and fetch time; mark data older than 15 min as stale; server determines the rate at execution time 

Risk2 - Uncertain outcome: server committed but response lost - Request ID + status endpoint (`GET /trades/by-key/:key`) 

Risk3 - Server never received the request - Safe resend with the same request ID 

Risk4 - Duplicate request creates two operations - Server stores request ID and first response; repeated ID returns the stored result 

Risk5 - Overlapping requests corrupt wallet balance - Database transaction, balance check inside the transaction 

Risk6 - NBP API unavailable - Use last stored rate with timestamp; fake rate source for tests 

Risk7 - JWT expires during an operation - Re-login flow; the request keeps the same ID when resent 

Risk8 - Concurrent writes in SQLite - Short transactions, single writer connection 

Risk9 - Cached data mistaken for current data - Freshness label on every cached screen 
