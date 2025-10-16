# HAProxy Flavour

This Ansible Playbook configures an existing virtual machine running
within the [European Weather Cloud (EWC)](https://europeanweather.cloud/), to operate as a [HAProxy](https://www.haproxy.org/) server.

HAProxy is a high-performance, open source load balancer and reverse proxy for TCP and Hypertext Transfer Protocol (HTTP) applications.
Users can leverage HAProxy to distribute workloads and improve website and application performance.

## Functionality

* Layer 4 TCP and Layer 7 HTTP load balancing.
* Protocol support for HTTP, HTTP/2, gRPC and FastCGI.
* Secure Sockets Layer (SSL) and Transport Layer Security termination.
* Dynamic SSL certificate storage.
* Content switching and inspection.
* Transparent proxying.
* Detailed logging.
* Command-line interface (CLI) for server management.
* HTTP authentication.
* Multithreading.
* URL rewrites.
* Health checking.
* Rate limits.

## Prerequisites

* Install [git](https://git-scm.com/downloads) (version 2.0 or higher )
* Install [python](https://www.python.org/downloads) (version 3.9 or higher) 
* Install [ansible](https://pypi.org/project/ansible) (version 2.15 or higher)
* Get OpenStack API credentials (see [How to request OpenStack Application Credentials](https://confluence.ecmwf.int/display/EWCLOUDKB/EWC+-+How+to+request+Openstack+Application+Credentials) section of the EWC documentation)
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

## Usage

### 1. Clone the repository

```bash
git clone https://github.com/ewcloud/ewc-ansible-playbook-flavours-and-provisioning.git
```

#### 1.1. Change to the specific Item's subdirectory

```bash
cd ewc-ansible-playbook-flavours-and-provisioning/playbooks/haproxy-flavour
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

### 3. Specify the target host and SSH credentials
Create an inventory file to specify address/credentials that Ansible should use
to reach the virtual machine you wish to configure:

```yaml
# inventory.yml
---
ewcloud:
  hosts:
    haproxy:
      ansible_python_interpreter: /usr/bin/python3
      ansible_host: <add the IPV4 address of the target host>
      ansible_ssh_private_key_file: <add the path to local SSH private key file>
      ansible_user: ubuntu
      ansible_ssh_common_args: -o StrictHostKeyChecking=accept-new
```

### 4. Configure and apply the template

#### 4.1. Interactive Mode

By running the following command, you can trigger an interactive session that
prompts you for the necessary user inputs, and then applies changes to your
target EWC environment:

```bash
ansible-playbook -i inventory.yml haproxy-flavour.yml
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
  -e "os_security_group_name_fact=ssh-http-https" \
  haproxy-flavour.yml
```

## Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|:--------:|
| os_security_group_name_fact | OpenStack security group containing all firewall rules required for HAProxy operation | `string` | `ssh-http-https` | yes |

## Dependencies
> ⚠️ Only Ubuntu 22.04 images are currently supported.
This is due to constrains imposed by the required ewc-ansible-role-haproxy
Ansible Role.

> 💡 A VM plan with at least 8GB of RAM is recommended for successful setup and
stable operation.

| Name | Version | License | Home URL |
|------|---------|------|------|
| ewc-ansible-role-haproxy | 1.0 |  MIT | https://github.com/ewcloud/ewc-ansible-role-haproxy |

## Operation
Checkout the following how-to guides to learn about management of the Item after initial setup:
* [How to add a new backend server to HAProxy](./docs/how-to/how-to-add-a-new-backend-server-to-haproxy.md)
* [How to add a new SSL certificate to a server in HAProxy](./docs/how-to/how-to-add-a-new-ssl-certifacte-to-server-in-haproxy.md)
* [How to modify HAproxy config file](./docs/how-to/how-to-modify-haproxy-config-file.md)