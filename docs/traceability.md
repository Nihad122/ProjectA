# Requirements -> use cases -> responsible components

Mobile components: RatesScreen, WalletScreen, TradeScreen, ApiClient,
RetryPolicy, ConnectivityMonitor, LocalCache, PendingIntentStore

Service components: AuthController, RatesController, TradeController,
TradeService, IdempotencyService, RateService, NbpClient, Repository

## Functional requirements

| Requirement | Use case | Responsible components | Decides the outcome |
|---|---|---|---|
| FR1 Register and log in | Register, Log in | Mobile: ApiClient. Service: AuthController, Repository | Service |
| FR2 Simulated funding | Add simulated funds | Mobile: TradeScreen, ApiClient. Service: TradeController, TradeService, IdempotencyService | Service (commit) |
| FR3 Current rates | View current rates | Mobile: RatesScreen, LocalCache. Service: RatesController, RateService, NbpClient | Service (rate source: NBP) |
| FR4 Rate history | View rate history | Mobile: RatesScreen, LocalCache. Service: RatesController, RateService, NbpClient | Service |
| FR5 Buy and sell | Buy currency, Sell currency, Validate rate and balance | Mobile: TradeScreen, ApiClient, PendingIntentStore. Service: TradeController, TradeService, RateService, Repository | Service (commit) |
| FR6 Wallet balances | View wallet | Mobile: WalletScreen, LocalCache. Service: TradeController or wallet endpoint, Repository | Service |
| FR7 Transaction history | View transaction history | Mobile: WalletScreen, LocalCache. Service: TradeController, Repository | Service |
| FR8 Rate time shown | View current rates | Mobile: RatesScreen, LocalCache (timestamp). Service: RateService (effectiveDate, fetchedAt) | Service provides data, app displays it |
| FR9 Server-side validation | Buy currency, Sell currency, Add simulated funds | Service: TradeService, Repository | Service |
| FR10 Authorisation | All account use cases | Service: AuthController and JWT check on every endpoint | Service |

## Non-functional and network requirements

| Requirement | Use case | Responsible components | Related scenarios |
|---|---|---|---|
| NFR1 Timeout 5 s | Buy, Sell, Add funds, reads | Mobile: ApiClient | S5, S6 |
| NFR2 Read retry | View rates, wallet, history | Mobile: ApiClient, RetryPolicy | S7 |
| NFR3 Write retry with same key | Buy, Sell, Add funds | Mobile: ApiClient, PendingIntentStore | S3, S4, S8 |
| NFR4 Cache and freshness | View rates, wallet, history | Mobile: LocalCache, RatesScreen, WalletScreen | S10 |
| NFR5 Offline behaviour | All actions | Mobile: ConnectivityMonitor, TradeScreen | S1 |
| NFR6 Synchronisation | View wallet, history, rates | Mobile: ConnectivityMonitor, LocalCache, ApiClient | S11, S12 |
| NFR7 Consistency (one transaction) | Buy, Sell, Add funds | Service: TradeService, Repository | S13, S14 |
| NFR8 No success without confirmation | Buy, Sell, Add funds | Mobile: TradeScreen. Service: TradeController | S2, S5 |
| NFR9 Duplicate protection | Buy, Sell, Add funds | Service: IdempotencyService | S8, S9 |
| NFR10 Uncertain outcome | Check status of uncertain request | Mobile: PendingIntentStore, ApiClient. Service: TradeService (statusByKey) | S2, S3, S4 |
| NFR11 Response time | All | Service: RatesController, RateService (stored rates) | Performance test |
| NFR12 Security | Register, Log in, all | Service: AuthController | S15 |
| NFR13 Usability | All | Mobile: screens | S1 to S12 |
| NFR14 Reproducibility | - | Repository README, FAULT_MODE in service | All |
