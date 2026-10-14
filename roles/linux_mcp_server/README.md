# linux_mcp_server

Installs the MCP server for RHEL (`linux-mcp-server`, developer preview) and
wires it into goose for **root and every interactive user** on the host.

## What it does

1. Asserts the target runs RHEL 9 or 10 on x86_64 or aarch64.
2. Installs `python3` and `python3-pip`, then `uv` system-wide (the path
   documented for the MCP server for RHEL). If pip drops the binary somewhere
   only root can reach, it is copied to `/usr/local/bin` so every user can
   run it.
3. Runs `uv tool install linux-mcp-server` as each target user, giving each
   their own `~/.local/bin/linux-mcp-server`. uv manages its own Python, so
   RHEL 9's system Python is fine.
4. Adds the `linux-tools` extension to each user's `~/.config/goose/config.yaml`
   (created by the goose_redhat role) pointing at their own MCP server binary.
   Configs with an unmanaged `extensions:` section are left untouched and
   reported for manual merge.
5. Verifies the server runs for every target user and reports per-user results.

Users already set up are skipped; re-runs only fill gaps.

## Requirements

- RHEL 9.8+ or 10.2+ with an active subscription, plus outbound internet
  access (uv downloads Python and the server from PyPI).
- Run the `goose_redhat` role first — the extension wiring needs
  `~/.config/goose/config.yaml`.
- ansible-core 2.14+ (builtin modules only) and `become` (sudo) to root.

## Role variables

| Variable | Default | Description |
| --- | --- | --- |
| `linux_mcp_server_packages` | `[python3, python3-pip]` | Packages installed to bootstrap uv. |
| `linux_mcp_server_package` | `linux-mcp-server` | PyPI package installed per user via `uv tool install`. |
| `linux_mcp_server_seed_root` | `true` | Also set up root. |
| `linux_mcp_server_min_uid` | `1000` | Minimum UID treated as an interactive user. |
| `linux_mcp_server_allowed_shells` | `[/bin/bash, /bin/sh, /bin/zsh]` | Login shells that qualify a user. |
| `linux_mcp_server_seed_users` | `[]` | Explicit user list; overrides discovery when non-empty. |
| `linux_mcp_server_goose_extension` | `linux-tools` | Extension ID/name written to the goose config. |
| `linux_mcp_server_goose_description` | `Linux system diagnostics` | Extension description. |
| `linux_mcp_server_goose_timeout` | `300` | Goose extension timeout in seconds. |
| `linux_mcp_server_allowed_log_paths` | `""` | Optional `LINUX_MCP_ALLOWED_LOG_PATHS` allowlist (comma-separated log paths) for `read_log_file`. |

## Notes

- On Linux installs the server defaults to host mode `any`: tools can run on
  the local system (no SSH) or on remote hosts over SSH using each user's own
  SSH setup. See the
  [configuration reference](https://rhel-lightspeed.github.io/linux-mcp-server/config-reference/)
  for details.
- Remote SSH use needs the user's SSH keys/config; see the
  [SSH guide](https://rhel-lightspeed.github.io/linux-mcp-server/ssh/).

## Example

```yaml
- hosts: workstations
  become: true
  roles:
    - role: linux_mcp_server
      vars:
        linux_mcp_server_seed_users: [alice, bob]
        linux_mcp_server_allowed_log_paths: "/var/log/messages,/var/log/secure"
```
