# goose-automation

Ansible playbooks that install **goose-redhat** (Red Hat's Lightspeed-backed
goose agent) and the **MCP server for RHEL** on RHEL 9.8+/10.2+, making both
work for root and every interactive user, no re-login needed.

## Playbooks

| Playbook | What it does |
| --- | --- |
| `site.yml` | Installs `goose` + `goose-redhat`, enables the `goose-proxy` socket, seeds the `rhel_cla` provider config per user. |
| `mcp_server.yml` | Installs `linux-mcp-server` via uv per user and wires the `linux-tools` extension into each user's goose config. Run after `site.yml`. |

## Usage

1. List your RHEL hosts in `inventories/production/hosts.yml`.
2. Enable the extensions repo on each target (one-time):

       sudo subscription-manager repos --enable=rhel-9-for-$(arch)-extensions-rpms    # RHEL 9
       sudo subscription-manager repos --enable=rhel-10-for-$(arch)-extensions-rpms   # RHEL 10

3. Run both playbooks in order:

       ansible-playbook -i inventories/production/hosts.yml site.yml
       ansible-playbook -i inventories/production/hosts.yml mcp_server.yml

Requirements: ansible-core 2.14+ (builtin modules only) and sudo access on the
targets. Both are idempotent — re-runs only fill gaps, and users with their own
goose config are left untouched.

## Tuning

Defaults live in `roles/*/defaults/main.yml`; override them in
`inventories/production/group_vars/all.yml` — e.g. `goose_redhat_seed_users` /
`linux_mcp_server_seed_users` to limit which users are configured. Full variable
references: `roles/goose_redhat/README.md`, `roles/linux_mcp_server/README.md`.
