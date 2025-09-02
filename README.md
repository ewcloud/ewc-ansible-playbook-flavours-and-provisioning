# Flavours and Provisioning
>💡 Two categories of templates in this collection, 1) one to configure previously provisioned instances, and 2) another to self-provisioning or manage provisioned instances (i.e. "one-line" deployment and full state management).

A collection of configuration templates
(i.e. [Ansible Playbooks](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks.html))
to customize your environment in the
[European Weather Cloud (EWC)](https://europeanweather.cloud/). See full list of Items in the [index](#index) below.

## Copyright and License
Copyright © EUMETSAT 2025.

The provided code and instructions are licensed under the [MIT license](./LICENSE).
They are intended to automate the setup of an environment that includes 
third-party software components.
The usage and distribution terms of the resulting environment are 
subject to the individual licenses of those third-party libraries.

Users are responsible for reviewing and complying with the licenses of
all third-party components included in the environment.

Contact [EUMETSAT](http://www.eumetsat.int) for details on the usage and distribution terms.

## Index
>💡 Grouped by capability and ordered alphabetically.

### Capability: Configuration (Only)

| Name  | Summary  |
|------|-----|
| [data-tailor-flavour](./playbooks/data-tailor-flavour/)    | Applies `ewc-ansible-role-data-tailor` on an existing instance   |
| [haproxy-flavour](./playbooks/haproxy-flavour/)    | Applies `ewc-ansible-role-haproxy-flavour` on an existing instance   |
| [ipa-client-disenroll-flavour](./playbooks/ipa-client-disenroll-flavour/)   |  Applies `ewc-ansible-role-ipa-client-disenroll` on and existing instance   |
| [ipa-client-enroll-flavour](./playbooks/ipa-client-enroll-flavour) | Apples `ewc-ansible-role-ipa-client-enroll` on an existing instance | 
| [ipa-server-flavour](./playbooks/ipa-server-flavour/)    |  Applies `ewc-ansible-role-ipa-server` on an existing instance  |
| [nginx-proxy-manager-flavour](./playbooks/nginx-proxy-manager/)    | Applies `ewc-ansible-role-nginx-proxy-manager` on an existing instance   |
| [remote-desktop-flavour](./playbooks/remote-desktop-flavour/) | Applies `ewc-ansible-role-remote-desktop` on an existing instance  | 
| [ssh-bastion-flavour](./playbooks/ssh-bastion-flavour/)   | Applies `ewc-ansible-role-ssh-bastion` on an existing instance | 

### Capability: Self-provisioning + Configuration

| Name  | Summary  |
|------|-----|
| [ipa-client-provisioning](./playbooks/ipa-client-provisioning/)    | Provision a new instance, or edits it if already existing, and then applies `ewc-ansible-role-ipa-client-enroll`    |
| [ipa-client-teardown](./playbooks/ipa-client-teardow/)   |  Applies `ewc-ansible-role-ipa-client-disenroll` on and existing instance and then tears it down  |
| [ipa-server-provisioning](./playbooks/ipa-server-provisioning/)    |  Provision a new instance, or edits it if already existing, and the applies `ewc-ansible-role-ipa-server`    |
| [remote-desktop-provisioning](./playbooks/remote-desktop-provisioning/) | Provision a new instance, or edits it if already existing, and then applies `ewc-ansible-role-remote-desktop`  | 
| [ssh-bastion-provisioning](./playbooks/ssh-bastion-provisioning/)   | Provision a new instance, or edits it if already existing, and then applies `ewc-ansible-role-ssh-bastion` | 

## Troubleshooting
Checkout the [troubleshooting documentation](./docs/troubleshooting.md) for
information on common problems and how to troubleshoot them.

## Changelog
All notable changes (i.e. fixes, features and breaking changes) are documented 
in the [CHANGELOG.md](./CHANGELOG.md).

## Contributing

Thanks for taking the time to join our community and start contributing!
Please make sure to:
* Familiarize yourself with our [Code of Conduct](./CODE_OF_CONDUCT.md) before 
contributing.
* See [CONTRIBUTING.md](./CONTRIBUTING.md) for instructions on how to request 
or submit changes.

## Authors

[European Weather Cloud](http://support.europeanweather.cloud/) 
<[support@europeanweather.cloud](mailto:support@europeanweather.cloud)>
