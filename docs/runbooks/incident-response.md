# Incident response runbook

How to classify, escalate, and resolve Trident incidents. Per-alert first steps
live in [alerts.md](alerts.md); this document covers severity and coverage.

## Severity levels

| Sev | Definition | Response target | Page? |
|-----|------------|-----------------|-------|
| SEV1 | Data loss/corruption, wrong data served, total outage, or security incident | Ack 5 min, mitigation started 15 min | Yes, immediately |
| SEV2 | Major degradation: sustained indexer lag, elevated 5xx, partial outage | Ack 15 min, mitigation 1 h | Yes |
| SEV3 | Minor degradation with a workaround, single-component warning | Next business day | No |

## Examples (network-agnostic)

- SEV2: `TridentIndexerLagCritical` (>1000 ledgers behind) — see [alerts.md](alerts.md).
- SEV2: `TridentAPIHTTP5xxRateCritical`.
- SEV3: `TridentIndexerLagWarning` that recovers within 30 minutes.
- SEV3: single RPC endpoint failing while failover endpoints are healthy.

## Mainnet triage differs from testnet

Mainnet carries real user funds and reputational stakes. Testnet incidents are
triaged at the levels above; **mainnet incidents are escalated one level for the
same symptom**, with these specific rules:

| Symptom | Testnet | Mainnet |
|---------|---------|---------|
| Indexer lag warning (>200 ledgers, 10 min) | SEV3 | SEV2 |
| Indexer lag critical (>1000 ledgers) | SEV2 | SEV1 |
| All RPC endpoints failing / `TridentIndexerRPCErrorRateCritical` | SEV2 | SEV1 |
| API 5xx rate high | SEV3 | SEV2 |
| Parse-error spike (events dropped or mis-decoded) | SEV3 | SEV1 — silent data loss |
| Wrong/missing asset tags on SAC events | SEV3 | SEV1 |
| Indexer running with `NETWORK` not matching the RPC endpoint | SEV3 | SEV1 — stop the indexer, fix config, re-index affected range |
| Partition missing for current ledger range (insert failures) | SEV2 | SEV1 |

Additional mainnet expectations:

- Announce SEV1/SEV2 on the status page / customer channel within 30 minutes.
- Never "wait and see" on lag or parse errors; mainnet consumers act on the data.
- A post-incident review is mandatory for every mainnet SEV1/SEV2.
- Any mainnet change made during an incident (config, rollback) is logged in the incident channel.

## On-call coverage

Rotation, escalation contacts, and paging tooling: **[FILL IN]** (tracked
separately).

### Heightened coverage around mainnet cutover

Around the mainnet go-live date (**[FILL IN: date/time UTC]**):

- From T-24h to T+72h, a primary **and** a secondary on-call are staffed, with
  a named incident commander available.
- Acknowledge targets are halved (SEV1: 2–3 min; SEV2: 7 min) for this window.
- The deploy owner stays online for the first 24 hours after cutover and
  watches lag, heartbeat, parse-error, and 5xx dashboards.
- No unrelated deploys or migrations during the window.
- Rollback criteria and owners are agreed before cutover (see
  [deployment.md](../deployment.md#rollback)).
- After T+72h with no SEV1/SEV2, return to the normal rotation.
