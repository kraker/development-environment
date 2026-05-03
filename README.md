# development-environment

Automate setting up local or remote development environments with Ansible.

## Install Prerequisites

This project requires Git, uv, and Podman.

* [Install Git](https://git-scm.com/install/)
* [Install uv](https://docs.astral.sh/uv/getting-started/installation/)
* [Install Podman](https://podman.io/docs/installation)

Podman is required to fetch the [community-ansible-dev-tools][1]
[execution envioronment][2] that commands like `ansible-navigator` need to
run.

[1]: https://docs.ansible.com/projects/dev-tools/container/
[2]: https://docs.ansible.com/projects/ansible/latest/getting_started_ee/introduction.html

## Project Setup

Clone project and change directories:

```bash
git clone https://github.com/kraker/development-environment.git
cd development-environment
```

Sync project:

```bash
uv sync
```

Setup pre-commit hooks:

```bash
source .venv/bin/activate
prek install --prepare-hooks
# Or
uv run prek install --prepare-hooks
```

Install Ansible collection and role dependencies:

```bash
ansible-galaxy collection install -r collections/requirements.yml
ansible-galaxy role install -r collections/requirements.yml
```

## Development Environment

Deploy a local (or remote) development environment:

```bash
source .venv/bin/activate       # Activate venv if you haven't already
ansible-navigator run site.yml
```

## LICENSE

MIT

## Author

Written by Alex Kraker ([@kraker](https://github.com/kraker))
