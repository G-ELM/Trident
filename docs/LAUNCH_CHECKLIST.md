# Launch Checklist

Gate list for a production launch. Every row must be checked off with an owner,
date, and evidence. A row left unverified fails the launch.

## Mainnet configuration (hard gate)

| # | Check | How to verify | Owner / date / evidence |
|---|-------|---------------|-------------------------|
| M1 | `NETWORK=mainnet` is set on the indexer (not defaulted) | `fly secrets list -a trident-indexer` / inspect `.env`; indexer start logs must NOT contain "NETWORK is not set — DEFAULTING TO TESTNET" | |
| M2 | `STELLAR_RPC_URL(S)` point at real mainnet endpoints and are reachable | `curl -s -X POST $URL -H 'content-type: application/json' -d '{"jsonrpc":"2.0","id":1,"method":"getNetwork"}'` returns `"passphrase":"Public Global Stellar Network ; September 2015"` for **every** URL; `getLatestLedger` returns a current sequence | |
| M3 | `NETWORK_PASSPHRASE` unset, or equal to the mainnet passphrase | inspect env | |
| M4 | `TRACKED_SAC_ASSETS` matches the reviewed list in [mainnet-sac-assets.md](mainnet-sac-assets.md) and derived ids were re-verified | follow its verification procedure | |
| M5 | Partitions cover the current mainnet ledger height plus headroom | take `latest_ledger` from M2, then in Postgres confirm a `soroban_events` partition covers it and the next range (`\d+ soroban_events`); add with `SELECT create_soroban_partition(start_ledger, end_ledger);` ([migration 0017](../database/migrations/0017_soroban_events_partitioning.sql)). Events landing in the default partition is a failure | |

## Core launch rows

| # | Check | Owner / date / evidence |
|---|-------|-------------------------|
| 1 | Alerts ([monitoring/alerts.yml](../monitoring/alerts.yml)) loaded and firing test passed; runbook links valid ([alerts.md](runbooks/alerts.md)) | |
| 2 | Backup and restore tested | |
| 3 | Soak test passed | |
| 4 | Chaos test passed | |
| 5 | End-to-end user journey verified | |
| 6 | SDKs released and documented | |
| 7 | Docs reviewed ([deployment.md](deployment.md)) | |
| 8 | On-call staffed incl. cutover coverage ([incident-response.md](runbooks/incident-response.md)) | |
| 9 | Rollback plan rehearsed ([deployment.md](deployment.md#rollback)) | |
