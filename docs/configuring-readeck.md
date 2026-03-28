<!--
SPDX-FileCopyrightText: 2020 Aaron Raimist
SPDX-FileCopyrightText: 2020 Chris van Dijk
SPDX-FileCopyrightText: 2020 Dominik Zajac
SPDX-FileCopyrightText: 2020 Mickaël Cornière
SPDX-FileCopyrightText: 2020-2024 MDAD project contributors
SPDX-FileCopyrightText: 2020-2024 Slavi Pantaleev
SPDX-FileCopyrightText: 2022 François Darveau
SPDX-FileCopyrightText: 2022 Julian Foad
SPDX-FileCopyrightText: 2022 Warren Bailey
SPDX-FileCopyrightText: 2023 Antonis Christofides
SPDX-FileCopyrightText: 2023 Felix Stupp
SPDX-FileCopyrightText: 2023 Julian-Samuel Gebühr
SPDX-FileCopyrightText: 2023 Pierre 'McFly' Marty
SPDX-FileCopyrightText: 2024 Thomas Miceli
SPDX-FileCopyrightText: 2024 noah
SPDX-FileCopyrightText: 2024, 2025 MASH project contributors
SPDX-FileCopyrightText: 2024-2026 Suguru Hirahara

SPDX-License-Identifier: AGPL-3.0-or-later
-->

# Setting up Readeck

This is an [Ansible](https://www.ansible.com/) role which installs [Readeck](https://readeck.org) to run as a [Docker](https://www.docker.com/) container wrapped in a systemd service.

Readeck is a simple web application that lets you save the precious readable content of web pages you like and want to keep forever.

See the project's [documentation](https://readeck.org/en/docs/) to learn what Readeck does and why it might be useful to you.

## Prerequisites

To run an Readeck it is necessary to prepare a database. You can use [Postgres](https://www.postgresql.org/) or [SQLite](https://www.sqlite.org/). The SQLite database file will be automatically created by the service if it is enabled.

If you are looking for an Ansible role for Postgres, you can check out [this role (ansible-role-postgres)](https://github.com/mother-of-all-self-hosting/ansible-role-postgres) maintained by the [Mother-of-All-Self-Hosting (MASH)](https://github.com/mother-of-all-self-hosting) team.

## Adjusting the playbook configuration

To enable Readeck with this role, add the following configuration to your `vars.yml` file.

**Note**: the path should be something like `inventory/host_vars/mash.example.com/vars.yml` if you use the [MASH Ansible playbook](https://github.com/mother-of-all-self-hosting/mash-playbook).

```yaml
########################################################################
#                                                                      #
# readeck                                                              #
#                                                                      #
########################################################################

readeck_enabled: true

########################################################################
#                                                                      #
# /readeck                                                             #
#                                                                      #
########################################################################
```

### Set the hostname

To enable Readeck you need to set the hostname as well. To do so, add the following configuration to your `vars.yml` file. Make sure to replace `example.com` with your own value.

```yaml
readeck_hostname: "example.com"
```

After adjusting the hostname, make sure to adjust your DNS records to point the domain to your server.

**Note**: hosting Readeck under a subpath (by configuring the `readeck_path_prefix` variable) does not seem to be possible due to Readeck's technical limitations.

### Specify database

It is necessary to select database used by Readeck from Postgres and SQLite.

To use Postgres, add the following configuration to your `vars.yml` file:

```yaml
readeck_database_type: postgres
```

Set `sqlite` to use SQLite. The SQLite database is stored in the directory specified with `readeck_data_path`.

For other settings, check variables such as `readeck_database_*` on [`defaults/main.yml`](../defaults/main.yml).

### Configuring connection to Postgres server (optional)

By default the role is configured to establish connection with the Postgres server via the Unix socket. You can mount the Unix socket by adding the following configuration to your `vars.yml` file:

```yaml
# Specify the path to the Postgres Unix socket path on the host (bind-mount source)
readeck_database_socket_path_host: ""
```

Setting it enables to connect to the Postgres server via Unix socket mounted in the container at `/run-postgres/.s.PGSQL.5432`.

If TCP connection is preferred, connection via the Unix socket can be disabled by adding the following configuration to your `vars.yml` file:

```yaml
# Disable the connection to Postgres server via a Unix socket
readeck_database_socket_enabled: false

readeck_database_hostname: YOUR_POSTGRES_SERVER_HOSTNAME_HERE
readeck_database_port: 5432
```

### Configuring a SMTP mailer (optional)

You can configure a SMTP mailer to enable email functions such as bookmark sharing and password recovery.

To configure it, add the following configuration to your `vars.yml` file as below (adapt to your needs):

```yaml
# Specify SMTP server hostname
readeck_environment_variables_readeck_mail_host: ""

# Specify SMTP server port
readeck_environment_variables_readeck_mail_port: 587

# Specify SMTP server username
readeck_environment_variables_readeck_mail_username: ""

# Specify SMTP server password
readeck_environment_variables_readeck_mail_password: ""

# Control if the server certificate is verified
readeck_environment_variables_readeck_mail_insecure: false

# Specify SMTP server encryption
# Valid values: starttls, ssltls
readeck_environment_variables_readeck_mail_encryption: ""

# Specify the email address that emails will be sent from
readeck_environment_variables_readeck_mail_from: ""

# Specify the email address that emails will be sent from
# For messages that do not need a reply
readeck_environment_variables_readeck_mail_fromnoreply: ""
```

>[!WARNING]
> Without setting an authentication method such as DKIM, SPF, and DMARC for your hostname, emails are most likely to be quarantined as spam at recipient's mail servers. The worst scenario is that your server's IP address or hostname will be included in the spam list such as the one managed by [Spamhaus](https://www.spamhaus.org/). If you have set up a mail server with the [MASH project's exim-relay Ansible role](https://github.com/mother-of-all-self-hosting/ansible-role-exim-relay), you can enable DKIM signing with it. Refer [its documentation](https://github.com/mother-of-all-self-hosting/ansible-role-exim-relay/blob/main/docs/configuring-exim-relay.md#enable-dkim-support-optional) for details.

### Extending the configuration

There are some additional things you may wish to configure about the service.

Take a look at:

- [`defaults/main.yml`](../defaults/main.yml) for some variables that you can customize via your `vars.yml` file. You can override settings (even those that don't have dedicated playbook variables) using the `readeck_environment_variables_additional_variables` variable

## Installing

After configuring the playbook, run the installation command of your playbook as below:

```sh
ansible-playbook -i inventory/hosts setup.yml --tags=setup-all,start
```

If you use the MASH playbook, the shortcut commands with the [`just` program](https://github.com/mother-of-all-self-hosting/mash-playbook/blob/main/docs/just.md) are also available: `just install-all` or `just setup-all`

## Usage

After running the command for installation, the Readeck instance becomes available at the URL specified with `readeck_hostname`. With the configuration above, the service is hosted at `https://example.com`.

To get started, open the URL with a web browser, and create a user.

## Troubleshooting

### Check the service's logs

You can find the logs in [systemd-journald](https://www.freedesktop.org/software/systemd/man/systemd-journald.service.html) by logging in to the server with SSH and running `journalctl -fu readeck` (or how you/your playbook named the service, e.g. `mash-readeck`).

#### Increase logging verbosity

If you want to increase the verbosity, add the following configuration to your `vars.yml` file:

```yaml
readeck_environment_variables_readeck_log_level: debug
```
