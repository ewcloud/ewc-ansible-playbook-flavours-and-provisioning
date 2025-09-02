# Remote desktop provisioning
The remote desktop is a regular RockyLinux instance equipped with [X2Go](https://wiki.x2go.org/doku.php).
It enables you to access a graphical desktop computer running in your remote
instance of choice, over a low bandwidth (or high bandwidth) connection.
This means that you can connect to it via the
[X2Go client](https://wiki.x2go.org/doku.php/doc:installation:x2goclient)
to enjoy a regular desktop user experience.

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
* Configure the existing or newly provisioned instance such that it:
  * Enables users to operate the remote hosts through a graphical desktop
    (i.e. a [MATE desktop environment](https://mate-desktop.org/)), over a low or high bandwidth connection.

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

## Usage

### 1. Configure and apply the template

#### 1.1. Interactive Mode

By running the following command, you can trigger an interactive session that
prompts you for the necessary user inputs, and then applies changes to your
target EWC environment:

```bash
ansible-playbook remote-desktop-provisioning.yml
```

#### 1.2. Non-Interactive Mode

>💡 To learn more about defining variables at runtime, checkout the
[official Ansible documentation](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_variables.html).

You can also run in non-interactive mode by passing the
`--extra-vars` or `-e` flag, followed by a map of  key-value pairs; one for
each and every available input (see [inputs section](#inputs) below). For example:

```bash
ansible-playbook \
  -e '{
        "ewc_provider": "eumetsat",
        "tf_project_path":"~/iac/x2go-server-1",
        "app_name":"x2go",
        "instance_name":"server",
        "instance_index": 1,
        "flavor_name":"eo2.medium",
        "image_name":"Rocky-8.10-20250204105303",
        "public_keypair_name":"john-claudy-publickey",
        "private_keypair_path":"~/.ssh/id_rsa",
        "private_network_name": "private",
        "security_group_name": "ssh"
    }' \
  remote-desktop-provisioning.yml
```
## Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|----------|
| ewc_provider | your target EWC provider. Must match that the provider of your OpenStack application credentials. Valid input values are `ecmwf` or `eumetsat`. | `string` | n/a | yes |
| tf_project_path | path to terraform working directory. Example: `~/iac/x2go-server-1` | `string` | n/a | yes |
| app_name | application name, used as prefix in the full instance name. Example: `x2go` | `string` | n/a | yes |
| instance_name| name of the instance, used in the full instance name.  Example: `server` | `string` | n/a | yes |
| instance_index | index or identifier for the instance, used as suffix in the full instance name. Example: `1` | `number` | n/a | yes |
| flavor_name | name the flavor to use for the instance. To learn about available options, checkout the [official EWC VM plans documentation](https://confluence.ecmwf.int/display/EWCLOUDKB/EWC+VM+plans). 💡 A VM plan with at least 4GB of RAM is recommended for successful setup and stable operation. | `string` | n/a | yes |
| image_name | name of the image to use for the instance. For complete information on  available options, see the [official EWC Images documentation](https://confluence.ecmwf.int/display/EWCLOUDKB/EWC+Virtual+Images+Available). ⚠️ Only RockyLinux 8.10 instances are currently supported due to constrains imposed by the required ewc-ansible-role-remote-desktop Ansible Role. Example: `Rocky-8.10-20250204105303`  | `string` | n/a | yes |
| public_keypair_name | name of public keypair (stored in OpenStack) to be copied into the instance for remote SSH access | `string` | n/a | yes |
| private_keypair_path| path to the local private keypair to use for SSH access to the instance. Example: `~/.ssh/id_rsa` | `string` | n/a | yes |
| private_networks_name | private network name to attach the instance to. Example: `private` | `string` | n/a | yes |
| security_group_name | security group name to apply to the instance. Example: `ipa` | `string` | n/a | yes |
| whitelisted_ip_ranges | IPv4 ranges (in CIDR format) to be whitelisted in Fail2ban configuration. When in doubt, do not set. Example: `['10.0.0.0/24','192.168.1.0/24']` | `list(string)` | n/a | no |

## Dependencies
> ⚠️ Only RockyLinux 8.10 instances are currently supported due
to constrains imposed by the required ewc-ansible-role-remote-desktop Ansible
Role.

> 💡 A VM plan with at least 4GB of RAM is recommended for successful setup and
stable operation.

| Name | Version | License | Package Info |
|------|---------|-------|------|
| ewc-tf-module-openstack-compute | 1.3 | MIT | https://github.com/ewcloud/ewc-tf-module-openstack-compute  |
| ewc-ansible-role-remote-desktop | 1.0 | MIT | https://github.com/ewcloud/ewc-ansible-role-remote-desktop |


## Troubleshooting
Checkout the [troubleshooting documentation](../../docs/troubleshooting.md) for
information on common problems and how to troubleshoot them.
