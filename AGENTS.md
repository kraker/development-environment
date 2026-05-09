# AGENTS.md

This file provides guidance to AI coding assistants when working with code in this repository. `CLAUDE.md` is a compatibility symlink for Claude Code.

## Agent Policy

When an AI assistant creates a commit, include this trailer:

```text
AI-assisted-by: OpenAI Codex
```

Do not amend already-pushed commits solely to add the trailer unless explicitly requested.

Do not commit credentials, subscription passwords, activation keys, tokens, or generated secret files.

Before committing, run the relevant lightweight checks:

- `uv run prek run --all-files` for broad repo checks when practical.
- `git diff --check`.

## What this is

Ansible automation for setting up a local (or remote) development environment on Enterprise Linux (Fedora 41+, RHEL 8+, Rocky, AlmaLinux). The default inventory points at `localhost ansible_connection=local`, so playbooks reconfigure the machine they run on.

## Common commands

Setup (one time):

```bash
uv sync                                                         # Python deps via uv
source .venv/bin/activate
uv run prek install --prepare-hooks                             # pre-commit hooks (prek = Rust prek)
ansible-galaxy collection install -r collections/requirements.yml
ansible-galaxy role install -r roles/requirements.yml
```

Run the full playbook:

```bash
source .venv/bin/activate
ansible-navigator run site.yml
```

Run a single sub-playbook (each top-level `*.yml` is independently runnable):

```bash
ansible-navigator run packages.yml
ansible-navigator run hashicorp.yml
ansible-navigator run lang.yml
ansible-navigator run quarto.yml
```

Lint (matches CI):

```bash
uv run prek run --all-files                # runs yamllint + ansible-lint + uv-lock + whitespace hooks
uv run ansible-lint                        # ansible-lint alone
```

## Architecture

`site.yml` is a thin orchestrator that `import_playbook`s each capability as a separate top-level YAML at the repo root. There are no `group_vars`/`host_vars` and no traditional `roles/` of our own — Galaxy roles install into `roles/` (gitignored).

Playbook order in `site.yml` is load-bearing:

1. `epel.yml` — enables CRB (CodeReady Builder) and installs EPEL. Branches by distro: `community.general.dnf_config_manager` for Rocky/Alma/CentOS, `community.general.rhsm_repository` for RHEL proper.
2. `packages.yml` — base packages (git, vim, podman, nodejs-npm).
3. `lang.yml` — common language build tools and R.
4. `quarto.yml` — installs Quarto from the official release metadata.
5. `prompt.yml` — starship via upstream installer.
6. `virtualization.yml` — `@Virtualization Host` dnf group, plus `libvirt-devel` and libvirt group membership (libvirt-devel requires CRB from step 1; needed by any tool linking against libvirt, e.g. vagrant-libvirt).
7. `hashicorp.yml` — adds the HashiCorp RPM repo, then installs Packer, Terraform, HCP CLI, and Vagrant from it. Second play installs `vagrant-libvirt` and `vagrant-registration` plugins as the invoking user (depends on `libvirt-devel` from step 6).

`virtualbox.yml` is **deliberately excluded** from `site.yml` (KVM/libvirt and VirtualBox conflict on the same host) and from `.ansible-lint` (`exclude_paths`). Most of its tasks are commented out — treat it as partial/draft.

## Conventions and gotchas

- **Execution environment is disabled** in `ansible-navigator.yml` (`enabled: false`). The comment explains why: the EE container can't target `localhost` the way these playbooks need to. Don't re-enable it without rethinking the inventory model.
- **`prek`, not `pre-commit`.** The repo uses [prek](https://github.com/j178/prek) (Rust reimplementation). CI invokes `j178/prek-action@v2`. Hooks are still defined in `.pre-commit-config.yaml` in the standard format.
- **Galaxy artifacts are gitignored** (`collections/ansible_collections/`, `roles/geerlingguy.*`, `.ansible/`). After cloning, you must run the `ansible-galaxy install` commands above before any playbook will work.
- **ansible-lint profile is `moderate`** (`.ansible-lint`). When adding new playbooks at the repo root, expect lint to apply unless explicitly excluded.
- **`localhost` + `become: true`** is the norm — playbooks assume passwordless sudo on the target.
