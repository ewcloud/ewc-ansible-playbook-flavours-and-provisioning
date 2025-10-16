# Nginx Proxy Manager Flavour
>💡 Not to be confused with Nginx, the web server and reverse proxy.

This Ansible Playbook configures an existing virtual machine running
within the [European Weather Cloud (EWC)](https://europeanweather.cloud/), to operate as a [Nginx Proxy Manager](https://nginxproxymanager.com/) server.

Nginx Proxy Manager is full-featured tool that helps to lower the barriers to entry for users who are interested in learning and working with [Nginx](https://nginx.org/en/) servers.

## Functionality
* Virtual host management
* Ability to cache assets
* Blocking of common exploits
* Websocket support
* Access list configuration
* SSL and HTTP/2 support
* Host redirection with HTTP code configuration
* TCP and UDP stream support
* User management
* Nginx Proxy
* Manager log auditing

## Prerequisites

* Install [git](https://git-scm.com/downloads) (version 2.0 or higher )
* Install [python](https://www.python.org/downloads) (version 3.9 or higher) 
* Install [ansible](https://pypi.org/project/ansible) (version 2.15 or higher)
* Get OpenStack API credentials (see [How to request OpenStack Application Credentials](https://confluence.ecmwf.int/display/EWCLOUDKB/EWC+-+How+to+request+Openstack+Application+Credentials) section of the EWC documentation)
* If you plan to configure an existing VM, jump to the [Usage](#usage) section below
* If you have not yet provisioned a VM, it is required to do so. You may choose one of the following approaches:
  * A) Provision a new VM via UI:
    * Create an SSH keypair (see [Creating the keys](https://confluence.ecmwf.int/display/EWCLOUDKB/Add+your+SSH+key+pair+to+Morpheus#AddyourSSHkeypairtoMorpheus-Creatingthekeys) section of the EWC documentation)
    * Import the SSH public key into Morpheus (see [Adding the keys in Morpheus](https://confluence.ecmwf.int/display/EWCLOUDKB/Add+your+SSH+key+pair+to+Morpheus#AddyourSSHkeypairtoMorpheus-AddingthekeysinMorpheus) section of the EWC documentation)
    * Provision a new VM through the web portal (see [Provision a new Instance - Web](https://confluence.ecmwf.int/display/EWCLOUDKB/Provision+a+new+instance+-+web) section of the EWC) documentation

    OR 
  * B) Provision a new VM via CLI:
    * Create an SSH keypair (see [Creating the keys](https://confluence.ecmwf.int/display/EWCLOUDKB/Add+your+SSH+key+pair+to+Morpheus#AddyourSSHkeypairtoMorpheus-Creatingthekeys) section of the EWC documentation)
    * Add you SSH public key to OpenStack (see [Import SSH Key](https://confluence.ecmwf.int/display/EWCLOUDKB/EWC+-+OpenStack+Command-Line+client#EWCOpenStackCommandLineclient-ImportSSHkey) section of the EWC documentation).
    * Provision a new VM via the OpenStack CLI (see [How to create a VM using the OpenStack CLI](https://confluence.ecmwf.int/display/EWCLOUDKB/EWC+-+How+to+create+a+VM+using+the+Openstack+CLI) section of the EWC documentation)

## Usage

### 1. Clone the repository

```bash
git clone https://github.com/ewcloud/ewc-ansible-playbook-flavours-and-provisioning.git
```

#### 1.1. Change to the specific Item's subdirectory

```bash
cd ewc-ansible-playbook-flavours-and-provisioning/playbooks/nginx-proxy-manager-flavour
```

#### 1.2. (Optional) Checkout an specific Item's version
>⚠️ Make sure to replace `x.y.z` in the command below, with your version of preference.

```bash
git checkout x.y.z
```

### 2. Download  Ansible dependencies
>💡 By default, Ansible Roles are installed under the `~/.ansible/roles` directory within your working environment.

Download the correct version of the Ansible dependencies, if you haven't done so already:

```
ansible-galaxy role install -r requirements.yml
```

### 3. Specify the target host and SSH credentials
Create an inventory file to specify address/credentials that Ansible should use
to reach the virtual machine you wish to configure:

```yaml
# inventory.yml
---
ewcloud:
  hosts:
    nginx_proxy_manager:
      ansible_python_interpreter: /usr/bin/python3
      ansible_host: <add the IPV4 address of the target host>
      ansible_ssh_private_key_file: <add the path to local SSH private key file>
      ansible_user: ubuntu
      ansible_ssh_common_args: -o StrictHostKeyChecking=accept-new
```

### 4. Configure and apply the template

#### 4.1. Interactive Mode

By running the following command, you can trigger an interactive session that
prompts you for the necessary user inputs, and then applies changes to your
target EWC environment:

```bash
ansible-playbook -i inventory.yml nginx-proxy-manager-flavour.yml
```

#### 4.2. Non-Interactive Mode

>💡 To learn more about defining variables at runtime, checkout the
[official Ansible documentation](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_variables.html).

You can also run in non-interactive mode by passing the
`--extra-vars` or `-e` flag, followed by a map of  key-value pairs; one for
each and every available input (see [inputs section](#inputs) below). For
example:

```bash
ansible-playbook \
  -i inventory.yml \
  -e "npm_admin_ui_port=8080" \
  -e "os_security_group_name=nginx-proxy-manager" \
  nginx-proxy-manager-flavour.yml
```

## Inputs
> ⛔ If deploying to an instance on the ECMWF site, using a high port numbers such as in the example above will prevent you from accessing the Nginx Proxy Manager UI from the pubic internet, even when a valid security group is attached to the instance. This is due to the outer perimeter firewall of the ECMWF site. For details see [EWC Security guidelines - Restrictive firewall (allow-listing)](https://confluence.ecmwf.int/display/EWCLOUDKB/EWC+Security+guidelines#EWCSecurityguidelines-Restrictivefirewall(allow-listing)).

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|:--------:|
| npm_admin_ui_port | port number at which the Nginx Proxy Manager admin UI is served | `number` | `8080`  | yes |
| os_security_group_name | OpenStack security group containing all firewall rules required for Nginx Proxy Manager operation  | `string` | `nginx-proxy-manager` | yes |


## Dependencies
> ⚠️ Only Ubuntu 22.04 images are currently supported.
This is due to constrains imposed by the required ewc-ansible-role-nginx-proxy-manager
Ansible Role.

> 💡 A VM plan with at least 8GB of RAM is recommended for successful setup and
stable operation.

| Name | Version | License | Home URL |
|------|---------|------|------|
| ewc-ansible-role-nginx-proxy-manager | 1.0 |  MIT | https://github.com/ewcloud/ewc-ansible-role-nginx-proxy-manager |

## Operation
Checkout the following how-to guides to learn about management of the Item after initial setup:
* [How to configure the NGINX Proxy manager initial setup](./docs/how-to/how-to-configure-the-nginx-proxy-manager-initial-setup.md)
* [How to add a new proxy host for specific domain](./docs/how-to/how-to-add-a-new-proxy-host-for-specific-domain.md)
* [How to add a new SSL Certificate for domain/s](./docs/how-to/how-to-add-a-new-ssl-certificate-for-domains.md)
* [How to create/modify an access list](./docs/how-to/how-to-create-modify-an-access-list.md)