# Flavours and Provisioning
>💡 Two categories of templates in this collection, 1) one to configure previously provisioned instances, and 2) another to self-provisioning or manage provisioned instances (i.e. "one-line" deployment and full state management).

A collection of configuration templates
(i.e. [Ansible Playbooks](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks.html))
to customize your
[European Weather Cloud (EWC)](https://europeanweather.cloud/) tenancy. See full list of Items in the [index](#index) below.

Want to learn how other users make the best out of these templates, or have ideas of your own? Head over to [the official discussion platform](https://chat.europeanweather.cloud/) and engage with the EWC community—feedback is always welcome!♥️



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
| [data-tailor-flavour](./playbooks/data-tailor-flavour/)    |Transforms an existing VM into a powerful satellite data customization hub, enabling users to efficiently subset, aggregate, reproject, and reformat data from METOP, MFG, MSG, MTG, and Sentinel-3 into GIS and image formats, offering faster processing and greater flexibility than web-based alternatives.  |
| [haproxy-flavour](./playbooks/haproxy-flavour/)    | Configures an existing VM as a high-performance load balancer, enhancing application speed, security, and scalability with easy management for TCP and HTTP workloads.   |
| [ipa-client-disenroll-flavour](./playbooks/ipa-client-disenroll-flavour/)   |  Simplifies the secure removal of a running VM from a FreeIPA-managed fleet of instances, reducing administrative overhead and enhancing security by eliminating stale credentials and DNS records. |
| [ipa-client-enroll-flavour](./playbooks/ipa-client-enroll-flavour) | Seamlessly integrates a running VM into a FreeIPA-managed fleet of instances, enabling centralized user authentication, DNS resolution, and secure remote access for simplified and scalable identity management. | 
| [ipa-server-flavour](./playbooks/ipa-server-flavour/)    | Turns an existing VM into a FreeIPA server, a central place for user authentication, authorization, and DNS-based resource discovery for secure and efficient identity management. |
| [nginx-proxy-manager-flavour](./playbooks/nginx-proxy-manager-flavour/)    | Configures an existing VM as a user-friendly Nginx Proxy Manager server, simplifying virtual host management, SSL/HTTP/2 support, and security features like exploit blocking for efficient and secure proxy operations.  |
| [remote-desktop-flavour](./playbooks/remote-desktop-flavour/) | Transforms an existing VM into a secure, graphical desktop environment using X2Go and MATE, enabling simple remote access and intuitive cloud-based development for tenant users.  | 
| [ssh-bastion-flavour](./playbooks/ssh-bastion-flavour/)   | Tightens the configuration of a running VM, to operate as a secure SSH proxy with Fail2ban, providing tenant admins and users a fortified entry point to safely access private EWC networks from the public internet. | 

### Capability: Self-provisioning + Configuration

| Name  | Summary  |
|------|-----|
| [ipa-client-provisioning](./playbooks/ipa-client-provisioning/)    | Automates the creation or state update of a VM, plus its configuration as an IPA client, effectively enabling integration with a fleet of instances with centralized authentication, secure remote access, and DNS-based resource discovery in the EWC. |
| [ipa-client-teardown](./playbooks/ipa-client-teardown/)   | Simplifies the secure teardown of an IPA client VM in the EWC, disabling LDAP authentication, removing DNS records, and safely decommissioning the instance and its resources.  |
| [ipa-server-provisioning](./playbooks/ipa-server-provisioning/)    |  Automates the creation or state update of VM, plus its configuration as FreeIPA server, streamlining centralized user management, authentication, authorization, and DNS resolution for secure and efficient resource management.  |
| [remote-desktop-provisioning](./playbooks/remote-desktop-provisioning/) | Automates the creation or state update of a remote desktop VM, a basic yet secure cloud-based development environment with graphical interface. Uses X2Go and MATE to enable easy remote access for tenant users | 
| [ssh-bastion-provisioning](./playbooks/ssh-bastion-provisioning/)   | Automates the creation or state update of a secure SSH bastion VM in the EWC, configured with Fail2ban to provide a fortified entry point for safely accessing private EWC networks from the public internet. | 

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
