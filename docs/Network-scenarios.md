# Network scenario matrix (Lab 02)

## Reads vs actions requiring authoritative confirmation

| Type | Operations | Offline | Retry | Confirmation |
|---|---|---|---|---|
| Read | View rates, rate history, wallet, transaction history | Allowed from local cache, with freshness label | Safe, max 3 attempts (1 s, 2 s, 4 s) | Not needed; data shown with its timestamp |
| Action | Register, add simulated funds, buy, sell | **Blocked** (decision: no offline queue) | Only with the same `Idempotency-Key`, max 3 attempts | Server response required; success is never shown without it |

Decision: offline changes are **blocked**, not queued. Reason: a queued trade
would run later at a rate the user never saw, and it needs more consistency
logic than this project justifies. Queuing is listed as a possible extension.

## Scenario matrix

Server state means the authoritative database state. Each scenario has a
test ID (S1 ...) used later in the failure-test evidence table.

| ID | Scenario | Expected user-visible state | Expected server state | Testable check |
|---|---|---|---|---|
| S1 | No connection before a request | Offline banner; buy/sell/fund disabled; cached data shown with "last updated" time | No request received; nothing changed | Disable network, open app, tap actions: no request in server log |
| S2 | Connection lost after submitting a buy | "Result uncertain"; no success message; status check starts | Either committed or not received (unknown to the app) | Use `FAULT_MODE` to cut the connection after send; the app shows uncertain state |
| S3 | Server committed, response lost | After reconnect, status check finds the request: app shows the completed trade; wallet refreshed | One transaction, balances changed once | `FAULT_MODE=drop_response`; DB has exactly 1 transaction; `GET /trades/by-key/:key` returns 200 |
| S4 | Server never received the request | Status check finds nothing; app offers or performs safe resend with the same key | No transaction before the resend; exactly one after | `FAULT_MODE=drop_request`; status returns 404; after resend, 1 transaction |
| S5 | Timeout (no response within 5 s) | Timeout message; "Result uncertain" for actions; retry per policy | Unknown to the app until status is checked | `FAULT_MODE=delay` with a delay above 5 s; the app stops waiting at 5 s |
| S6 | Delayed response (below 5 s) | Loading indicator; success shown only after the response arrives | One transaction after completion | `delay` of about 3 s; no timeout, one transaction |
| S7 | Retry of a read request | Brief loading state; data shown when a retry succeeds; error after 3 failed attempts | No change | Fail first 2 reads, third succeeds; server log shows 3 requests |
| S8 | Repeated request with the same key | Same result shown once; no second trade in history | One transaction; the second response equals the stored first response | Send the same request twice; DB has 1 transaction |
| S9 | Same key, different body | Error message "request conflict" | No change from the second request | Send the same key with a different amount; server returns 422 |
| S10 | Cached display while offline | Rates and wallet from cache with timestamp; marked stale after 15 min | No request | Go offline after one successful read; screens still show data with age |
| S11 | Connectivity restored | Offline banner disappears; data refreshes; pending uncertain requests resolved | State unchanged by reconnect itself; app reads it | Restore network; app shows server wallet and history |
| S12 | Synchronisation after reconnect | Wallet, history and rates replaced by server values (server wins) | Authoritative state | Change data on the server while the app is offline; after reconnect the app matches the server |
| S13 | Two competing buy requests at once | One may fail with "insufficient balance" if funds do not cover both; balance never negative | Operations applied one after the other; balance consistent | Send two parallel requests with different keys; final balance = initial minus accepted trades only |
| S14 | Insufficient balance | Clear error message; no change on screen | No transaction | Buy above balance: 4xx response, no DB change |
| S15 | Expired login (JWT) during an action | Login prompt; the action is not reported as done | Not applied, or applied once if it was already committed | Expire the token; after login, the same key gives one effect |
| S16 | NBP API unavailable | Last stored rate shown with its date and "cached" label; trading allowed only if the stored rate is recent enough (rule documented in report) | Stored rates unchanged | Use the fake rate source in failure mode |

## Notes

- A trade's rate is chosen by the server at execution time and returned in
  the response, so a stale displayed rate cannot change the executed price
  without the user seeing the final value.
- Open decision for the report: maximum age of a stored rate that still
  allows trading (proposal: the latest NBP business-day rate).
