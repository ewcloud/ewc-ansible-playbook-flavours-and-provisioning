# IPA Server Provisioning
>✅ This template can be safely applied from any local work environment, even running outside an EWC tenancy's private network.

IPA (acronym for identity, policy and audit) and its open-source
implementation [FreeIPA](https://www.freeipa.org/page/Main_Page), serve
both as a user management system and as your internal DNS nameserver.

This is a configuration template
(i.e. an [Ansible Playbook](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks.html))
to customize your environment in the
[European Weather Cloud (EWC)](https://europeanweather.cloud/).

## Functionality

The template is designed to:

* Provision an instance via [Terraform](https://developer.hashicorp.com/terraform),
with your specified VM image and desired flavor (a.k.a VM plan):
  * If a `terraform.tfstate` [state file](https://developer.hashicorp.com/terraform/language/state)
  is not found under the user-defined directory, attempts to create the
  instance from scratch

    OR
  * If a `terraform.tfstate` file is found, leverages Terraform's out-of-the-box
  functionality to update the instance referenced on it
* Validate that network/subnet configuration in the EWC tenancy
* Configure the existing or newly provisioned instance such that it:
  * Provides DNS resolutions for discovery of resources (i.e. other virtual machines)
  * Enables centralized user and credentials creation/edition/deletion/authentication
  * Allows centralized authorization between users and resources
* Automatically update the underlying subnet DNS nameserver to point to the
newly configured IPA server

After successful provisioning, you can leverage Terraform's functionality to modify or delete individual components safely. Each will have its own `main.tf` definition and `terraform.tfstate` state file under the corresponding user-defined local directories.

To learn the basics about managing infrastructure with Terraform, check out [Terraform in 100 seconds](https://youtu.be/tomUWcQ0P3k?si=CJwZJ7UaqpynDU-d) on YouTube. You can also find a step-by-step example applied to the EWC on the [official EWC documentation](https://confluence.ecmwf.int/x/2EDOIQ).

>⚠️ Successful execution leads to changes of the DNS nameserver(s) in your
OpenStack subnet (includes now only the IP address of the new IPA server).
This can negatively affect existing VMs within your subnet.
To prevent issues, programmatically update each VM via the
[IPA Client Enroll Flavour](https://europeanweather.cloud/community-hub/ipa-client-enroll-flavour)
CommunityHub Item. Alternatively, you can manually
[add the new nameserver](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/6/html/identity_management_guide/domain-dns)
to their DNS configuration.

>💡 This template can be deployed in combination with complementary infrastructure as part of the [Default Stack Provisioning](https://europeanweather.cloud/community-hub/default-stack-provisioning) Community Hub Item.

## Prerequisites

* Install [git](https://git-scm.com/downloads) (version 2.0 or higher )
* Install [python](https://www.python.org/downloads) (version 3.9 or higher) 
* Install [ansible](https://pypi.org/project/ansible) (version 2.15 or higher)
* Install [terraform](https://confluence.ecmwf.int/display/EWCLOUDKB/EWC+-+IaC+via+Terraform+and+OpenTofu#EWCIaCviaTerraformandOpenTofu-InstallationoftheCLI) (version 1.0 or higher)
* Get OpenStack API credentials (see [How to request OpenStack Application Credentials](https://confluence.ecmwf.int/display/EWCLOUDKB/EWC+-+How+to+request+Openstack+Application+Credentials) section of the EWC documentation)
* Create an SSH keypair (see [Creating Keys](https://confluence.ecmwf.int/display/EWCLOUDKB/Add+your+SSH+key+pair+to+Morpheus#AddyourSSHkeypairtoMorpheus-Creatingthekeys) section of the EWC documentation)
* Import your public SSH key to OpenStack (see [Import SSH Key](https://confluence.ecmwf.int/display/EWCLOUDKB/EWC+-+OpenStack+Command-Line+client#EWCOpenStackCommandLineclient-ImportSSHkey) section of the EWC documentation).

## Usage

### 1. Clone the repository

```bash
git clone https://github.com/ewcloud/ewc-ansible-playbook-flavours-and-provisioning.git
```

#### 1.1. Change to the specific Item's subdirectory

```bash
cd ewc-ansible-playbook-flavours-and-provisioning/playbooks/ipa-server-provisioning
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
ansible-playbook ipa-server-provisioning.yml
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
        "ewc_provider": "eumetsat",
        "ipa_server_tf_project_path":"~/ewc/ipa-server-1",
        "ipa_server_app_name":"ipa",
        "ipa_server_instance_name":"server",
        "ipa_server_instance_index": 1,
        "ipa_server_flavor_name":"eo1.large",
        "ipa_server_image_name":"Rocky-8.10-20250604144456",
        "public_keypair_name":"my-public-key-name",
        "private_keypair_path":"~/.ssh/id_rsa",
        "private_network_name": "private",
        "security_group_name": "ipa",
        "ipa_domain":"eumetsat.sandbox.ewc",
        "ipa_admin_username":"ipaadmin",
        "ipa_admin_password":"my-secret-password",
        "ipa_admin_givenname": "IPAADMIN",
        "ipa_admin_surname": "EWC"
    }' \
  ipa-server-provisioning.yml
```
## Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|----------|
| ewc_provider | your target EWC provider. Must match that the provider of your OpenStack application credentials. Valid input values are `ecmwf` or `eumetsat`. | `string` | `eumetsat` | yes |
| ipa_server_tf_project_path | path to terraform working directory | `string` | `~/ewc/ipa-server-1` | yes |
| ipa_server_app_name | application name, used as prefix in the full instance name | `string` | `ipa` | yes |
| ipa_server_instance_name| name of the instance, used in the full instance name | `string` | `server` | yes |
| ipa_server_instance_index | index or identifier for the instance, used as suffix in the full instance name | `number` | `1` | yes |
| ipa_server_flavor_name | name the flavor to use for the instance. To learn about available options, checkout the [official EWC VM plans documentation](https://confluence.ecmwf.int/display/EWCLOUDKB/EWC+VM+plans). 💡 A VM plan with at least 4GB of RAM is recommended for successful setup and stable operation. | `string` | `eo1.large` | yes |
| ipa_server_image_name | name of the image to use for the instance. For complete information on  available options, see the [official EWC Images documentation](https://confluence.ecmwf.int/display/EWCLOUDKB/EWC+Virtual+Images+Available). ⚠️ Only RockyLinux 9.5 and RockyLinux 8.10 VM images are currently supported. This is due to constrains imposed by the required ewc-ansible-role-ipa-server Ansible Role | `string` | `Rocky-8.10-20250604144456` | yes |
| public_keypair_name | name of public keypair (stored in OpenStack) to be copied into the instance for remote SSH access | `string` | n/a | yes |
| private_keypair_path | path to the local private keypair to use for SSH access to the instance  | `string` | `~/.ssh/id_rsa` | yes |
| private_network_name | private network name to attach the instance | `string` | `private`  | yes |
| security_group_name | security group name to apply to the instance  | `string` | `ipa` | yes |
| ipa_domain | domain name to be managed by the IPA server. Example: `eumetsat.sandbox.ewc` | `string` | n/a | yes |
| ipa_admin_username | username of administrator account to replace the default IPA admin | `string` | `ipaadmin` | yes |
| ipa_admin_password | password of administrator account to replace the default IPA admin | `string` | n/a | yes |
| ipa_admin_givenname | given name of the administrator to replace the default IPA admin (needs not be a physical person) | `string` | `EWC` | yes |
| ipa_admin_surname | surname of the administrator to replace the default IPA admin (needs not to belong to a physical person)  | `string` | `IPAADMIN` | yes |


## Dependencies
> ⚠️ Only RockyLinux 9.5 and RockyLinux 8.10 VM images are currently supported.
This is due to constrains imposed by the required ewc-ansible-role-ipa-server
Ansible Role.

> 💡 A VM plan with at least 4GB of RAM is recommended for successful setup and
stable operation.

| Name | Version | License | Home URL |
|------|---------|-------|------|
| ewc-tf-module-openstack-compute | 1.4 | MIT | https://github.com/ewcloud/ewc-tf-module-openstack-compute  |
| ewc-ansible-role-ipa-server | 1.0 | MIT | https://github.com/ewcloud/ewc-ansible-role-ipa-server |


## Operation
Checkout the following how-to guides to learn about management of the Item after initial setup:
* [How to configure the IPA Server](../ipa-server-flavour/docs/how-to/how-to-configure-the-ipa-server.md)

## Troubleshooting
Checkout the [troubleshooting documentation](../../docs/troubleshooting.md) for
information on common problems and how to troubleshoot them.
