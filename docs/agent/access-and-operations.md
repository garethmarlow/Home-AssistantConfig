# Access and Operations

Read this file for filesystem access, git, validation, reload, restart or drift work.

## Access topology

- **Live config on disk:** `/config` inside the `homeassistant` container. The same files are reachable from Gareth's Mac at `/Volumes/docker/homeassistant/config` through an SMB mount. This is a live bind mount, not a copy.
- **File-level read/diff:** read and edit the mounted folder directly when it is explicitly attached to the session. No network access is needed.
- **Shell, git and Docker work:** Gareth runs exact commands through **Synology DSM → Container Manager → select `homeassistant` → Open terminal** and returns the output.
- There is no HA add-on terminal and no Tailscale route. Do not try alternate LAN access or browser automation.

## Git and authentication

- Remote: `git@github.com:garethmarlow/Home-AssistantConfig.git`.
- Git is initialized directly inside live `/config`; this is not a separate deployment clone.
- Authentication uses the write-enabled deploy key at `/config/ssh_keys/id_rsa_homeassistant`.
- The preferred one-time container setting is:

  ```sh
  git config core.sshCommand "ssh -i /config/ssh_keys/id_rsa_homeassistant"
  ```

- Without that setting, prefix a git command with:

  ```sh
  GIT_SSH_COMMAND="ssh -i /config/ssh_keys/id_rsa_homeassistant" git <command>
  ```

Never read, print or commit the private key.

## Tracking boundary

`.gitignore` is deliberately allowlist-shaped. YAML, Jinja and selected configuration folders are tracked; runtime and secret material is excluded.

Never commit:

- `secrets.yaml`, `known_devices.yaml`, `device_tracker.yaml` or `config_ic3.yaml`;
- `.storage/` except the already explicit dashboard exceptions;
- HACS `custom_components/`, `www/`, images, TTS cache, logs, databases or SSH keys;
- generated `lifeos_bridge/*.json` transport files.

When adding a stable source file, add the narrowest possible `.gitignore` exception.

## Safe change sequence

1. Read the root kernel and relevant routed leaf.
2. Inspect only the files involved and preserve unrelated dirty changes.
3. Edit the smallest surface.
4. Review a focused diff.
5. Validate YAML and Home Assistant configuration in proportion to the change.
6. Explain any reload or restart required and obtain Gareth's approval.
7. Gareth runs container-only commands through Container Manager.
8. After validation, commit only the intended source files.

Documentation-only changes do not require an HA reload.

## Drift

The live repo was reconciled on 2026-07-15 after roughly 2.3 years of drift; the previous substantive commit was 2024-03-02. UI-driven automation, scene and script edits can accumulate locally without reaching git. Periodically compare the live working tree with `git ls-files`, but never treat all untracked runtime state as source.
