# goose-automation

Ansible playbook that installs **goose-redhat** — Red Hat's Lightspeed-backed
goose agent — on RHEL 9.8+/10.2+ and makes it work for root and every
interactive user, no re-login needed.

## Usage

1. List your RHEL hosts in `inventories/production/hosts.yml`.
2. Enable the extensions repo on each target (one-time, outside this playbook):

       sudo subscription-manager repos --enable=rhel-9-for-$(arch)-extensions-rpms    # RHEL 9
       sudo subscription-manager repos --enable=rhel-10-for-$(arch)-extensions-rpms   # RHEL 10

3. Run the playbook:

       ansible-playbook -i inventories/production/hosts.yml site.yml

Requirements: ansible-core 2.14+ (builtin modules only) and sudo access on the
targets.

## What it does

Installs `goose` and `goose-redhat`, enables the socket-activated `goose-proxy`
unit, and seeds the `rhel_cla` provider config for root plus every interactive
user by running the package's own `/etc/profile.d` script as each user.
Idempotent — re-runs only fill gaps, and users with their own goose config are
left untouched.

## Tuning

Defaults live in `roles/goose_redhat/defaults/main.yml`; override them in
`inventories/production/group_vars/all.yml` — e.g. `goose_redhat_seed_users`
to limit seeding to specific users. Full variable reference:
`roles/goose_redhat/README.md`.
