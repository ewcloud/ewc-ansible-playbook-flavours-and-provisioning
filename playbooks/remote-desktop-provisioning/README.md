# Remote desktop provisioning
>✅ This template can be safely applied from any local work environment, even running outside an EWC tenancy's private network.

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

After successful provisioning, you can leverage Terraform's functionality to modify or delete individual components safely. Each will have its own `main.tf` definition and `terraform.tfstate` state file under the corresponding user-defined local directories.

To learn the basics about managing infrastructure with Terraform, check out [Terraform in 100 seconds](https://youtu.be/tomUWcQ0P3k?si=CJwZJ7UaqpynDU-d) on YouTube. You can also find a step-by-step example applied to the EWC on the [official EWC documentation](https://confluence.ecmwf.int/x/2EDOIQ).


>💡 This template can be deployed in combination with complementary infrastructure as part of the [Default Stack Provisioning](https://europeanweather.cloud/community-hub/default-stack-provisioning) Community Hub Item.

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
ansible-playbook remote-desktop-provisioning.yml
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
        "ewc_provider":"eumetsat",
        "remote_desktop_tf_project_path":"~/ewc/remote-desktop-1",
        "remote_desktop_app_name":"remote",
        "remote_desktop_instance_name":"desktop",
        "remote_desktop_instance_index":1,
        "remote_desktop_flavor_name":"eo1.large",
        "remote_desktop_image_name":"Rocky-9.5-20250604142417",
        "remote_desktop_instance_has_fip":"no",
        "public_keypair_name":"my-public-key-name",
        "private_keypair_path":"~/.ssh/id_rsa",
        "private_network_name":"private",
        "security_group_name":"ssh",
        "fail2ban_whitelisted_ip_ranges":""
    }' \
  remote-desktop-provisioning.yml
```

### 3. Install the local client and connect to your remote desktop
>⚠️ When configuring a connection, be sure to select "MATE" (instead of
"KDE" or any other options) in the `Session Type` drop-down list, towards the
bottom of the `Session` tab. This is required for the local client to correctly
communicate with your remote desktop.

Install the remote desktop client on Microsoft Window, Mac OS or Linux by
following the links on the [official X2Go installation page](https://wiki.x2go.org/doku.php/doc:installation:x2goclient). Then follow the [official X2Go client usage page](https://wiki.x2go.org/doku.php/doc:usage:x2goclient)
if you do not know how to configure a new session.

For a session creation
example, representative of a typical EWC environment, checkout the Remote
Desktop section of
[this official EWC documentation page](https://confluence.ecmwf.int/display/EWCLOUDKB/EUMETSAT+tenancy%3A+Default+setup).


## Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|----------|
| ewc_provider | your target EWC provider. Must match that the provider of your OpenStack application credentials. Valid input values are `ecmwf` or `eumetsat`. | `string` | `eumetsat` | yes |
| remote_desktop_tf_project_path | path to terraform working directory | `string` | `~/ewc/remote-desktop-1` | yes |
| remote_desktop_app_name | application name, used as prefix in the full instance name | `string` | `remote` | yes |
| remote_desktop_instance_name| name of the instance, used in the full instance name | `string` | `desktop` | yes |
| remote_desktop_instance_index | index or identifier for the instance, used as suffix in the full instance name | `number` | `1` | yes |
| remote_desktop_flavor_name | name the flavor to use for the instance. To learn about available options, checkout the [official EWC VM plans documentation](https://confluence.ecmwf.int/display/EWCLOUDKB/EWC+VM+plans). 💡 A VM plan with at least 4GB of RAM is recommended for successful setup and stable operation. | `string` | `eo1.large` | yes |
| remote_desktop_image_name | name of the image to use for the instance. For complete information on  available options, see the [official EWC Images documentation](https://confluence.ecmwf.int/display/EWCLOUDKB/EWC+Virtual+Images+Available). ⚠️ Only RockyLinux 8.10 and 9.5 instances are currently supported due to constrains imposed by the required ewc-ansible-role-remote-desktop Ansible Role | `string` | `Rocky-9.5-20250604142417` | yes |
| remote_desktop_instance_has_fip | whether to assign a floating IP to the instance. Only `yes` will be accepted to approve | `string` | `no` | no |
| public_keypair_name | name of public keypair (stored in OpenStack) to be copied into the instance for remote SSH access | `string` | n/a | yes |
| private_keypair_path| path to the local private keypair to use for SSH access to the instance | `string` | `~/.ssh/id_rsa` | yes |
| private_networks_name | private network name to attach the instance to  | `string` | `private` | yes |
| security_group_name | security group name to apply to the instance | `string` | `ipa` | yes |
| fail2ban_whitelisted_ip_ranges | IPv4 ranges (in CIDR format) to be whitelisted in Fail2ban configuration. When in doubt, do not set. Example: `['10.0.0.0/24','192.168.1.0/24']` | `list(string)` | `''` | no |

## Dependencies
> ⚠️ Only RockyLinux 8.10 and 9.5 instances are currently supported due
to constrains imposed by the required ewc-ansible-role-remote-desktop Ansible
Role.

> 💡 A VM plan with at least 4GB of RAM is recommended for successful setup and
stable operation.

| Name | Version | License | Home URL |
|------|---------|-------|------|
| ewc-tf-module-openstack-compute | 1.4 | MIT | https://github.com/ewcloud/ewc-tf-module-openstack-compute  |
| ewc-ansible-role-remote-desktop | 1.2 | MIT | https://github.com/ewcloud/ewc-ansible-role-remote-desktop |


## Troubleshooting
Checkout the [troubleshooting documentation](../../docs/troubleshooting.md) for
information on common problems and how to troubleshoot them.
