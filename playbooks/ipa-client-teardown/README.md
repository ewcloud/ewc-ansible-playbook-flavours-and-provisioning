# IPA Client Teardown
>✅ This template can be safely applied from any local work environment, even running outside an EWC tenancy's private network.

This is a configuration template
(i.e. an [Ansible Playbook](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks.html))
to customize your environment in the
[European Weather Cloud (EWC)](https://europeanweather.cloud/).

## Functionality

The template is designed to run on an existing virtual machine, running an
[IPA client](https://www.freeipa.org/page/Client) previously enrolled in
your IPA server, such that it:

* Checks if a `terraform.tfstate`[state file](https://developer.hashicorp.com/terraform/language/state)
  for the target instance is available under the user-defined directory
* Requests configuration changes to said IPA server for:
  * Stopping user authentication/authorization management (LDAP) to target
  instance
  * Deletion of IPA server-internal DNS records referencing  the target
  instance machine, if and when found
* Teardown the target instance and any attached volumes or IP addresses.

After successful provisioning, you can leverage Terraform's functionality to modify or delete individual components safely. Each will have its own `main.tf` definition and `terraform.tfstate` state file under the corresponding user-defined local directories.

To learn the basics about managing infrastructure with Terraform, check out [Terraform in 100 seconds](https://youtu.be/tomUWcQ0P3k?si=CJwZJ7UaqpynDU-d) on YouTube. You can also find a step-by-step example applied to the EWC on the [official EWC documentation](https://confluence.ecmwf.int/x/2EDOIQ).

## Prerequisites

* Install [git](https://git-scm.com/downloads) (version 2.0 or higher )
* Install [python](https://www.python.org/downloads) (version 3.9 or higher) 
* Install [python-openstackclient](https://pypi.org/project/python-openstackclient) (version 8.0 or higher)
* Install [ansible](https://pypi.org/project/ansible) (version 2.15 or higher)
* Install [terraform](https://confluence.ecmwf.int/display/EWCLOUDKB/EWC+-+IaC+via+Terraform+and+OpenTofu#EWCIaCviaTerraformandOpenTofu-InstallationoftheCLI) (version 1.0 or higher)
* Get OpenStack API credentials (see [How to request OpenStack Application Credentials](https://confluence.ecmwf.int/display/EWCLOUDKB/EWC+-+How+to+request+Openstack+Application+Credentials) section of the EWC documentation)
* You require an existing VM already enrolled into your IPA Server and provisioned via any of the EWC Community Hub Items with the name suffix `Provisioning` (i.e. [IPA Client Provisioning](https://europeanweather.cloud/community-hub/ipa-client-provisioning), [Default Stack Provisioning](https://europeanweather.cloud/community-hub/default-stack-provisioning/), etc.)
* Your require SSH access to the exiting VM (i.e. your public SSH key must be registered on the target machine)

## Usage

### 1. Clone the repository

```bash
git clone https://github.com/ewcloud/ewc-ansible-playbook-flavours-and-provisioning.git
```

#### 1.1. Change to the specific Item's subdirectory

```bash
cd ewc-ansible-playbook-flavours-and-provisioning/playbooks/ipa-client-teardown
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
ansible-playbook ipa-client-teardown.yml
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
        "tf_project_path": "~/ewc/ipa-client-1",
        "private_keypair_path": "~/.ssh/id_rsa",
        "ipa_domain": "eumetsat.sandbox.ewc",
        "ipa_server_hostname": "ipa-server-1",
        "ipa_admin_username": "ipaadmin",
        "ipa_admin_password": "my-secret-password"
    }' \
  ipa-client-teardown.yml
```
## Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|----------|
| tf_project_path | path to terraform working directory. Example: `~/ewc/ipa-client-1` | `string` | n/a | yes |
| private_keypair_path | path to the local private keypair to use for SSH access to the instance | `string` | `~/.ssh/id_rsa` | yes |
| ipa_domain | domain name managed by the IPA server. Example: `eumetsat.sandbox.ewc` | `string` | n/a | yes |
| ipa_server_hostname | hostname of the IPA server. | `string`| `ipa-server-1` | yes |
| ipa_admin_username | username of the administrator account from the IPA server | `string` | `ipaadmin` | yes |
| ipa_admin_password | password of the administrator account from the IPA server | `string` | n/a | yes |


## Dependencies

| Name | Home URL |
|------|---------|
| ewc-tf-module-openstack-compute | https://github.com/ewcloud/ewc-tf-module-openstack-compute  |
| ewc-ansible-role-ipa-client-disenroll | https://github.com/ewcloud/ewc-ansible-role-ipa-client-disenroll |


## Troubleshooting
Checkout the [troubleshooting documentation](../docs/troubleshooting.md) for
information on common problems and how to troubleshoot them.
