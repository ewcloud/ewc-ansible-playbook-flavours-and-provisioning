# SSH Bastion Flavour
The [SSH](https://en.wikipedia.org/wiki/Secure_Shell) bastion or proxy server
is a barrier between your internal machines (which lack a public or floating IP
address) and the public internet. With the SSH proxy, you'll have an extra layer of
security on top of your instances. It's equipped with
[Fail2ban](https://github.com/fail2ban/fail2ban),
intrusion prevention software designed to prevent brute-force attacks.

This template is for tenant admins wishing to hardening the way tenant users
connect to the [European Weather Cloud (EWC)](https://europeanweather.cloud/),
as well as tenant users whom are mindful about safe-keeping the compute resources
or data withing their work environments.

## Functionality
The template is designed to:

* Configure a pre-existing virtual machine running RockyLinux, with public IP
address, and a minimum recommended 4GB of RAM, as entrypoint for users who
wish to reach private EWC networks, from the public internet, via SSH.

## Prerequisites
> ⚠️ Only RockyLinux versions 9 or 8 are supported due
to constrains imposed by [dependencies](#dependencies).

> 💡 A VM plan with at least 4GB of RAM is recommended for successful setup and
stable operation.

* Install [git](https://git-scm.com/downloads) (version 2.0 or higher )
* Install [python](https://www.python.org/downloads) (version 3.9 or higher) 
* Install [ansible](https://pypi.org/project/ansible) (version 2.15 or higher)
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
  * C) Deploy this template, together with a new VM, as part of the [SSH Bastion Provisioning Community Hub Item](https://europeanweather.cloud/community-hub/ssh-bastion-provisioning)

    OR
  * D) Deploy this template, together with a new VM, via the [EWCCLI](https://pypi.org/project/ewccli/)


## Usage

### 1. Clone the repository

```bash
git clone https://github.com/ewcloud/ewc-ansible-playbook-flavours-and-provisioning.git
```

#### 1.1. Change to the specific Item's subdirectory

```bash
cd playbooks/ssh-bastion-flavour
```

#### 1.2. (Optional) Checkout an specific Item's version
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
Create an inventory file to specify address/credentials that Ansible should use
to reach the virtual machine you wish to configure:

```yaml
# inventory.yml
---
ewcloud:
  hosts:
    ssh_bastion:
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
ansible-playbook -i inventory.yml ssh-bastion-flavour.yml
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
  -e '{"fail2ban_whitelisted_ip_ranges":""}' \
  ssh-bastion-flavour.yml
```

## Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|----------|
| fail2ban_whitelisted_ip_ranges | IPv4 ranges (in CIDR format) to be whitelisted in Fail2ban configuration. When in doubt, do not set. Example: `['10.0.0.0/24','192.168.1.0/24']` | `list(string)` | `''` | no |

## Dependencies

> 💡 Upon execution, a SBOM (SPDX format) is auto-generated and stored in the VM's file system root directory (see `/sbom.json`).

| Name  | Home URL |
|-------|-------|
| ewc-ansible-role-ssh-bastion | https://github.com/ewcloud/ewc-ansible-role-ssh-bastion |

## Operation
Checkout the following how-to guides to learn about management of Item after initial setup:
* [How to connect to a VM hidden behind the SSH Bastion](./docs/how-to/how-to-connect-to-a-vm-hidden-behind-the-ssh-bastion.md)
