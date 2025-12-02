# SSH bastion provisioning
>✅ This template can be safely applied from any local work environment, even running outside an EWC tenancy's private network.

The [SSH](https://en.wikipedia.org/wiki/Secure_Shell) proxy or bastion server
is a barrier between your internal machines (without public or floating IPs)
and the public internet. With the SSH proxy, you'll have an extra layer of
security on top of your instances. It's equipped with
[Fail2ban](https://github.com/fail2ban/fail2ban),
intrusion prevention software designed to prevent brute-force attacks.

This is a configuration template
(i.e. an [Ansible Playbook](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks.html))
to customize your environment in the
[European Weather Cloud (EWC)](https://europeanweather.cloud/).

## Functionality

The template is designed to:

* Provision an instance via [Terraform](https://developer.hashicorp.com/terraform),
with your specified Linux distribution and desired flavor (a.k.a VM plan):
  * If a `terraform.tfstate` [state file](https://developer.hashicorp.com/terraform/language/state)
  is not found under the user-defined directory, attempts to create the
  instance from scratch

    OR
  * If  `terraform.tfstate` file is found, leverages Terraform's out-of-the-box functionality to update the instance referenced on it
* Configure the existing or newly provisioned RockyLinux virtual machines (with
public IP address), as entrypoint for users who wish to reach private EWC networks
 from the public internet via SSH.

After successful provisioning, you can leverage Terraform's functionality to modify or delete individual components safely. Each will have its own `main.tf` definition and `terraform.tfstate` state file under the corresponding user-defined local directories.

To learn the basics about managing infrastructure with Terraform, check out [Terraform in 100 seconds](https://youtu.be/tomUWcQ0P3k?si=CJwZJ7UaqpynDU-d) on YouTube. You can also find a step-by-step example applied to the EWC on the [official EWC documentation](https://confluence.ecmwf.int/x/2EDOIQ).

## Prerequisites

* Install [git](https://git-scm.com/downloads) (version 2.0 or higher )
* Install [python](https://www.python.org/downloads) (version 3.9 or higher) 
* Install [ansible](https://pypi.org/project/ansible) (version 2.15 or higher)
* Install [terraform](https://confluence.ecmwf.int/display/EWCLOUDKB/EWC+-+IaC+via+Terraform+and+OpenTofu#EWCIaCviaTerraformandOpenTofu-InstallationoftheCLI) (version 1.0 or higher)
* Get OpenStack API credentials (see [How to request OpenStack Application Credentials](https://confluence.ecmwf.int/display/EWCLOUDKB/EWC+-+How+to+request+Openstack+Application+Credentials) section of the EWC documentation)
* Create an SSH keypair (see [Creating Keys](https://confluence.ecmwf.int/display/EWCLOUDKB/Add+your+SSH+key+pair+to+Morpheus#AddyourSSHkeypairtoMorpheus-Creatingthekeys) section of the EWC documentation)
* Import your public SSH key to OpenStack (see [Import SSH Key](https://confluence.ecmwf.int/display/EWCLOUDKB/EWC+-+OpenStack+Command-Line+client#EWCOpenStackCommandLineclient-ImportSSHkey) section of the EWC documentation).

## Usage
> ⚠️ Only RockyLinux version 9 and 8 supported due to constrains imposed by [dependencies](#dependencies).

> 💡 A VM plan with at least 4GB of RAM is recommended for successful setup and
stable operation.

### 1. Clone the repository

```bash
git clone https://github.com/ewcloud/ewc-ansible-playbook-flavours-and-provisioning.git
```

#### 1.1. Change to the specific Item's subdirectory

```bash
cd ewc-ansible-playbook-flavours-and-provisioning/playbooks/ssh-bastion-provisioning
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

### 3. Configure and apply the template

#### 3.1. Interactive Mode

By running the following command, you can trigger an interactive session that
prompts you for the necessary user inputs, and then applies changes to your
target EWC environment:

```bash
ansible-playbook ssh-bastion-provisioning.yml
```

#### 3.2. Non-Interactive Mode

>💡 To learn more about defining variables at runtime, checkout the
[official Ansible documentation](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_variables.html).

You can also run in non-interactive mode by passing the
`--extra-vars` or `-e` flag, followed by a map of  key-value pairs; one for
each and every available input (see [inputs section](#inputs) below). For example:

```bash
ansible-playbook \
  -e '{
        "ewc_provider":"eumetsat",
        "ssh_bastion_tf_project_path":"~/ewc/ssh-bastion-1",
        "ssh_bastion_app_name":"ssh",
        "ssh_bastion_instance_name":"bastion",
        "ssh_bastion_instance_index":1,
        "ssh_bastion_flavor_name":"eo1.large",
        "ssh_bastion_image_name":"Rocky-9.5-20250604142417",
        "public_keypair_name":"my-public-key-name",
        "private_keypair_path":"~/.ssh/id_rsa",
        "private_network_name":"private",
        "security_group_name":"ssh",
        "fail2ban_whitelisted_ip_ranges":""
    }' \
  ssh-bastion-provisioning.yml
```
## Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|----------|
| ewc_provider | your target EWC provider. Must match that the provider of your OpenStack application credentials. Valid input values are `ecmwf` or `eumetsat`. | `string` | `eumetsat` | yes |
| ssh_bastion_tf_project_path | path to terraform working directory | `string` | `~/ewc/ssh-bastion-1` | yes |
| ssh_bastion_app_name | application name, used as prefix in the full instance name  | `string` | `ssh` | yes |
| ssh_bastion_instance_name| name of the instance, used in the full instance name | `string` | `bastion` | yes |
| ssh_bastion_instance_index | index or identifier for the instance, used as suffix in the full instance name | `number` | `1` | yes |
| ssh_bastion_flavor_name | name the flavor to use for the instance. To learn about available options, checkout the [official EWC VM plans documentation](https://confluence.ecmwf.int/display/EWCLOUDKB/EWC+VM+plans) | `string` | `eo1.large` | yes |
| ssh_bastion_image_name | name of the image to use for the instance. For complete information on  available options, see the [official EWC Images documentation](https://confluence.ecmwf.int/display/EWCLOUDKB/EWC+Virtual+Images+Available)  | `string` | `Rocky-9.5-20250604142417` | yes |
| public_keypair_name | name of public keypair (stored in OpenStack) to be copied into the instance for remote SSH access | `string` | n/a | yes |
| private_keypair_name | path to the local private keypair to use for SSH access to the instance  | `string` | `~/.ssh/id_rsa` | yes |
| private_network_name | private network name to attach the instance to  | `string` | `private` | yes |
| security_group_name | security group name to apply to the instance | `string` | `ssh` | yes |
| fail2ban_whitelisted_ip_ranges | IPv4 ranges (in CIDR format) to be whitelisted in Fail2ban configuration. Example: `['10.0.0.0/24','192.168.1.0/24']` | `list(string)` | `''` | no |

## Dependencies
> 💡 Upon execution, a SBOM (SPDX format) is auto-generated and stored in the VM's file system root directory (see `/sbom.json`).

| Name |  Home URL |
|------|---------|
| ewc-tf-module-openstack-compute |  https://github.com/ewcloud/ewc-tf-module-openstack-compute  |
| ewc-ansible-role-ssh-bastion |  https://github.com/ewcloud/ewc-ansible-role-ssh-bastion |

## Operation
Checkout the following how-to guides to learn about management of Item after initial setup:
* [How to connect to a VM hidden behind the SSH Bastion](../ssh-bastion-flavour/docs/how-to/how-to-connect-to-a-vm-hidden-behind-the-ssh-bastion.md)


## Troubleshooting
Checkout the [troubleshooting documentation](../../docs/troubleshooting.md) for
information on common problems and how to troubleshoot them.

