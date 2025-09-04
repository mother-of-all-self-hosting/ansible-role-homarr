<!--
SPDX-FileCopyrightText: 2020 - 2024 MDAD project contributors
SPDX-FileCopyrightText: 2020 - 2024 Slavi Pantaleev
SPDX-FileCopyrightText: 2020 Aaron Raimist
SPDX-FileCopyrightText: 2020 Chris van Dijk
SPDX-FileCopyrightText: 2020 Dominik Zajac
SPDX-FileCopyrightText: 2020 Mickaël Cornière
SPDX-FileCopyrightText: 2022 François Darveau
SPDX-FileCopyrightText: 2022 Julian Foad
SPDX-FileCopyrightText: 2022 Warren Bailey
SPDX-FileCopyrightText: 2023 Antonis Christofides
SPDX-FileCopyrightText: 2023 Felix Stupp
SPDX-FileCopyrightText: 2023 Pierre 'McFly' Marty
SPDX-FileCopyrightText: 2024 - 2025 Suguru Hirahara

SPDX-License-Identifier: AGPL-3.0-or-later
-->

# Setting up Homarr

This is an [Ansible](https://www.ansible.com/) role which installs [Homarr](https://homarr.dev) to run as a [Docker](https://www.docker.com/) container wrapped in a systemd service.

Homarr is a highly customizable dashboard for managing your favorite applications and services with a drag-and-drop grid system. It also integrates with various self-hosted applications.

See the project's [documentation](https://homarr.dev/docs/getting-started) to learn what Homarr does and why it might be useful to you.

## Adjusting the playbook configuration

To enable Homarr with this role, add the following configuration to your `vars.yml` file.

**Note**: the path should be something like `inventory/host_vars/mash.example.com/vars.yml` if you use the [MASH Ansible playbook](https://github.com/mother-of-all-self-hosting/mash-playbook).

```yaml
########################################################################
#                                                                      #
# homarr                                                               #
#                                                                      #
########################################################################

homarr_enabled: true

########################################################################
#                                                                      #
# /homarr                                                              #
#                                                                      #
########################################################################
```

### Set the hostname

To enable Homarr you need to set the hostname as well. To do so, add the following configuration to your `vars.yml` file. Make sure to replace `example.com` with your own value.

```yaml
homarr_hostname: "example.com"
```

After adjusting the hostname, make sure to adjust your DNS records to point the domain to your server.

**Note**: hosting Homarr under a subpath (by configuring the `homarr_path_prefix` variable) does not seem to be possible due to Homarr's technical limitations.

### Set 32-byte hex digits for secret key

You also need to specify **32-byte hex digits** to encrypt integration secrets on the database. To do so, add the following configuration to your `vars.yml` file. The value can be generated with `openssl rand -hex 32` or in another way.

```yaml
homarr_environment_variables_secret_encryption_key: YOUR_SECRET_KEY_HERE
```

>[!NOTE]
> Other type of values such as one generated with `pwgen -s 64 1` does not work.

### Use a database compatible with MySQL2 (optional)

By default the service adopts `better-sqlite3` as a database driver, which stores a SQLite database in `homarr_data_path`.

Alternatively, it is possible to use a database compatible with [MySQL2](https://sidorares.github.io/node-mysql2/docs), a MySQL client for Node.js. To use it, add the following configuration to your `vars.yml` file.

```yaml
homarr_database_type: mysql2
```

>[!NOTE]
> Currently (as of v1.17.0) MariaDB is not supported but planned. See [this issue at GitHub](https://github.com/homarr-labs/homarr/issues/2305) for the latest information.

### Extending the configuration

There are some additional things you may wish to configure about the component.

Take a look at:

- [`defaults/main.yml`](../defaults/main.yml) for some variables that you can customize via your `vars.yml` file. You can override settings (even those that don't have dedicated playbook variables) using the `homarr_environment_variables_additional_variables` variable

See [the official documentation](https://homarr.dev/docs/advanced/environment-variables/) for a complete list of Homarr's config options that you could put in `homarr_environment_variables_additional_variables`.

## Installing

After configuring the playbook, run the installation command of your playbook as below:

```sh
ansible-playbook -i inventory/hosts setup.yml --tags=setup-all,start
```

If you use the MASH playbook, the shortcut commands with the [`just` program](https://github.com/mother-of-all-self-hosting/mash-playbook/blob/main/docs/just.md) are also available: `just install-all` or `just setup-all`

## Usage

After running the command for installation, Homarr becomes available at the specified hostname like `https://example.com`.

You can open the page with a web browser to start the onboarding process. See [this official guide](https://homarr.dev/docs/getting-started/after-the-installation/) for details.

## Troubleshooting

### Check the service's logs

You can find the logs in [systemd-journald](https://www.freedesktop.org/software/systemd/man/systemd-journald.service.html) by logging in to the server with SSH and running `journalctl -fu homarr` (or how you/your playbook named the service, e.g. `mash-homarr`).
