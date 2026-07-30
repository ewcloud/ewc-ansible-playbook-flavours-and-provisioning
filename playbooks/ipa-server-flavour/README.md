# IPA Server Flavour

This Ansible Playbook configures an existing virtual machine running
within the [European Weather Cloud (EWC)](https://europeanweather.cloud/)
to operate as a [FreeIPA](https://www.freeipa.org/page/Main_Page) server.

IPA (acronym for identity, policy and audit), provides integrated identity
management and DNS services, enabling centralized user authentication, authorization,
and resource discovery.

Ideal for tenant administrators, this template simplifies the setup
of a secure, open-source identity and DNS solution in the EWC environment. Follow the
[instructions below](#usage) to configure your server.

## Functionality
The template is designed to:
* Validate that network/subnet configuration in the EWC tenancy
* Configure a pre-existing virtual machine running RockyLinux version 8 or 9,
and with a minimum recommended 4GB of RAM, such that it:
  * Provides DNS resolutions for discovery of resources (i.e. other virtual
  machines)
  * Enables centralized user and credentials creation/edition/deletion/authentication
  * Allows centralized authorization between users and resources

## Prerequisites

> 💡 This Item is supported by the [EWCCLI](https://www.europeanweather.cloud/community-hub/ewc-cli), 
and can be deployed, together with a compatible VM, via it. Checkout the [EWC User Stories: I want to use the ewccli](https://confluence.ecmwf.int/x/NlYiK) documentation pages to learn how.

* Install [git](https://git-scm.com/downloads) (version 2.0 or higher )
* Install [python](https://www.python.org/downloads) (version 3.9 or higher) 
* Install [ansible](https://pypi.org/project/ansible) (version 2.15 or higher)

* Verify the `ipa` OpenStack Security Group exists in your EWC tenancy
  > 💡 You may create Security Groups via [this EWC Community Hub Item](https://europeanweather.cloud/community-hub/openstack-compute-instance) if pre-required ones are missing.
* If you plan to configure an existing VM, ensure it meets the minium requirements before moving on to the [Usage](#usage) section below:
  * VM Image: RockyLinux 9 or 8
  * VM Plan: 4 CPU cores, 4GB RAM, 30GB Disk
  * Network: Private
  * Security Groups: `ipa`
  * Floating IP: Optional (not recommended from a security standpoint)
  
  Otherwise, provision a new VM with above specifications before continuing (see [EWC Getting Started: Provision a VM](https://confluence.ecmwf.int/x/2RvEJg) for details).

## Usage

### 1. Clone the repository

```bash
git clone https://github.com/ewcloud/ewc-ansible-playbook-flavours-and-provisioning.git
```

#### 1.1. Change to the specific Item's subdirectory

```bash
cd ewc-ansible-playbook-flavours-and-provisioning/playbooks/ipa-server-flavour
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
ansible-galaxy role install -r requirements.yml
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
        ansible_python_interpreter: auto
        ansible_host: <add the IP address of the target host>
        ansible_ssh_private_key_file: <add the path to local SSH private key file>
        ansible_user: cloud-user
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
          ansible_ssh_user: cloud-user
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
  ansible-playbook -i inventory.yml ipa-server-flavour.yml
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
    -e '{
        "ipa_domain": "eumetsat.sandbox.ewc",
        "ipa_server_hostname": "ipa-server-1",
        "ipa_admin_username": "ipaadmin",
        "ipa_admin_password": "my-secret-password",
        "ipa_admin_givenname": "EWC",
        "ipa_admin_surname": "IPAADMIN",
        "os_network_name": "private",
        "os_security_group_name": "ipa"
      }' \
    ipa-server-flavour.yml
  ```

### 5. Manually update DNS nameserver(s)

>⛔ Changes described in this section can potentially affect DNS resolution on existing VMs within your subnet. To prevent issues, enroll them to the new IPA server via the
[IPA Client Enroll Flavour](https://europeanweather.cloud/community-hub/ipa-client-enroll-flavour)
CommunityHub Item, OR manually [edit nameservers in their DNS configuration](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/6/html/identity_management_guide/domain-dns).

Manual changes to your OpenStack subnet are required for IPA to be fully enabled.

**Step 1: Note down the private IP address of the VM newly configured as IPA server**

The IP address may already be in your `inventory.yml` file, or in the logs of your VM deployment method of choice otherwise. You can also obtain it via OpenStack CLI or find it by exploring VMs via the UI.

For illustration purposes, supposes it IP is `10.0.0.53`.


**Step 2: Ensure the IPA server's IP address is part of the list of subnet DSN nameservers**

  * **Via [EWC Cloud Management UI](https://confluence.ecmwf.int/x/KnAEJQ)**
    1. Click on `Project > Network > Networks`.
    2. A table will be displayed in on the middle of the view port, listing available networks. Click on network name.
    3. Click on the `Subnets` tab.
    4. A new table will be displayed in on the middle of the view port, this time listing the available subnets. Click on `Edit Subnet`, on the same row where the correct subnet is listed.
    5. Within the pop-up edit form, click on `Subnet Details`
    6. Replace the contents of the `DNS Name Servers` entry with the IP of your new IPA server, for example:

        <img src="https://raw.githubusercontent.com/ewcloud/ewc-ansible-playbook-flavours-and-provisioning/main/playbooks/ipa-server-flavour/docs/images/horinzon-ui-dns-name-servers.jpg" height=600px>

    7. Click on `Save`

  **OR**

  * **Via [OpenStack CLI](pypi.org/project/python-openstackclient/)**
    
    Following the example in the prior step and assuming the subnet name `private-subnet`, then we would execute:

    ```bash
    openstack subnet set --dns-nameserver 10.0.0.53 private-subnet
    ```

    Supposing the two additional IP address, `1.1.1.1` and `8.8.8.8`, are still part of the DNS nameservers list, you can remove then with:

    ```bash
    openstack subnet unset --dns-nameserver 1.1.1.1 private-subnet
    ```
    ```bash
    openstack subnet unset --dns-nameserver 8.8.8.8 private-subnet
    ```



## Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|----------|
| ipa_domain | domain name to be managed by the IPA server. Example: `eumetsat.sandbox.ewc` | `string` | n/a | yes |
| ipa_server_hostname | hostname of the target vm where the IPA server will be installed | `string`| `ipa-server-1` | yes |
| ipa_admin_username | username of administrator account to replace the default IPA admin | `string` | `ipaadmin` | yes |
| ipa_admin_password | password of administrator account to replace the default IPA admin. Example: `my-secret-password` | `string` | n/a | yes |
| ipa_admin_givenname | given name of the administrator to replace the default IPA admin (not necessarily a real person's name) | `string` | `EWC` | yes |
| ipa_admin_surname | surname of the administrator to replace the default IPA admin (not necessarily a real person's name) | `string` | `IPAADMIN` | yes |
| os_network_name | OpenStack network to which the target virtual machine has access to | `string` | `private` | yes |
| os_security_group_name | OpenStack security group containing all firewall rules required by the IPA server/client communication | `string` | `ipa` | yes |

## Dependencies

> 💡 Upon execution, a SBOM (SPDX format) is auto-generated and stored in the VM's file system root directory (see `/sbom.json`).

| Name | Home URL |
|------|---------|
| ewc-ansible-role-ipa-server | https://github.com/ewcloud/ewc-ansible-role-ipa-server |

## Operation
Checkout the following how-to guides to learn about management of the Item after initial setup:
* [How to configure the IPA Server](./docs/how-to/how-to-configure-the-ipa-server.md)
