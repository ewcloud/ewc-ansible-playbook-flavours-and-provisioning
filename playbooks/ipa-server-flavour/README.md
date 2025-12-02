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

> ⚠️ Only RockyLinux versions 9 or 8 are supported due
to constrains imposed by [dependencies](#dependencies).

> 💡 A VM plan with at least 4GB of RAM is recommended for successful setup and
stable operation.

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
  
    OR
  * C) Deploy this template, together with a new VM, as part of the [IPA Server Provisioning Community Hub Item](https://europeanweather.cloud/community-hub/ipa-server-provisioning)

    OR
  * D) Deploy this template, together with a new VM, via the [EWCCLI](https://pypi.org/project/ewccli/)


## Usage

### 1. Clone the repository

```bash
git clone https://github.com/ewcloud/ewc-ansible-playbook-flavours-and-provisioning.git
```

#### 1.1. Change to the specific Item's subdirectory

```bash
cd ewc-ansible-playbook-flavours-and-provisioning/playbooks/ipa-server-flavour
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
    ipa_server:
      ansible_python_interpreter: /usr/bin/python3
      ansible_host: <add the IPV4 address of the target host>
      ansible_ssh_private_key_file: <add the path to local SSH private key file>
      ansible_user: cloud-user
      ansible_ssh_common_args: -o StrictHostKeyChecking=accept-new
```

### 4. Configure and apply the template

#### 4.1. Interactive Mode

By running the following command, you can trigger an interactive session that
prompts you for the necessary user inputs, and then applies changes to your
target EWC environment:

```bash
ansible-playbook -i inventory.yml ipa-server-flavour.yml
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

### 5. Manullay update DNS nameserver(s)

After successful execution of the template, additional changes to the OpenStack subnet is required.
You can edit your specific OpenStack subnet, as well as any other OpenStack resource, with the native [OpenStack CLI](pypi.org/project/python-openstackclient/).

Take note of the IP address of your newly configured IPA server and the subnet attached to it, replace these information in the command below, and execute:

```bash
openstack subnet set \
  --dns-nameserver <IPV4 address of the IPA server> \
  <ID or name of the OpenStack Subnet attached to the IPA server>
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

| Name | Version | License | Home URL |
|------|---------|------|------|
| ewc-ansible-role-ipa-server | 1.1 |  MIT | https://github.com/ewcloud/ewc-ansible-role-ipa-server |

## Operation
Checkout the following how-to guides to learn about management of the Item after initial setup:
* [How to configure the IPA Server](./docs/how-to/how-to-configure-the-ipa-server.md)
