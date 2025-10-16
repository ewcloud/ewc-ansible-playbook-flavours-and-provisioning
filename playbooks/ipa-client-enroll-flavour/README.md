# IPA Client Enrollment Flavour

This Ansible Playbook configures an existing virtual machine in the
[European Weather Cloud (EWC)](https://europeanweather.cloud/) to operate as a
client of [IPA services](../ipa-server-flavour/).

IPA provides integrated identity management and DNS services, enabling
centralized user authentication, authorization, and resource discovery.

Suitable for tenant admins and tenant users alike, this template simplifies the
integration of VMs into a [FreeIPA](https://www.freeipa.org/page/Main_Page)-managed
fleet of instances within the EWC environment. Follow the [instructions below](#usage)
to enroll your instance.

## Functionality
The template is designed to:
- Configure a pre-existing virtual machine running Rocky Linux 8/9 or Ubuntu to connect to a
IPA server on the same subnet.
- Enable DNS resolution for discovering private hosts and public addresses.
- Allow remote access to the VM using centrally managed LDAP users via password authentication.

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
  
    OR
  * C) Deploy this template, together with a new VM, as part of the [IPA Client Provisioning Community Hub Item](https://europeanweather.cloud/community-hub/ipa-client-provisioning)

    OR
  * D) Deploy this template, together with a new VM, via the [EWCCLI](https://pypi.org/project/ewccli/)


## Usage

### 1. Clone the repository

```bash
git clone https://github.com/ewcloud/ewc-ansible-playbook-flavours-and-provisioning.git
```

#### 1.1. Change to the specific Item's subdirectory

```bash
cd ewc-ansible-playbook-flavours-and-provisioning/playbooks/ipa-client-enroll-flavour
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
>💡 To find out which is the default user for your chosen VM image,
checkout the [official EWC documentation](https://confluence.ecmwf.int/display/EWCLOUDKB/EWC+-+VM+images+and+default+users).

Create an inventory file to specify address/credentials that Ansible should use
to reach the virtual machine you wish to configure:

```yaml
# inventory.yml
---
ewcloud:
  hosts:
    ipa_client:
      ansible_python_interpreter: /usr/bin/python3
      ansible_host: <add the IPV4 address of the target host>
      ansible_ssh_private_key_file: <add the path to local SSH private key file>
      ansible_user: <add the default user according to your chosen VM image>
      ansible_ssh_common_args: -o StrictHostKeyChecking=accept-new

```

### 4. Configure and apply the template

#### 4.1. Interactive Mode

By running the following command, you can trigger an interactive session that
prompts you for the necessary user inputs, and then applies changes to your
target EWC environment:

```bash
ansible-playbook -i inventory.yml ipa-client-enroll-flavour.yml
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
        "ipa_admin_password": "my-secret-password"
    }' \
  ipa-client-enroll-flavour.yml
```

## Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|----------|
| ipa_domain | domain name managed by the IPA server. Example: `eumetsat.sandbox.ewc` | `string` | n/a | yes |
| ipa_server_hostname | hostname of the IPA server | `string`| `ipa-server-1` | yes |
| ipa_admin_username | username of the administrator account from the IPA server | `string` | `ipaadmin` | yes |
| ipa_admin_password | password of the administrator account from the IPA server. Example: `my-secret-password` | `string` | n/a | yes |

## Dependencies
> ⚠️ Only Ubuntu 22.04 and RockyLinux 8.10 VM images are currently supported.
This is due to constrains imposed by the required
ewc-ansible-role-ipa-client-enroll Ansible Role.

| Name | Version | License |Home URL |
|------|---------|------|-----|
| ewc-ansible-role-ipa-client-enroll | 1.1 | MIT | https://github.com/ewcloud/ewc-ansible-role-ipa-client-enroll |
