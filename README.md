# Mobile Currency Exchange System

Course project (Project A), individual work

## 1. Topic and scope

Mobile app, web service and database for a simulated currency exchange:
registration and login, simulated account funding, current and historical
exchange rates (National Bank of Poland API), buying and selling currencies,
wallet balances and transaction history.

Account funding is **virtual** and is labelled as a simulation in the UI.
The project focuses on unreliable mobile connectivity: offline state, timeout,
retry, local cache, synchronisation and duplicate-request protection.

## 2. Technology choices

| Layer | Choice |
|---|---|
| Mobile client | React Native (Expo), TypeScript |
| Web service | Node.js, Express, TypeScript |
| Database | SQLite (migrations in `server/migrations`) |
| Mobile cache | AsyncStorage |
| Authentication | E-mail + password (bcrypt), JWT |
| External data | NBP Web API (Table A: history, Table C: bid/ask) |

Communication path: mobile app -> HTTPS/JSON -> Express service -> SQLite;
the service calls the NBP API and stores a copy of the rates.

## 3. Repository structure

```
/mobile    Expo mobile app
/server    Express service and database migrations
/docs      Diagrams, scenarios, report
/tests     Network-failure test scripts
README.md
```

## 4. Setup

_To be completed in M2._

- Requirements: Node.js (version TBD), npm, Expo Go or an emulator
- Install: `cd server && npm install`, `cd mobile && npm install`

## 5. Configuration

Copy `server/.env.example` to `server/.env` and fill in the values.
No secrets are stored in the repository.

_Variables to be listed in M2 (e.g. `PORT`, `JWT_SECRET`, `NBP_BASE_URL`, `FAULT_MODE`)._

## 6. Running the components

_To be completed in M2._

- Service: `cd server && npm run dev`
- Mobile app: `cd mobile && npx expo start`

## 7. Network-failure demonstration

_To be completed in M3._

The service will support a `FAULT_MODE` setting to reproduce failures:

- `delay` - slow response
- `drop_response` - request is processed, but the response is not sent
  (server committed, response lost)
- `drop_request` - request is not processed at all (server never received it)

Scenarios and expected results are listed in `docs/scenarios.md`.

## 8. External API and test-data assumptions

- NBP publishes rates on business days only; on weekends and holidays the
  last published rate is used and its effective date is shown to the user.
- If the NBP API is unavailable, the service returns the last stored rate
  with its timestamp. For tests, a local fake rate source replaces the NBP API.
- Test users and balances are created by seed scripts (to be added).

## 9. Milestones

| Milestone | Git revision | Notes |
|---|---|---|
| M1 - Analysis and design | _commit hash_ | Design report, UML, ERD |
| M2 - Working prototype | _commit hash_ | |
| M3 - Network-resilient app | _commit hash_ | |
| M4 - Final system | _commit hash_ | |
