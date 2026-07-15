# Home-AssistantConfig — Agent Notes

Gareth's live Home Assistant config. This file governs how an agent (Claude Code, Cowork, etc.) should work in this repo. Read this before touching anything.

## What this is

Home Assistant, **Container install method** (plain `homeassistant/home-assistant` Docker image) — **not HAOS, not Supervised**. That means: no Supervisor, no Add-on store. Anything that assumes add-ons (e.g. the official "Terminal & SSH" add-on) will not work here.

Runs in Docker on a Synology NAS, hostname `EBW`, on Gareth's home LAN.

## Access topology

- **Live config on disk**: `/config` inside the `homeassistant` container on the NAS. Same files are reachable from Gareth's Mac at `/Volumes/docker/homeassistant/config` (SMB mount) — this is a live bind mount, not a copy.
- **File-level read/diff**: if this folder is connected to the agent's session, just Read/Write/Edit/Grep it directly. No network access needed.
- **Shell / git / docker-level work** (commit, push, restart container, check logs, `docker exec`): there is no add-on and no Tailscale set up for this box. The working pattern is: agent gives Gareth exact commands, Gareth runs them himself via **Synology DSM → Container Manager → select `homeassistant` container → "Open terminal"**, and pastes output back. This is the established, working path — don't try anything else.
- **Do not use Claude in Chrome or browser automation for this project.** Explicit preference — this box is reached by Gareth's own hands via Container Manager's web terminal, not by an agent driving a browser.
- **Do not attempt to reach the NAS/HA instance from an agent's own sandbox shell.** Sandboxed agent environments have no route to this LAN — file access only works via the mounted folder above, and only if a human explicitly connected it.
- No Tailscale is configured on this box and none is needed for the above workflow.

## Git / auth

- Remote: `git@github.com:garethmarlow/Home-AssistantConfig.git` (SSH, switched from HTTPS — GitHub doesn't accept password auth for git operations).
- Auth is a **deploy key** (write access) at `/config/ssh_keys/id_rsa_homeassistant`, registered on the repo's GitHub Deploy keys page. Not in the default `~/.ssh` location, so either:
  - run git with `GIT_SSH_COMMAND="ssh -i /config/ssh_keys/id_rsa_homeassistant" git <cmd>`, or
  - set it once so plain `git push`/`pull` work: `git config core.sshCommand "ssh -i /config/ssh_keys/id_rsa_homeassistant"`
- git is already initialized directly inside `/config` in the container (not a separate clone elsewhere) — Gareth edits the live config and commits from the same working tree HA reads from.

## What's tracked

`.gitignore` is deliberately narrow — only `*.yaml`, `*.jinja`, and a specific set of folders (`packages/` incl. `household`, `appliance_power_monitoring`, `energy_management` + its `appliance_power_monitoring` subfolder, `heating`, `car`; also `custom_templates`, `ui-panel`) are tracked. Everything else is excluded by default, notably:

- `secrets.yaml`, `known_devices.yaml`, `device_tracker.yaml`, `config_ic3.yaml` — **never commit these**, they're excluded on purpose.
- `.storage/`, HACS `custom_components/`, `www/`, `themes/`, `blueprints/`, images, TTS cache, `ssh_keys/` — all untracked runtime/generated state, not meant to be in git.

If you add a new file that should be tracked, either put it somewhere already covered by the patterns above, or add an explicit `!filename` exception to `.gitignore`.

## Drift check

As of 2026-07-15 the repo was reconciled with live config after ~2.3 years of drift (last real commit before that was 2024-03-02). Worth periodically diffing live vs. `git ls-files` content when working in this repo — UI-driven changes (automations, scenes, scripts created via the HA frontend) accumulate locally and never make it into git unless someone explicitly commits them.
