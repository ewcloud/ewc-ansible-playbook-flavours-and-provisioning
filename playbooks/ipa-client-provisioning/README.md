# IPA client provisioning

This is a configuration template
(i.e. an [Ansible Playbook](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks.html))
to customize your environment in the
[European Weather Cloud (EWC)](https://europeanweather.cloud/).

The template is designed to:

* Provision an instance via [Terraform](https://developer.hashicorp.com/terraform),
with your specified Linux distribution and desired flavor (a.k.a VM plan):
  * If a `terraform.tfstate` [state file](https://developer.hashicorp.com/terraform/language/state)
  is not found under the user-defined directory, attempts to create the
  instance from scratch

  OR
  * if  `terraform.tfstate` file is found, leverages Terraform's out-of-the-box
  functionality to update the instance referenced on it
* Configure the existing or newly provisioned instance to connect to an IPA
server running on the same subnet, such that it:
  * Is remotely accessible via public key or password to centrally
    managed LDAP users
  * Is able to leverage DNS resolution and discover other private
    hosts or public addresses

After successful provisioning, you can take advantage of Terraform built-in
functionality to safely modify or delete the instance. You'll find the definition of your
instance in `main.tf`, and its current state in `terraform.tfstate`, under the user-defined
`tf_project_path` directory.

To learn the basics about managing infrastructure with Terraform, checkout the
[official documentation examples](https://developer.hashicorp.com/terraform/tutorials/aws-get-started).

## Authentication

Before proceeding, if you lack OpenStack Application Credentials or do not know
how to make them available to Ansible in your development environment, make sure
to check out [this page](https://confluence.ecmwf.int/display/EWCLOUDKB/EWC+-+How+to+request+Openstack+Application+Credentials)
and [this page](https://confluence.ecmwf.int/display/EWCLOUDKB/EWC+-+OpenStack+Command-Line+client#EWCOpenStackCommandLineclient-GettingStarted)
from EWC documentation.

Additionally, in order to configure the virtual machine after provisioning, you
required a private and public SSH keypair. Checkout this
[EWC documentation page](https://confluence.ecmwf.int/display/EWCLOUDKB/EWC+-+OpenStack+Command-Line+client#EWCOpenStackCommandLineclient-ImportSSHkey)
for details on how import your public key into OpenStack.

## Prerequisites
>💡 Versions listed correspond to minimal prerequisites.

To successfully run this playbook, the following packages should be available in your work environment:

| Name | Version | License | Home URL |
|------|---------|----- |-----|
| git | 2.0 | GPLv2  | https://git-scm.com/downloads |
| python | 3.9   | PSF | https://www.python.org/downloads  |
| ansible | 2.15 |  GPLv3+ | https://pypi.org/project/ansible  |
| terraform | 0.14  | BSL   | https://developer.hashicorp.com/terraform/install |

## Usage

### 1. Download  Ansible dependencies
>💡 By default, Ansible Roles are installed under the `~/.ansible/roles` directory within your working environment.

Download the correct version of the Ansible dependencies, if you haven't done so already:

```
ansible-galaxy role install -r requirements.yml
```

### 2. Configure and apply the template

#### 2.1. Interactive Mode

By running the following command, you can trigger an interactive session that
prompts you for the necessary user inputs, and then applies changes to your
target EWC environment:

```bash
ansible-playbook ipa-client-provisioning.yml
```

#### 2.2. Non-Interactive Mode

>💡 To learn more about defining variables at runtime, checkout the
[official Ansible documentation](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_variables.html).

You can also run in non-interactive mode by passing the
`--extra-vars` or `-e` flag, followed by a map of  key-value pairs; one for
each and every available input (see [inputs section](#inputs) below). For example:

```bash
ansible-playbook \
  -e '{
        "ewc_provider": "eumetsat",
        "tf_project_path": "~/iac/ipa-client-1",
        "app_name": "ipa",
        "instance_name": "client",
        "instance_index": 1,
        "flavor_name": "eo2.medium",
        "image_name": "ubuntu-22.04-20250204105649",
        "public_keypair_name": "john-claudy-publickey",
        "private_keypair_path": "~/.ssh/id_rsa",
        "private_network_name": "private",
        "security_group_name": "ipa",
        "instance_has_fip": "no",
        "ipa_domain": "eumetsat.sandbox.ewc",
        "ipa_server_hostname": "ipa-server-1",
        "ipa_admin_username": "iapadmin",
        "ipa_admin_password": "my-secret-password"
    }' \
  ipa-client-provisioning.yml
```
## Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|----------|
| ewc_provider | your target EWC provider. Must match that the provider of your OpenStack application credentials. Valid input values are `ecmwf` or `eumetsat`. | `string` | n/a | yes |
| tf_project_path | path to terraform working directory. Example: `~/iac/ipa-client-1` | `string` | n/a | yes |
| app_name | application name, used as prefix in the full instance name. Example: `ipa` | `string` | n/a | yes |
| instance_name| name of the instance, used in the full instance name.  Example: `client` | `string` | n/a | yes |
| instance_index | index or identifier for the instance, used as suffix in the full instance name. Example: `1` | `number` | n/a | yes |
| flavor_name | name the flavor to use for the instance. To learn about available options, checkout the [official EWC VM plans documentation](https://confluence.ecmwf.int/display/EWCLOUDKB/EWC+VM+plans) | `string` | n/a | yes |
| image_name | name of the image to use for the instance. For complete information on  available options, see the [official EWC Images documentation](https://confluence.ecmwf.int/display/EWCLOUDKB/EWC+Virtual+Images+Available).⚠️ Only Ubuntu 22.04 and RockyLinux 8.10 VM images are currently supported. This is due to constrains imposed by the required ewc-ansible-role-ipa-client-enroll Ansible Role. Example: `ubuntu-22.04-20250204105649`  | `string` | n/a | yes |
| public_keypair_name | name of public keypair (stored in OpenStack) to be copied into the instance for remote SSH access | `string` | n/a | yes |
| private_keypair_path | path to the local private keypair to use for SSH access to the instance. Example: `~/.ssh/id_rsa` | `string` | n/a | yes |
| private_network_name | private network name to attach the instance to. Example: `private` | `string` | n/a | yes |
| security_group_name | security group name to apply to the instance. Example: `ipa` | `string` | n/a | yes |
| instance_has_fip | whether to assign a floating IP to the instance. Only `yes` will be accepted to approve | `string` | n/a | yes |
| ipa_domain | domain name managed by the IPA server. Example: `eumetsta.sandbox.ewc` | `string` | n/a | yes |
| ipa_server_hostname | hostname of the IPA server. Example: `ipa-server-1` | `string`| n/a | yes |
| ipa_admin_username | username of the administrator account from the IPA server | `string` | n/a | yes |
| ipa_admin_password | password of the administrator account from the IPA server | `string` | n/a | yes |
| password_allowed_ip_ranges | IP addresses or IP ranges (in CIDR format) to be allowed for password access in SSHD configuration. When in doubt, add only IP addresses you know and trust. Example: `['10.0.0.0/24','192.168.1.0/24']` | `list(string)` | `['10.0.0.0/8','172.16.0.0/12','192.168.0.0/16']` | no |

## Dependencies
> ⚠️ Only Ubuntu 22.04 and RockyLinux 8.10 VM images are currently supported.
This is due to constrains imposed by the required
ewc-ansible-role-ipa-client-enroll Ansible Role.

| Name | Version | License | Home URL |
|------|---------|-------|------|
| ewc-tf-module-openstack-compute | 1.3 | MIT | https://github.com/ewcloud/ewc-tf-module-openstack-compute  |
| ewc-ansible-role-ipa-client-enroll | 1.0 | MIT | https://github.com/ewcloud/ewc-ansible-role-ipa-client-enroll |


## Troubleshooting
Checkout the [troubleshooting documentation](../docs/troubleshooting.md) for
information on common problems and how to troubleshoot them.
