# freeipa_client

This role is installing and configuring the FreeIPA Client according to
your needs.

In combination with
[`freeipa`](https://galaxy.ansible.com/timorunge/freeipa)
([Github](https://github.com/timorunge/ansible-freeipa)) it's
possible (and tested) to use `freeipa_client` with the latest version of
FreeIPA itself on Debian 9.4 and Ubuntu >= 18.04 (take a look at
the [example section](https://github.com/timorunge/ansible-freeipa#5-install-freeipa-with-timorungesssd-and-timorungefreeipa_client)).

## Requirements

This role requires
[Ansible 2.5.0](https://docs.ansible.com/ansible/devel/roadmap/ROADMAP_2_5.html)
or higher.

You can simply use pip to install (and define) a stable version:

```sh
pip install ansible==2.7.5
```

All platform requirements are listed in the metadata file.

In order to setup this role correctly it's required that you can communicate on
the
[required ports](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/linux_domain_identity_authentication_and_policy_guide/installing-ipa#prereq-ports-list)
to an [FreeIPA Server](https://galaxy.ansible.com/timorunge/freeipa-server)
([Github Repo](https://github.com/timorunge/ansible-freeipa-server)).

## Install

```sh
ansible-galaxy install timorunge.freeipa_client
```

## Role Variables

It is required to set the following variables in order to get this role up and
running (without customisation). Those variables don't have any default values:

```yaml
# Primary DNS domain of the IPA deployment
# Type: Str
freeipa_client_domain: example.com
# The hostname of this machine (FQDN)
# Type: Str
freeipa_client_fqdn: srv-1-eu-central-1.example.com
# Password to join the IPA realm
# Type: Str
freeipa_client_password: Passw0rd
# Principal to use to join the IPA realm
# Type: Str
freeipa_client_principal: admin
# Kerberos realm name of the IPA deployment
# Type: Str
freeipa_client_realm: EXAMPLE.COM
# FQDN of IPA server
# Type: Str
freeipa_client_server: ipa.example.com
```

The variables that can be passed to this role and a brief description about
them are as follows. (For all variables, take a look at [defaults/main.yml](defaults/main.yml))

```yaml
# The base command for the FreeIPA installation
# Type: Str
freeipa_client_install_base_command: ipa-client-install --unattended

# The default FreeIPA installation options
# Type: List
freeipa_client_install_options:
  - "--domain={{ freeipa_client_domain }}"
  - "--server={{ freeipa_client_server }}"
  - "--realm={{ freeipa_client_realm }}"
  - "--principal={{ freeipa_client_principal }}"
  - "--password={{ freeipa_client_password }}"
  - "--mkhomedir"
  - "--hostname={{ freeipa_client_fqdn }}"
  - "--force-join"
```

## Examples

To keep the document lean the install options are stripped.
You can find the install options either in [this
document](#freeipa-client-install-options) or in the online
[man page for ipa-client-install](https://linux.die.net/man/1/ipa-client-install).

### 1) Install the FreeIPA client with default settings

```yaml
- hosts: freeipa-clients
  vars:
    freeipa_client_domain: example.com
    freeipa_client_server: ipa.example.com
    freeipa_client_realm: EXAMPLE.COM
    freeipa_client_principal: admin
    freeipa_client_password: Passw0rd
    freeipa_client_fqdn: srv-1-eu-central-1.example.com
  roles:
    - timorunge.freeipa_client
```

### 2) Install the FreeIPA server with custom install options

```yaml
- hosts: freeipa-clients
  vars:
    freeipa_client_domain: example.com
    freeipa_client_server: ipa.example.com
    freeipa_client_realm: EXAMPLE.COM
    freeipa_client_principal: admin
    freeipa_client_password: Passw0rd
    freeipa_client_fqdn: srv-1-eu-central-1.example.com
    freeipa_client_install_options:
      - "--no-ntp"
      - "--ssh-trust-dns"
      - "--ip-address=172.20.1.2"
      - "--ip-address=172.20.2.2"
  roles:
    - timorunge.freeipa_client
```

### 3) Install the FreeIPA client and add multiple IPA servers

```yaml
- hosts: freeipa-clients
  vars:
    freeipa_client_domain: example.com
    freeipa_client_server:
      - ipa-eu-central-1.example.com
      - ipa-eu-west-1.example.com
      - ipa-eu-west-2.example.com
      - ipa-eu-west-3.example.com
      - ipa.example.com
    freeipa_client_realm: EXAMPLE.COM
    freeipa_client_principal: admin
    freeipa_client_password: Passw0rd
    freeipa_client_fqdn: srv-1-eu-central-1.example.com
    freeipa_client_install_options:
      - "--server={{ freeipa_client_server | join(' --server=') }}"
  roles:
    - timorunge.freeipa_client
```

## FreeIPA client install options

An overview of the install options for ipa-client-install (4.6.4).

```sh
Usage: ipa-client-install [options]

Options:
  --version             show program's version number and exit
  -h, --help            show this help message and exit
  -U, --unattended      unattended (un)installation never prompts the user
  --uninstall           uninstall an existing installation. The uninstall can
                        be run with --unattended option

  Basic options:
    -p PRINCIPAL, --principal=PRINCIPAL
                        principal to use to join the IPA realm
    --ca-cert-file=FILE
                        load the CA certificate from this file
    --ip-address=IP_ADDRESS
                        Specify IP address that should be added to DNS. This
                        option can be used multiple times
    --all-ip-addresses  All routable IP addresses configured on any interface
                        will be added to DNS
    --domain=DOMAIN_NAME
                        primary DNS domain of the IPA deployment (not
                        necessarily related to the current hostname)
    --server=SERVER     FQDN of IPA server
    --realm=REALM_NAME  Kerberos realm name of the IPA deployment (typically
                        an upper-cased name of the primary DNS domain)
    --hostname=HOST_NAME
                        The hostname of this machine (FQDN). If specified, the
                        hostname will be set and the system configuration will
                        be updated to persist over reboot. By default the
                        result of getfqdn() call from Python's socket module
                        is used.

  Client options:
    -w PASSWORD, --password=PASSWORD
                        password to join the IPA realm (assumes bulk password
                        unless principal is also set)
    -W                  Prompt for a password to join the IPA realm
    --noac              do not modify the nsswitch.conf and PAM configuration
    -f, --force         force setting of LDAP/Kerberos conf
    --configure-firefox
                        configure Firefox to use IPA domain credentials
    --firefox-dir=FIREFOX_DIR
                        specify directory where Firefox is installed (for
                        example: '/usr/lib/firefox')
    -k KEYTAB, --keytab=KEYTAB
                        path to backed up keytab from previous enrollment
    --mkhomedir         create home directories for users on their first login
    --force-join        Force client enrollment even if already enrolled
    --ntp-server=NTP_SERVER
                        ntp server to use. This option can be used multiple
                        times
    -N, --no-ntp        do not configure ntp
    --force-ntpd        Stop and disable any time&date synchronization
                        services besides ntpd
    --nisdomain=NISDOMAIN
                        NIS domain name
    --no-nisdomain      do not configure NIS domain name
    --ssh-trust-dns     configure OpenSSH client to trust DNS SSHFP records
    --no-ssh            do not configure OpenSSH client
    --no-sshd           do not configure OpenSSH server
    --no-sudo           do not configure SSSD as data source for sudo
    --no-dns-sshfp      do not automatically create DNS SSHFP records
    --kinit-attempts=KINIT_ATTEMPTS
                        number of attempts to obtain host TGT (defaults to 5).
    --request-cert      request certificate for the machine

  SSSD options:
    --fixed-primary     Configure sssd to use fixed server as primary IPA
                        server
    --permit            disable access rules by default, permit all access.
    --enable-dns-updates
                        Configures the machine to attempt dns updates when the
                        ip address changes.
    --no-krb5-offline-passwords
                        Configure SSSD not to store user password when the
                        server is offline
    --preserve-sssd     Preserve old SSSD configuration if possible

  Automount options:
    --automount-location=AUTOMOUNT_LOCATION
                        Automount location

  Logging and output options:
    -v, --verbose       print debugging information
    -d, --debug         alias for --verbose (deprecated)
    -q, --quiet         output only errors
    --log-file=FILE     log to the given file
```

## Testing

[![Build Status](https://travis-ci.org/timorunge/ansible-freeipa-client.svg?branch=master)](https://travis-ci.org/timorunge/ansible-freeipa-client)

Travis tests are done with [Docker](https://www.docker.com) and
[docker_test_runner](https://github.com/timorunge/docker-test-runner). Tests
on Travis are performing linting and syntax checks.

For further details and additional checks take a look at the
[docker_test_runner configuration](tests/docker_test_runner.yml) and the
[Docker entrypoint](tests/docker/docker-entrypoint.sh).

```sh
# Testing locally:
curl https://raw.githubusercontent.com/timorunge/docker-test-runner/master/install.sh | sh
./docker_test_runner.py -f tests/docker_test_runner.yml
```

## Dependencies

This role requires an up and running
[FreeIPA Server](https://galaxy.ansible.com/timorunge/freeipa_server)
([Github Repo](https://github.com/timorunge/ansible-freeipa-server)).

## License

[BSD 3-Clause "New" or "Revised" License](https://spdx.org/licenses/BSD-3-Clause.html)

## Author Information

- Timo Runge
