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

* Verify the `ssh-https` OpenStack Security Group exists in your EWC tenancy
  >💡 You may create Security Groups via [this EWC Community Hub Item](https://europeanweather.cloud/community-hub/openstack-compute-instance) if pre-required ones are missing.


## Usage

### Deploy via EWCCLI

>⚠️ By default, deploying via EWCCLI is only possible from within your EWC private network. You may override by passing the `--external-ip` flag upon deployment. Caution is advised as this weakens security on the deployed VM.

#### 1. Setup working environment

```bash
pip install ewccli
```

#### 2. Configure access credentials

```bash
ewc login
```

#### 3. Deploy
>💡 To lean about EWCCLI deployment customization, checkout the [EWC User Stories: I want to use the ewccli](https://confluence.ecmwf.int/x/NlYiK) documentation pages.


```bash
ewc hub deploy ssh-bastion-flavour
```

### Deploy via native tooling (Ansible)

#### 1. Setup working environment

* Install [git](https://git-scm.com/downloads) (version 2.0 or higher )
* Install [python](https://www.python.org/downloads) (version 3.9 or higher)
* Install [ansible](https://pypi.org/project/ansible) (version 2.15 or higher)
* If you plan to configure an existing VM, ensure it meets the minium requirements before proceeding:
  * VM Image: RockyLinux 8 or 9
  * VM Plan: 2 CPU cores, 4GB RAM, 30GB Disk
  * Network: Private
  * Security Groups: `ssh-https`
  * Floating IP: Required

  Otherwise, provision a new VM with above specifications before continuing (see [EWC Getting Started: Provision a VM](https://confluence.ecmwf.int/x/2RvEJg) for details).


#### 2. Clone the repository

```bash
git clone https://github.com/ewcloud/ewc-ansible-playbook-flavours-and-provisioning.git
```

#### 2.1. Change to the specific Item's subdirectory

```bash
cd playbooks/ssh-bastion-flavour
```

#### 2.2. Checkout an specific Item's version
>⚠️ Make sure to replace `x.y.z` in the command below, with your version of preference.

```bash
git checkout x.y.z
```


#### 3. Download Ansible dependencies
>💡 By default, Ansible Roles are installed under the `~/.ansible/roles` directory within your working environment.

Download the correct version of the Ansible dependencies, if you haven't done so already:

```
ansible-galaxy role install --force -r requirements.yml
```

#### 4. Specify the target host and SSH credentials
Create an inventory file, to specify address/credentials that your local working environment should use
to connect to the target VM.

Copy into the file one of the two snippets below, and replace the placeholders (i.e. values enclosed in `<` `>` characters):

* **To connect within the EWC private network**

  ```yaml
  # inventory.yml
  ---
  ewcloud:
    hosts:
      target:
        ansible_python_interpreter: auto
        ansible_host: <add the PRIVATE IP address of the target host>
        ansible_ssh_private_key_file: <add the path to local SSH private key file>
        ansible_user: cloud-user
        ansible_ssh_common_args: -o StrictHostKeyChecking=no
  ```

**OR**

* **To connect from the public internet to a EWC public IP address**

  ```yaml
  # inventory.yml
  ---
  ewcloud:
    hosts:
      target:
        ansible_python_interpreter: auto
        ansible_host: <add the PUBLIC IP address of the target host>
        ansible_ssh_private_key_file: <add the path to local SSH private key file>
        ansible_user: cloud-user
        ansible_ssh_common_args: -o StrictHostKeyChecking=no
  ```

**OR**

*  **To connect from the public the public internet to a EWC private IP address**

    > ⚠️ This requires an [SSH Bastion](https://europeanweather.cloud/community-hub/ssh-bastion-provisioning) to be already provisioned within your EWC tenancy.

    ```yaml
    # inventory.yml
    ---
    ewcloud:
      hosts:
        target:
          ansible_host: <add the PRIVATE IP address of the target host>
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
                          cloud-user@<add the PUBLIC IP address of the SSH bastion>"

    ```

#### 5. Configure and apply the template

* **Interactive Mode**

  By running the following command, you can trigger an interactive session that
  prompts you for the necessary user inputs, and then applies changes to your
  target EWC environment:

  ```bash
  ansible-playbook -i inventory.yml ssh-bastion-flavour.yml
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
        "fail2ban_whitelisted_ip_ranges": ""
      }' \
    ssh-bastion-flavour.yml
  ```

## Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|----------|
| fail2ban_whitelisted_ip_ranges | IPv4 ranges (in CIDR format) to be whitelisted in Fail2ban configuration. When in doubt, do not set. Example: `['10.0.0.0/24','192.168.1.0/24']` | `list(string)` | `null` | no |

## Dependencies


> 💡 Upon execution, a SBOM (SPDX format) is auto-generated and stored in the VM's file system root directory (see `/sbom.json`).

| Name  | Home URL |
|-------|-------|
| ewc-ansible-role-ssh-bastion | https://github.com/ewcloud/ewc-ansible-role-ssh-bastion |

## Operation
Checkout the following how-to guides to learn about management of Item after initial setup:
* [How to connect to a VM hidden behind the SSH Bastion](./docs/how-to/how-to-connect-to-a-vm-hidden-behind-the-ssh-bastion.md)
