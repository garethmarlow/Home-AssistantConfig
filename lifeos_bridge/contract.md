# LifeOS Bridge Contract

Status: **scaffolded, disabled**. No exporter, importer, polling sensor or automation is activated by these files.

## Purpose

`lifeos_bridge` is a small file transport between Home Assistant and LifeOS when both folders are attached to the same Claude Cowork Project.

- LifeOS owns intent, commitments, preferences, interpretation and memory.
- Home Assistant owns live physical state, devices, event history, automation and actuation.
- Cowork orchestrates review and translation across the boundary.
- Transport files are generated state, never a source of truth.

## Files

| File | Producer | Consumer | Tracking |
|---|---|---|---|
| `contract.md` | Human/Cowork | Human/Cowork | Tracked |
| `entity-map.yaml` | Human/Cowork | Exporter/importer | Tracked |
| `ha-to-lifeos.json` | HA exporter | Cowork | Untracked |
| `lifeos-to-ha.json` | Cowork | HA importer | Untracked |

Do not create generated JSON until its producer exists. Consumers must tolerate a missing file.

## Envelope

Every generated message uses this envelope:

```json
{
  "schema_version": 1,
  "generated_at": "2026-07-19T10:00:00+01:00",
  "expires_at": "2026-07-19T12:00:00+01:00",
  "source": "home_assistant",
  "message_id": "unique-idempotency-key",
  "payload": {}
}
```

Requirements:

- timestamps are ISO 8601 with an explicit offset;
- `expires_at` is mandatory for live context;
- consumers ignore expired messages, unknown schema versions and unknown payload keys;
- repeated `message_id` values are idempotent;
- writes must be atomic: write a sibling temporary file, validate it, then replace the target;
- failure leaves the previous valid file intact, which will expire normally.

## HA to LifeOS

Export only facts that may change a decision or create a meaningful operational loop:

- routine or occupancy exceptions, not raw location trails;
- comfort, energy or appliance anomalies, not continuous samples;
- device or automation failures;
- bounded daily aggregates with source entities and aggregation windows.

Cowork reviews the message and writes any useful conclusion into the correct LifeOS leaf with provenance. HA never edits LifeOS notes directly.

## LifeOS to HA

Import only allowlisted semantic context, initially:

- `capacity`: `strong`, `steady`, `constrained` or `unknown`;
- `day_mode`: disabled until its vocabulary and decision use are defined;
- `quiet_until`: an expiring timestamp.

These values may influence presentation or low-risk automation selection. They must not bypass HA's heating bounds, alarms, safety constraints or device interlocks. Arbitrary domain/service/entity calls are forbidden in bridge payloads.

## Data firewall

Never transport:

- secrets, tokens, passwords or keys;
- email bodies or whole daily briefings;
- medical notes or detailed health history;
- raw presence/location history;
- recorder, InfluxDB or entity-registry dumps;
- arbitrary shell commands or HA service calls.

Use the least personal, least precise fact that still supports the decision.

## Activation gate

Before enabling either direction:

1. describe the decision the field supports;
2. select and document exact source/target entities in `entity-map.yaml`;
3. define aggregation, TTL, fallback and privacy classification;
4. test missing, malformed, duplicate and expired input;
5. inspect a focused diff and validate HA configuration;
6. obtain Gareth's approval for reload/restart and any physical effect.

Start with one read-only HA-to-LifeOS export. Add a reverse context field only after the read path proves useful.
