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

* Verify the `ssh-https` OpenStack Security Group exists in your EWC tenancy
  > 💡 You may create Security Groups via [this EWC Community Hub Item](https://europeanweather.cloud/community-hub/openstack-compute-instance) if pre-required ones are missing.
* If you plan to configure an existing VM, ensure it meets the minium requirements before proceeding:
  * VM Image: Ubuntu 24 or 22
  * VM Plan: 4 CPU cores, 8GB RAM, 40GB Disk
  * Network: Private
  * Security Groups: `ssh-https`
  * Floating IP: Required
  
  Otherwise, provision a new VM with above specifications before continuing (see [EWC Getting Started: Provision a VM](https://confluence.ecmwf.int/x/2RvEJg) for details).


## Usage

#### 2. Clone the repository

```bash
git clone https://github.com/ewcloud/ewc-ansible-playbook-flavours-and-provisioning.git
```

##### 2.1. Change to the specific Item's subdirectory

```bash
cd ewc-ansible-playbook-flavours-and-provisioning/playbooks/haproxy-flavour
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
Create an inventory file, to specify address/credentials that your local working environment should use
to connect to the target VM.

Copy into the file one of the two snippets below, and replace the placeholders (i.e. values enclosed in `<` `>` characters):

* **To connect within the EWC private network**

  ```yaml
  # inventory.yml
  ---
  ewcloud:
    hosts:
      haproxy:
        ansible_python_interpreter: /usr/bin/python3
        ansible_host: <add the PRIVATE IP address of the target host>
        ansible_ssh_private_key_file: <add the path to local SSH private key file>
        ansible_user: ubuntu
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
        ansible_python_interpreter: /usr/bin/python3
        ansible_host: <add the PUBLIC IP address of the target host>
        ansible_ssh_private_key_file: <add the path to local SSH private key file>
        ansible_user: ubuntu
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
          ansible_ssh_user: ubuntu
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

By running the following command, you can trigger an interactive session that
prompts you for the necessary user inputs, and then applies changes to your
target EWC environment:

```bash
ansible-playbook -i inventory.yml haproxy-flavour.yml
```

## Dependencies

> 💡 Upon execution, a SBOM (SPDX format) is auto-generated and stored in the VM's file system root directory (see `/sbom.json`).

| Name | Home URL |
|------|---------|
| ewc-ansible-role-haproxy | https://github.com/ewcloud/ewc-ansible-role-haproxy |

## Operation
Checkout the following how-to guides to learn about management of the Item after initial setup:
* [How to add a new backend server to HAProxy](./docs/how-to/how-to-add-a-new-backend-server-to-haproxy.md)
* [How to add a new SSL certificate to a server in HAProxy](./docs/how-to/how-to-add-a-new-ssl-certifacte-to-server-in-haproxy.md)
* [How to modify HAproxy config file](./docs/how-to/how-to-modify-haproxy-config-file.md)