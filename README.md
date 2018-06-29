freeipa-client
==============

This role is building and installing an FreeIPA Client according to your needs.

Requirements
------------

This role requires Ansible 2.4.0 or higher. It's fully tested with the latest
stable release (2.5.5).

You can simply use pip to install (and define) the latest stable version:

```sh
pip install ansible==2.5.5
```

All platform requirements are listed in the metadata file.

In order to setup this role correctly it's required that you can communicate on
the
[required ports](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/linux_domain_identity_authentication_and_policy_guide/installing-ipa#prereq-ports-list)
to an [FreeIPA Server](https://galaxy.ansible.com/timorunge/freeipa-server/)
([Github Repo](https://github.com/timorunge/ansible-freeipa-server)).

Install
-------

```sh
ansible-galaxy install timorunge.freeipa-client
```

Role Variables
--------------

It is required to set the following variables in order to get this role up and
running (without customisation). Those variables don't have any default values:

```yaml
# Primary DNS domain of the IPA deployment
freeipa_client_domain: example.com
# The hostname of this machine (FQDN)
freeipa_client_fqdn: srv-1-eu-central-1.example.com
# Password to join the IPA realm
freeipa_client_password: Passw0rd
# Principal to use to join the IPA realm
freeipa_client_principal: admin
# Kerberos realm name of the IPA deployment
freeipa_client_realm: EXAMPLE.COM
# FQDN of IPA server
freeipa_client_server: ipa.example.com
```

The variables that can be passed to this role and a brief description about
them are as follows. (For all variables, take a look at [defaults/main.yml](defaults/main.yml))

```yaml
# The base command for the FreeIPA installation
freeipa_client_install_base_command: ipa-client-install --unattended

# The default FreeIPA installation options
freeipa_client_install_options:
  - "--domain={{ freeipa_client_domain }}"
  - "--server={{ freeipa_client_server }}"
  - "--realm={{ freeipa_client_realm }}"
  - "--principal={{ freeipa_client_principal }}"
  - "--password={{ freeipa_client_password }}"
  - '--mkhomedir'
  - "--hostname={{ freeipa_client_fqdn }}"
  - '--force-join'
```

Examples
--------

To keep the document lean the install options are stripped.
You can find the install options either in [this
document](#freeipa-client-install-options) or in the online
[man page for ipa-client-install](https://linux.die.net/man/1/ipa-client-install).

## 1) Install the FreeIPA client with default settings

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
    - timorunge.freeipa-client
```

## 2) Install the FreeIPA server with custom install options

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
      - '--no-ntp'
      - '--ssh-trust-dns'
      - '--ip-address=172.20.1.2'
      - '--ip-address=172.20.2.2'
  roles:
    - timorunge.freeipa-client
```

## 3) Install the FreeIPA client and add multiple IPA servers

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
    - timorunge.freeipa-client
```

FreeIPA client install options
------------------------------

An overview of the install options for ipa-client-install (4.3.1).

```sh
Options:
  --version             show program's version number and exit
  -h, --help            show this help message and exit

  basic options:
    --domain=DOMAIN     domain name
    --server=SERVER     IPA server
    --realm=REALM_NAME  realm name
    --fixed-primary     Configure sssd to use fixed server as primary IPA
                        server
    -p PRINCIPAL, --principal=PRINCIPAL
                        principal to use to join the IPA realm
    -w PASSWORD, --password=PASSWORD
                        password to join the IPA realm (assumes bulk password
                        unless principal is also set)
    -k KEYTAB, --keytab=KEYTAB
                        path to backed up keytab from previous enrollment
    -W                  Prompt for a password to join the IPA realm
    --mkhomedir         create home directories for users on their first login
    --hostname=HOSTNAME
                        The hostname of this machine (FQDN). If specified, the
                        hostname will be set and the system configuration will
                        be updated to persist over reboot. By default a
                        nodename result from uname(2) is used.
    --force-join        Force client enrollment even if already enrolled
    --ntp-server=NTP_SERVERS
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
    --noac              do not modify the nsswitch.conf and PAM configuration
    -f, --force         force setting of LDAP/Kerberos conf
    --kinit-attempts=KINIT_ATTEMPTS
                        number of attempts to obtain host TGT (defaults to 5).
    -d, --debug         print debugging information
    -U, --unattended    unattended (un)installation never prompts the user
    --ca-cert-file=CA_CERT_FILE
                        load the CA certificate from this file
    --request-cert      request certificate for the machine
    --automount-location=LOCATION
                        Automount location
    --configure-firefox
                        configure Firefox to use IPA domain credentials
    --firefox-dir=FIREFOX_DIR
                        specify directory where Firefox is installed (for
                        example: '/usr/lib/firefox')
    --ip-address=IP_ADDRESSES
                        Specify IP address that should be added to DNS. This
                        option can be used multiple times
    --all-ip-addresses  All routable IP addresses configured on any inteface
                        will be added to DNS

  SSSD options:
    --permit            disable access rules by default, permit all access.
    --enable-dns-updates
                        Configures the machine to attempt dns updates when the
                        ip address changes.
    --no-krb5-offline-passwords
                        Configure SSSD not to store user password when the
                        server is offline
    -S, --no-sssd       Do not configure the client to use SSSD for
                        authentication
    --preserve-sssd     Preserve old SSSD configuration if possible

  uninstall options:
    --uninstall         uninstall an existing installation. The uninstall can
                        be run with --unattended option
```

Testing
-------

[![Build Status](https://travis-ci.org/timorunge/ansible-freeipa-client.svg?branch=master)](https://travis-ci.org/timorunge/ansible-freeipa-client)

Dependencies
------------

This role requires an up and running
[FreeIPA Server](https://galaxy.ansible.com/timorunge/freeipa-server/)
([Github Repo](https://github.com/timorunge/ansible-freeipa-server)).

License
-------
BSD

Author Information
------------------

- Timo Runge
