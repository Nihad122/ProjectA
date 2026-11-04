# Implementation plan, possible extensions and sources

## Implementation plan

Individual project: all responsibilities belong to one person (Nihad).
Weeks are teaching weeks; the plan is a proposal and may be adjusted.

| Phase | Weeks / labs | Work | Output |
|---|---|---|---|
| Analysis and design | 1-4 (Lab 01-04) | Topic, architecture, requirements, UML, ERD, API outline, failure scenarios | M1: PDF report, diagrams, Git revision |
| Working prototype | 5-8 | Project skeleton, SQLite migrations, registration and login (JWT), rates from NBP (or documented test substitute), one read and one write, setup notes | M2: runnable client, service, database |
| Core features | 9-10 | Funding, buy and sell with database transactions, wallet and history screens | Full normal path |
| Network resilience | 11-13 | Timeout, retry, offline state, cache with freshness, idempotency keys, status endpoint, `FAULT_MODE`, failure tests with evidence | M3: failure-test evidence table |
| Final system | 14-15 | Bug fixing, final report, short user manual, presentation, live demo | M4: final package |

Responsibilities (all: Nihad Nurmammadov): mobile app, web service, database, tests,
documentation, demonstration.

## Possible extensions

- Queue offline trade intentions (explicit intent, executed only after server confirmation and user re-confirmation at the current rate)
- Push notifications for rate alerts
- More currencies and rate charts
- Background synchronisation
- Biometric login
- Administrator view for monitoring failed or uncertain requests

## Sources

- National Bank of Poland, NBP Web API documentation (api.nbp.pl)
- IETF RFC 9110, HTTP Semantics (idempotent methods, status codes)
- IETF RFC 7519, JSON Web Token
- IETF draft, The Idempotency-Key HTTP Header Field (httpapi working group)
- Documentation of the chosen tools: React Native and Expo, Node.js and Express, SQLite
- Course materials: Laboratory project handbook and Project_Documentation_Template_EN.docx
