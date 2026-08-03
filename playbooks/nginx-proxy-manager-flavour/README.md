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

> 💡 This Item is supported by the [EWCCLI](https://www.europeanweather.cloud/community-hub/ewc-cli), 
and can be deployed, together with a compatible VM, via it. Checkout the [EWC User Stories: I want to use the ewccli](https://confluence.ecmwf.int/x/NlYiK) documentation pages to learn how.

* Install [git](https://git-scm.com/downloads) (version 2.0 or higher )
* Install [python](https://www.python.org/downloads) (version 3.9 or higher) 
* Install [ansible](https://pypi.org/project/ansible) (version 2.15 or higher)

* Verify the `nginx-proxy-manager` OpenStack Security Group exists in your EWC tenancy
  > 💡 You may create Security Groups via [this EWC Community Hub Item](https://europeanweather.cloud/community-hub/openstack-compute-instance) if pre-required ones are missing.
* If you plan to configure an existing VM, ensure it meets the minium requirements before moving on to the [Usage](#usage) section below:
  * VM Image: Ubuntu 24 or 22
  * VM Plan: 4 CPU cores, 8GB RAM, 40GB Disk
  * Network: Private
  * Security Groups: `nginx-proxy-manager`
  * Floating IP: Optional (not recommended from a security standpoint)
  
  Otherwise, provision a new VM with above specifications before continuing (see [EWC Getting Started: Provision a VM](https://confluence.ecmwf.int/x/2RvEJg) for details).


## Usage

### 1. Clone the repository

```bash
git clone https://github.com/ewcloud/ewc-ansible-playbook-flavours-and-provisioning.git
```

#### 1.1. Change to the specific Item's subdirectory

```bash
cd ewc-ansible-playbook-flavours-and-provisioning/playbooks/nginx-proxy-manager-flavour
```

#### 1.2. Checkout an specific Item's version
>⚠️ Make sure to replace `x.y.z` in the command below, with your version of preference.

```bash
git checkout x.y.z
```

### 2. Download Ansible dependencies
>💡 By default, Ansible Roles are installed under the `~/.ansible/roles` directory within your working environment.

Download the correct version of the Ansible dependencies, if you haven't done so already:

```
ansible-galaxy role install --force -r requirements.yml
```

### 3. Specify the target host and SSH credentials
Create an inventory file, to specify address/credentials that your local working environment should use
to connect to the target VM.

Copy into the file one of the two snippets below, and replace the placeholders (i.e. values enclosed in `<` `>` characters):

* **Connecting form within the EWC tenancy's network**

  ```yaml
  # inventory.yml
  ---
  ewcloud:
    hosts:
      target:
        ansible_python_interpreter: /usr/bin/python3
        ansible_host: <add the IPV4 address of the target host>
        ansible_ssh_private_key_file: <add the path to local SSH private key file>
        ansible_user: ubuntu
        ansible_ssh_common_args: -o StrictHostKeyChecking=no
  ```

**OR**


*  **Connecting from outside the EWC tenancy's network**

    > ⚠️ This requires an [SSH Bastion](https://europeanweather.cloud/community-hub/ssh-bastion-provisioning) to be already provisioned within your EWC tenancy.

    ```yaml
    # inventory.yml
    ---
    ewcloud:
      hosts:
        target:
          ansible_host: <add the IP address of the target host>
          ansible_ssh_user: ubuntu
          ansible_ssh_private_key_file: <add the path to local SSH private key file>
          ansible_python_interpreter: auto

    all:
      vars:
        ansible_ssh_common_args: >- 
          -o StrictHostKeyChecking=no
          -o UserKnownHostsFile=/dev/null
          -o ProxyCommand="ssh 
                          -o StrictHostKeyChecking=no
                          -o UserKnownHostsFile=/dev/null
                          -o BatchMode=yes
                          -W %h:%p
                          -i <add the path to local SSH private key file> 
                          cloud-user@<add the IP address of the ssh bastion>"

    ```

### 4. Configure and apply the template

* **Interactive Mode**

  By running the following command, you can trigger an interactive session that
  prompts you for the necessary user inputs, and then applies changes to your
  target EWC environment:

  ```bash
  ansible-playbook -i inventory.yml nginx-proxy-manager-flavour.yml
  ```

**OR**

* **Non-Interactive Mode**

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
    nginx-proxy-manager-flavour.yml
  ```

## Inputs
> ⛔ If deploying to an instance on the ECMWF site, using a high port numbers such as in the example above will prevent you from accessing the Nginx Proxy Manager UI from the pubic internet, even when a valid security group is attached to the instance. This is due to the outer perimeter firewall of the ECMWF site. For details see [EWC Security guidelines - Restrictive firewall (allow-listing)](https://confluence.ecmwf.int/x/o4DlFg#EWCSecurityguidelines-Restrictivefirewall(allow-listing)).

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|:--------:|
| npm_admin_ui_port | port number at which the Nginx Proxy Manager admin UI is served | `number` | `8080`  | yes |


## Dependencies

> 💡 Upon execution, a SBOM (SPDX format) is auto-generated and stored in the VM's file system root directory (see `/sbom.json`).

| Name | Home URL |
|------|---------|
| ewc-ansible-role-nginx-proxy-manager | https://github.com/ewcloud/ewc-ansible-role-nginx-proxy-manager |

## Operation
Checkout the following how-to guides to learn about management of the Item after initial setup:
* [How to configure the NGINX Proxy manager initial setup](./docs/how-to/how-to-configure-the-nginx-proxy-manager-initial-setup.md)
* [How to add a new proxy host for specific domain](./docs/how-to/how-to-add-a-new-proxy-host-for-specific-domain.md)
* [How to add a new SSL Certificate for domain/s](./docs/how-to/how-to-add-a-new-ssl-certificate-for-domains.md)
* [How to create/modify an access list](./docs/how-to/how-to-create-modify-an-access-list.md)