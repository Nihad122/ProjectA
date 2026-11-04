# Requirements (Lab 02)

## Objective and scope

A mobile app, web service and database for a simulated currency exchange
Users register, add simulated funds, view current and historical NBP rates,
buy and sell currencies, and view wallet balances and transaction history
The system must behave safely on an unreliable mobile network

Out of scope: real money, real payment providers, trading other than
PLN <-> foreign currency, push notifications

## Functional requirements

| ID | Requirement |
|---|---|
|FR1 | The user can register and log in with e-mail and password. |
|FR2 | The user can add simulated funds to the PLN wallet. The UI marks this clearly as a simulation. |
|FR3 | The user can view current exchange rates (bid/ask) from NBP. |
|FR4 | The user can view historical rates of a currency (e.g. last 30 days). |
|FR5 | The user can buy and sell a currency. The service updates the relevant wallet balances and the transaction record in one operation. |
|FR6 | The user can view wallet balances per currency. |
|FR7 | The user can view transaction history. |
|FR8 | Every displayed rate shows when it was obtained; cached rates are marked as cached. |
|FR9 | The service validates all data-changing requests (amount, currency, sufficient balance) and records the authoritative outcome. |
|FR10 | A user can access only their own wallet and transactions (authorisation). |

## Non-functional and network requirements

The values below are design choices of this project, not given by the course
Justifications are written for the report

| ID | Requirement | Value / rule | Justification |
|---|---|---|---|
|NFR1 | Request timeout | 5 s | Most successful mobile responses arrive earlier, longer waits leave the user without feedback. |
|NFR2 | Retry of read requests | Max 3 attempts, waits 1 s, 2 s, 4 s | Reads are safe to repeat, backoff avoids overloading the service. |
|NFR3 | Retry of data-changing requests | Max 3 attempts, always with the same "Idempotency-Key" | A repeated request must have one effect. |
|NFR4 | Rate cache on the mobile app | Kept up to 24 h, marked stale after 15 min | NBP rates change once per business day, but the user must see data age. |
|NFR5 | Offline behaviour | Reading cached data allowed, buy, sell and fund disabled, no action queue | Avoids executing an action later at a rate the user did not see. |
|NFR6 | Synchronisation | On reconnect, refresh rates, wallet and history from the service; the service state wins | Service is the authoritative source. |
|NFR7 | Consistency | Each buy/sell/fund runs in one database transaction | Partial updates must never be visible. |
|NFR8 | Success display | The app never shows success without server confirmation | Prevents false confirmations after lost responses. |
|NFR9 | Duplicate protection | The service stores request ID and first response, same ID returns the stored result, same ID with a different body is rejected | One effect per request. |
|NFR10 | Uncertain outcome | If the response is lost, the app shows "result uncertain" and queries the request status before any new attempt | Distinguishes "committed, response lost" from "never received". |
|NFR11 | Response time | 95% of API calls below 500 ms on a local network, excluding calls to NBP (rates are stored by the service) | Measurable target for the prototype environment. |
|NFR12 | Security | Passwords hashed with bcrypt, JWT valid 60 min, HTTPS for external calls, no secrets in the repository | Basic protection of accounts and data. |
|NFR13 | Usability | Offline state, stale data and uncertain results are shown in plain language | The user must understand what happened. |
|NFR14 | Reproducibility | README describes setup, demo and failure reproduction ('FAULT_MODE') | Required by the course. |

## Use cases

Actors: User, NBP API (external system)

| Use case | Notes |
|---|---|
|Register | |
|Log in | |
|Add simulated funds | |
|View current rates | uses NBP API; falls back to cache |
|View rate history | uses NBP API; falls back to cache |
|Buy currency | includes "Validate rate and balance" |
|Sell currency | includes "Validate rate and balance" |
|View wallet | |
|View transaction history | |
|Check status of uncertain request | extends Buy / Sell / Add funds when the response is lost |
