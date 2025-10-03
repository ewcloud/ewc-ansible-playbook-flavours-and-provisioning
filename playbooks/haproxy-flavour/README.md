# HAProxy flavour

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
>💡 Versions listed correspond to minimal prerequisites.

To successfully run this playbook, the following packages should be available in your work environment:

| Name | Version | License | Home URL |
|------|---------|----- |-----|
| git | 2.0 | GPLv2  | https://git-scm.com/downloads |
| python | 3.9   | PSF | https://www.python.org/downloads  |
| ansible | 2.15 |  GPLv3+ | https://pypi.org/project/ansible  |

## Usage

### 1. Download  Ansible dependencies
>💡 By default, Ansible Roles are installed under the `~/.ansible/roles` directory within your working environment.

Download the correct version of the Ansible dependencies, if you haven't done so already:

```
ansible-galaxy role install -r requirements.yml
```

### 2. Specify the target host and SSH credentials
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

### 3. Configure and apply the template

#### 3.1. Interactive Mode

By running the following command, you can trigger an interactive session that
prompts you for the necessary user inputs, and then applies changes to your
target EWC environment:

```bash
ansible-playbook -i inventory.yml haproxy-flavour.yml
```

#### 3.2. Non-Interactive Mode

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
