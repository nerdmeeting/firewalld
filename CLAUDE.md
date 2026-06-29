# nerdmeeting.firewalld

An Ansible role that configures `firewalld` on Enterprise Linux (CentOS/RHEL 7) and
optionally opens inbound `80/tcp` and/or `8080/tcp`. When either port is opened it also
enables an SELinux boolean permitting outbound web connections, so a co-located web app
can reach the network. The role is published to Ansible Galaxy as `nerdmeeting.firewalld`.

## Architecture

Standard Ansible role layout:

- `tasks/main.yml` — the role's logic: enable the SELinux boolean when either port is
  requested, then open `80/tcp` and/or `8080/tcp` via `ansible.posix.firewalld`
  (`permanent: true`, `immediate: true`). Guarded by the `allow_remote_port_*` vars.
- `defaults/main.yml` — default values for the role variables (both ports `false`).
- `vars/main.yml` — intentionally empty (no internal vars override the defaults).
- `handlers/main.yml` — no handlers; firewalld tasks apply immediately (`immediate: true`).
- `templates/`, `files/` — placeholders; the role ships no templates or static files yet.
- `meta/main.yml` — Galaxy metadata (role name, platforms, tags) and the
  `ansible.posix` collection dependency.
- `molecule/default/` — Molecule (Docker) scenario: `molecule.yml` (CentOS 7 container),
  `converge.yml` (applies the role with both ports enabled), `verify.yml` (asserts the
  ports are open).
- `tests/` — `inventory` + `test.yml` for a quick local/syntax run.
- `.github/workflows/` — CI: lint/syntax checks, optionally a `molecule` job.

## Role variables

```yaml
allow_remote_port_80: false    # open inbound 80/tcp
allow_remote_port_8080: false  # open inbound 8080/tcp
```

## Local dev / testing

```bash
# Quick syntax check (no containers)
ansible-playbook -i tests/inventory tests/test.yml --syntax-check

# Lint
ansible-lint

# Full Molecule (Docker) end-to-end test — spins up CentOS 7, applies the role, verifies
molecule test
```

> Molecule uses the Docker driver. On macOS/Windows, Docker Desktop must be installed and
> running before `molecule test`.

There are no application env vars or secrets in this role; behavior is driven entirely by
the `allow_remote_port_*` variables passed in by the consuming playbook.

## Project constants

- `<JIRA_PREFIX>` — `NMFIREWALLD`
- `<HOST>` — GitHub
- `<PR_CLI>` — `gh`
- `<PR_TERM>` — PR
- `<TARGET_BRANCH>` — `develop`

@~/projects/nerdmeeting/nerdmeeting-rules/RULES.md

## PRs — host commands

Host is GitHub (`origin` on github.com); the CLI is `gh`. The PR title must be the literal
branch name (`feature/NMFIREWALLD-<parent>/<slug>` or `bugfix/NMFIREWALLD-<key>/<slug>`),
and the base branch is `develop`.

```bash
# Open the PR (title = literal branch name, base = develop)
gh pr create --base develop \
  --title "feature/NMFIREWALLD-<parent>/<slug>" \
  --body "$(cat <<'EOF'
## Summary
- One bullet per logical change in the PR.

## Test plan

### Setup
- [ ] `git checkout feature/NMFIREWALLD-<parent>/<slug>`
- [ ] `pip install "ansible>=2.9,<10.0" molecule molecule-plugins[docker] ansible-lint`
- [ ] Ensure Docker Desktop is running (macOS/Windows).

### Role behavior
- [ ] `molecule test` → CentOS 7 container converges and `verify.yml` passes
      (asserts `80/tcp` and `8080/tcp` are open).
- [ ] Set both `allow_remote_port_*` to `false` → neither port is opened, SELinux
      boolean is left untouched.

### Tests
- [ ] `ansible-playbook -i tests/inventory tests/test.yml --syntax-check` → no syntax errors.
- [ ] `ansible-lint` → clean (0 violations).
- [ ] `molecule test` → all assertions pass, container destroyed.
EOF
)"

# Update the PR body later (e.g. to cover a follow-up commit — add a new section, don't rewrite)
gh pr edit <number> --body "$(cat <<'EOF'
... updated body ...
EOF
)"

# Inspect the PR
gh pr view <number>
```

Note: `develop` is currently local-only. Publish it to origin once (`git push origin develop`)
so `gh pr create --base develop` can target it. After approval, merge with
`gh pr merge <number> --merge` (never squash — preserve the per-sub-task commits).
