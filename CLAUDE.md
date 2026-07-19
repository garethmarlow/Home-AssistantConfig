# Home-AssistantConfig — Agent Kernel

Gareth's live Home Assistant config. This file is the router and safety boundary, not the whole manual. Read it before touching the repo, then load only the leaf needed for the task.

## Operating principle: load just enough

1. Read this kernel.
2. Use the routing table below.
3. Open only the relevant package or leaf.
4. Do not scan runtime state, logs, secrets or the whole tree by default.

## What this is

Home Assistant **Container** install using the plain `homeassistant/home-assistant` Docker image on a Synology NAS named `EBW`. It is not HAOS or Supervised: there is no Supervisor or Add-on store.

The live bind-mounted config is `/config` inside the container and `/Volumes/docker/homeassistant/config` from Gareth's Mac. Edits here are live configuration edits, not work on a disposable clone.

## Routing

| If the task is about… | Read next |
|---|---|
| Access, git, validation, reload, restart or drift | [`docs/agent/access-and-operations.md`](docs/agent/access-and-operations.md) |
| Energy/carbon chart, Sankey, panel layout, colour or accessibility | [`docs/agent/dashboard-contracts.md`](docs/agent/dashboard-contracts.md) |
| A package or entity definition | [`packages/_index.md`](packages/_index.md), then only the relevant YAML |
| LifeOS integration or Cowork cross-system work | [`lifeos_bridge/contract.md`](lifeos_bridge/contract.md) and [`lifeos_bridge/entity-map.yaml`](lifeos_bridge/entity-map.yaml) |
| Available agent documentation | [`docs/agent/_index.md`](docs/agent/_index.md) |

## Non-negotiable safety boundary

- Do not read or expose `secrets.yaml`, `.env`, `.storage/`, databases, logs, SSH keys, device registries or entity registries unless the task explicitly requires it.
- Do not attempt to reach the NAS or HA LAN from an agent sandbox. File access works only through the mounted folder.
- Do not use browser automation for NAS or HA administration. Gareth runs required commands through Synology DSM → Container Manager → `homeassistant` → Open terminal.
- Do not reload, restart, trigger a physical action or make a security-sensitive change without Gareth's approval.
- Preserve unrelated work. This live repo may be dirty.

## Change discipline

- Keep secrets and generated/runtime state untracked.
- For YAML changes, inspect the focused diff and validate configuration before reload or restart.
- UI-created automations, scenes and scripts can drift from git; include them only deliberately.
- If a new tracked file does not match the narrow allowlist in `.gitignore`, add the smallest explicit exception.
- Record durable task-specific rules in the routed leaf, not in this kernel.

## Cross-system authority

LifeOS owns human intent, commitments, preferences, interpretation and memory. Home Assistant owns physical state, devices, event history, automation and actuation. Cowork may orchestrate across both attached folders but exchanges only curated derived facts and expiring semantic intents through `lifeos_bridge`.
