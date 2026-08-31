# IPA Client Disenrollment Flavour

This Ansible Playbook configures an existing virtual machine in the
[European Weather Cloud (EWC)](https://europeanweather.cloud/) to disenroll
from a [IPA server](../ipa-server-flavour/).

IPA provides integrated identity management and DNS services for centralized
user authentication, authorization, and resource discovery. This template safely
removes a VM from a [FreeIPA](https://www.freeipa.org/page/Main_Page)-managed
environment, which is essential when decommissioning infrastructure to prevent
stale entries in the IPA directory—such as obsolete host records, and DNS
pointers—that could lead to management overhead, naming conflicts or security
vulnerabilities from lingering credentials.

Suitable for both tenant admin and tenant users, this template streamlines client
removal, ensuring a clean and efficient identity system. The end benefit for
users is reduced administrative burden, improved security posture,
and easier scaling or redeployment of resources. Follow the [instructions below](#usage)
to disenroll your instance.

## Functionality
The template is designed to:
- Configure a pre-existing virtual machine, previously enrolled (likely with help of the [IPA client](../ipa-client-enroll-flavour/) template) to disconnect from an [IPA server](../ipa-server-flavour/).
- Disable user authentication and authorization (LDAP) for the target instance.
- Remove IPA server-internal DNS records referencing the target instance, if present.


## Prerequisites

* Install [git](https://git-scm.com/downloads) (version 2.0 or higher )
* Install [python](https://www.python.org/downloads) (version 3.9 or higher) 
* Install [ansible](https://pypi.org/project/ansible) (version 2.15 or higher)
* Verify the IP address of the existing VM already enrolled into your IPA Server.
* Verify your SSH access to the exiting VM (i.e. your public SSH key must be registered on the target machine)

## Usage

#### 2. Clone the repository

```bash
git clone https://github.com/ewcloud/ewc-ansible-playbook-flavours-and-provisioning.git
```

##### 2.1. Change to the specific Item's subdirectory

```bash
cd ewc-ansible-playbook-flavours-and-provisioning/playbooks/ipa-client-disenroll-flavour
```

##### 2.2. Checkout an specific Item's version
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
>💡 To find out which is the default user for your chosen VM image,
checkout the [official EWC documentation](https://confluence.ecmwf.int/x/USRNH#EWCOpenStackAPIaccessVMimagesanddefaultusers-Defaultusers).

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
        ansible_user: <add the default user according to your chosen VM image>
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
        ansible_user: <add the default user according to your chosen VM image>
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
          ansible_ssh_user: <add the default user according to your chosen VM image>
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
  ansible-playbook -i inventory.yml ipa-client-disenroll-flavour.yml
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
        "ipa_domain": "internal-eumetsat-sandbox.ewc",
        "ipa_client_hostname": "ipa-client-1",
        "ipa_server_hostname": "ipa-server-1",
        "ipa_admin_username": "ipaadmin",
        "ipa_admin_password": "my-secret-password"
      }' \
    ipa-client-disenroll-flavour.yml
  ```

## Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|----------|
| ipa_domain | domain name managed by the IPA server. Example: `internal-eumetsat-sandbox.ewc` | `string` | n/a | yes |
| ipa_client_hostname | hostname of the target vm to be disenrolled from the IPA server. Example: `ipa-client-1` | `string` | n/a | yes |
| ipa_server_hostname | hostname of the IPA server | `string` | `ipa-server-1` | yes |
| ipa_admin_username | username of the administrator account from the IPA server | `string` | `ipaadmin` | yes |
| ipa_admin_password | password of the administrator account from the IPA server. Example: `my-secret-password` | `string` | n/a | yes |


## Dependencies


| Name | Home URL |
|------|---------|
| ewc-ansible-role-ipa-client-disenroll | https://github.com/ewcloud/ewc-ansible-role-ipa-client-disenroll |
