# goose_redhat

Installs Red Hat's `goose-redhat` build of the goose AI agent and makes it
usable by **root and every interactive user** on the host.

## What it does

1. Asserts the target runs RHEL 9 or 10 on x86_64 or aarch64.
2. Installs the `goose` and `goose-redhat` packages (the binary lands in
   `/usr/bin`, so it is system-wide and runnable by every user).
3. Enables the socket-activated `goose-proxy` unit when the package ships one.
4. Seeds the `rhel_cla` provider configuration for root and every interactive
   user by running the package's `/etc/profile.d` setup script as each user.
   Normally that configuration only appears when a user starts a fresh login
   shell, so this step makes `goose` work immediately — no logout/login
   required.
5. Verifies the installed packages and reports per-user results.

Users who already configured goose themselves (for example with their own API
key) are detected and left untouched.

## Requirements

- RHEL 9.8+ or 10.2+ with an active subscription.
- The extensions repository enabled on the target (managed outside this role):

      subscription-manager repos --enable=rhel-9-for-$(arch)-extensions-rpms   # RHEL 9
      subscription-manager repos --enable=rhel-10-for-$(arch)-extensions-rpms  # RHEL 10

- ansible-core 2.14+ (builtin modules only; no external collections needed).
- `become` (sudo) access to root on the target hosts.

## Role variables

| Variable | Default | Description |
| --- | --- | --- |
| `goose_redhat_packages` | `[goose, goose-redhat]` | Packages to install. |
| `goose_redhat_seed_root` | `true` | Also seed the provider config for root. |
| `goose_redhat_min_uid` | `1000` | Minimum UID treated as an interactive user. |
| `goose_redhat_allowed_shells` | `[/bin/bash, /bin/sh, /bin/zsh]` | Login shells that qualify a user for seeding. |
| `goose_redhat_seed_users` | `[]` | Explicit user list; overrides discovery when non-empty. |
| `goose_redhat_profile_script_pattern` | `*goose*` | Glob used to find the package's profile.d setup script. |

## Example

```yaml
- hosts: workstations
  become: true
  roles:
    - role: goose_redhat
      vars:
        goose_redhat_seed_users: [alice, bob]
```
