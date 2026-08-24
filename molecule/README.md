<!--
SPDX-FileCopyrightText: 2018-2025 Slavi Pantaleev
SPDX-FileCopyrightText: 2019-2022 Aaron Raimist
SPDX-FileCopyrightText: 2019-2023 MDAD project contributors
SPDX-FileCopyrightText: 2023 QEDeD
SPDX-FileCopyrightText: 2024 Fabio Bonelli
SPDX-FileCopyrightText: 2024 Nikita Chernyi
SPDX-FileCopyrightText: 2024-2026 Suguru Hirahara
SPDX-FileCopyrightText: 2026 spatterlight

SPDX-License-Identifier: AGPL-3.0-or-later
-->

# Molecule Testing

This role supports [Molecule](https://docs.ansible.com/projects/molecule/), an Ansible testing framework designed for developing and testing Ansible collections, playbooks, and roles.

## Prerequisites

To utilize Molecule you need to prepare several requirements:

- **x86** computer running one of these operating systems that make use of [systemd](https://systemd.io/):
  - **Archlinux**
  - **CentOS**, **Rocky Linux**, **AlmaLinux**, or possibly other RHEL alternatives (although your mileage may vary)
  - **Debian** (10/Buster or newer)
  - **Ubuntu** (18.04 or newer, although [20.04 may be problematic](https://github.com/mother-of-all-self-hosting/mash-playbook/blob/main/docs/ansible.md#supported-ansible-versions) if you run the Ansible playbook on it)
- `root` access on the computer which Molecule runs against
- [Ansible](http://ansible.com/) program
- [Python](https://www.python.org/)
  - Most distributions install Python by default, but some don't (e.g. Ubuntu 18.04) and require manual installation (something like `apt-get install python3`)
- [Docker](https://www.docker.com)
  - Access to Docker UNIX socket (`/var/run/docker.sock`) is required by default

## Installation

To set up the environment for using Molecule, run the command below on the terminal:

```bash
python3 -m venv ./molecule/venv
source ./molecule/venv/bin/activate
pip3 install -r ./molecule/requirements.txt
```

## Scenarios

Currently these testing scenarios are available:

### `default`

Tests a standard Homarr installation, using the container image published upstream.

### `default-selfbuild`

The same, with the container image built from Homarr's source instead of pulled.

Both scenarios run the same checks, which live in [`verify_tasks.yml`](verify_tasks.yml).

## What the scenarios verify

Homarr is a Next.js application, and it renders a page for very nearly any path you ask it for. A test which fetched a URL and found a `200` would pass against a Homarr whose database is unreachable, against one somebody else has already set up, and against a fair few things that are not Homarr. So the suite establishes what a fresh, un-onboarded Homarr serves *before* it touches anything, and then makes every one of those facts flip:

| | before | after |
| --- | --- | --- |
| `onboard.currentStep` | `start` | `finish` |
| `/`, `/manage`, `/boards/dashboard` | `307` to `/init` | `200` |
| `user.getAll`, `info.getInfo` without a session | `401` | `200` (with a session) |
| the private board the suite creates | does not exist | visible to its creator, invisible anonymously |
| the `onboarding` row in SQLite | `start` | `finish` |

On top of that it waits on `/api/health/live`, which is the deep health check — it answers `500` unless Homarr can reach both its database and its Redis — asserts that the version the running instance reports matches the one `defaults/main.yml` pins, and cross-checks the whole story against the SQLite database the role bind-mounted into the container.

Two assertions are deliberately made to fail on purpose, because they would otherwise prove nothing: a login with the wrong password (Auth.js answers a rejected login with the same `302` it answers an accepted one with, so only the absence of a session cookie distinguishes them), and the version comparison against a version that has never been released.

The onboarding wizard is completed from `verify.yml`, never from `converge.yml`. Walking a user through a setup wizard is not the role's job, and a converge which did it would leave the negative control with nothing to establish.

## Running

By default it is configured to run the scenarios on Ubuntu 26.04.

```bash
molecule test --scenario-name default
```

You can utilize other distributions by setting one to the `MOLECULE_DISTRO` environment variable:

```bash
# Ubuntu 24.04
MOLECULE_DISTRO=ubuntu2404 molecule test --scenario-name default

# Debian 13
MOLECULE_DISTRO=debian13 molecule test --scenario-name default

# Debian 12
MOLECULE_DISTRO=debian12 molecule test --scenario-name default
```
