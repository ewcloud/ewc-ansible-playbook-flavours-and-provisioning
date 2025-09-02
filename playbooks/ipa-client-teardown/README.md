# IPA client teardown

This is a configuration template
(i.e. an [Ansible Playbook](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks.html))
to customize your environment in the
[European Weather Cloud (EWC)](https://europeanweather.cloud/).

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

After successful teardown, you can take advantage of Terraform built-in
functionality to safely re-provision the instance from scratch.

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
ansible-playbook ipa-client-teardown.yml
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
        "tf_project_path": "~/iac/ipa-client-1",
        "private_keypair_path": "~/.ssh/id_rsa",
        "ipa_domain": "eumetsat.sandbox.ewc",
        "ipa_server_hostname": "ipa-server-1",
        "ipa_admin_username": "iapadmin",
        "ipa_admin_password": "my-secret-password"
    }' \
  ipa-client-teardown.yml
```
## Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|----------|
| tf_project_path | path to terraform working directory. Example: `~/iac/ipa-client-1` | `string` | n/a | yes |
| private_keypair_path | path to the local private keypair to use for SSH access to the instance. Example: `~/.ssh/id_rsa` | `string` | n/a | yes |
| ipa_domain | domain name managed by the IPA server. Example: `eumetsat.sandbox.ewc` | `string` | n/a | yes |
| ipa_server_hostname | hostname of the IPA server. Example: `ipa-server-1` | `string`| n/a | yes |
| ipa_admin_username | username of the administrator account from the IPA server | `string` | n/a | yes |
| ipa_admin_password | password of the administrator account from the IPA server | `string` | n/a | yes |


## Dependencies

| Name | Version | License | Package Info |
|------|---------|-------|-----|
| ewc-tf-module-openstack-compute | 1.3 | MIT | https://github.com/ewcloud/ewc-tf-module-openstack-compute  |
| ewc-ansible-role-ipa-client-disenroll | 1.0 | MIT |  https://github.com/ewcloud/ewc-ansible-role-ipa-client-disenroll |


## Troubleshooting
Checkout the [troubleshooting documentation](../docs/troubleshooting.md) for
information on common problems and how to troubleshoot them.
