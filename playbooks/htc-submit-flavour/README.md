# HTC Batch Processing Submit Flavour

Many computational tasks in Earth Observation and environmental modelling apply the same processing algorithm independently across thousands or millions of input data items. These parallel tasks are best served by High Throughput Computing (HTC).

The [European Weather Cloud (EWC)](https://europeanweather.cloud/) provides a centrally-managed [EWC HTC Batch Processing](https://confluence.ecmwf.int/x/jRVkFQ) service based on [HTCondor](https://htcondor.readthedocs.io/en/latest/users-manual/quick-start-guide.html), a specialized workload-management system designed for compute-intensive jobs.

The principal advantage of this EWC service is access to a much larger, resource pool than any single tenant could provide alone. Capacity is drawn from:

1. Capacity made available by EWC itself
2. Spare VMs contributed by EWC tenants for shared usage (i.e. those configured by the [HTC Batch Processing Execute Flavour](../htc-execute-flavour/))

EWC users can submit jobs from their VMs. Jobs are placed in the queue, matched to available resources, executed, and results are returned to VM where the job was submitted from.

This template is EWC users wishing to configure their VMs as HTCondor Submit Nodes:

![High Throughput Batch Processing Overview](https://raw.githubusercontent.com/ewcloud/ewc-ansible-playbook-flavours-and-provisioning/refs/heads/main/playbooks/htc-submit-flavour/docs/images/htc-submit-flavour.png)


## Functionality
The template is designed to:

* Configure a pre-existing virtual machine running RockyLinux, with private IP
address, and a minimum recommended 16GB of RAM, as:
  * an HTCondor submit node, capable of dispatching jobs to any nodes enrolled into the EWC centrally-managed pool of shared compute resources

## Prerequisites

> 💡 This Item is supported by the [EWCCLI](https://www.europeanweather.cloud/community-hub/ewc-cli), 
and can be deployed, together with a compatible VM, via it. Checkout the [EWC User Stories: I want to use the ewccli](https://confluence.ecmwf.int/x/NlYiK) documentation pages to learn how.

* Request authorization keys:
  * Write to EWC Support ([support@europeanweather.cloud](mailto:support@europeanweather.cloud)), making sure to provide your EWC tenancy name alongside your request for VPN and HTCondor access.

* Install [git](https://git-scm.com/downloads) (version 2.0 or higher )
* Install [python](https://www.python.org/downloads) (version 3.9 or higher) 
* Install [ansible](https://pypi.org/project/ansible) (version 2.15 or higher)

* Verify the `htcondor` OpenStack Security Group exists in your EWC tenancy
  > 💡 You may create Security Groups via [this EWC Community Hub Item](https://europeanweather.cloud/community-hub/openstack-compute-instance) if pre-required ones are missing.
* If you plan to configure an existing VM, ensure it meets the minium requirements before moving on to the [Usage](#usage) section below:
  * VM Image: RockyLinux 8 or 9
  * VM Plan: 8 CPU cores, 32GB RAM, 30GB Disk
  * Network: Private
  * Security Groups: `htcondor`
  * Floating IP: Optional (not recommended from a security standpoint)
  
  Otherwise, provision a new VM with above specifications before continuing (see [EWC Getting Started: Provision a VM](https://confluence.ecmwf.int/x/2RvEJg) for details).


## Usage

### 1. Clone the repository

```bash
git clone https://github.com/ewcloud/ewc-ansible-playbook-flavours-and-provisioning.git
```

#### 1.1. Change to the specific Item's subdirectory

```bash
cd playbooks/htc-submit-flavour
```

#### 1.2. Checkout an specific Item's version
>⚠️ Make sure to replace `x.y.z` in the command below, with your version of preference.

```bash
git checkout x.y.z
```


### 2. Download Ansible dependencies
>💡 By default, Ansible Roles are installed under the `~/.ansible/roles` directory within your working environment.

Download the correct version of the Ansible dependencies, if you haven't done so already:

```
ansible-galaxy role install --force -r requirements.yml
```

### 3. Specify the target host and SSH credentials
Create an inventory file, to specify address/credentials that your local working environment should use
to connect to the target VM.

Copy into the file one of the two snippets below, and replace the placeholders (i.e. values enclosed in `<` `>` characters):

* **Connecting form within the EWC tenancy's network**

  ```yaml
  # inventory.yml
  ---
  ewcloud:
    hosts:
      target:
        ansible_python_interpreter: /usr/bin/python3
        ansible_host: <add the IPV4 address of the target host>
        ansible_ssh_private_key_file: <add the path to local SSH private key file>
        ansible_user: cloud-user
        ansible_ssh_common_args: -o StrictHostKeyChecking=no
  ```

**OR**


*  **Connecting from outside the EWC tenancy's network**

    > ⚠️ This requires an [SSH Bastion](https://europeanweather.cloud/community-hub/ssh-bastion-provisioning) to be already provisioned within your EWC tenancy.

    ```yaml
    # inventory.yml
    ---
    ewcloud:
      hosts:
        target:
          ansible_host: <add the IP address of the target host>
          ansible_ssh_user: cloud-user
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
                          cloud-user@<add the IP address of the ssh bastion>"

    ```

### 4. Configure and apply the template

* **Interactive Mode**

  By running the following command, you can trigger an interactive session that
  prompts you for the necessary user inputs, and then applies changes to your
  target EWC environment:

  ```bash
  ansible-playbook -i inventory.yml htc-submit-flavour.yml
  ```

**OR**


* **Non-Interactive Mode**

  >💡 To learn more about defining variables at runtime, checkout the
  [official Ansible documentation](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_variables.html).

  You can also run in non-interactive mode by passing the
  `--extra-vars` or `-e` flag, followed by a map of  key-value pairs; one for
  each and every available input (see [inputs section](#inputs) below). For
  example:

  ```bash
  ansible-playbook \
    -i inventory.yml \
      -e '{
        "headscale_preauthkey":"<redacted>",
        "headscale_login_server":"http://headscale.batchprocessing.eumetsat.ewcloud.host:80",
        "htcondor_cm_external_ip":"100.64.0.32",
        "htcondor_password":"<redacted>"
      }' \
    htc-submit-flavour.yml 
  ```

## Inputs


| Name                        | Description                                                                                        | Type     | Default                                                           | Required |
| --------------------------- | ----------------------------------------------------------------------------------------------     | -------- | --------                                                          | :------: |
| headscale_login_server      | URI of the VPN server.                                                                             | `string` | `http://headscale.batchprocessing.eumetsat.ewcloud.host:80`       |    yes   |
| headscale_preauthkey        | Credentials of the VPN server.                                                                     | `string` | n/a                                                               |    yes   |
| htcondor_cm_external_ip     | IP Address of the HTCondor Manager node to which this submit node reports to.                      | `string` | `100.64.0.32`                                                     |    yes   |
| htcondor_password           | Password to authenticate against HTCondor Submit node pool                                        | `string` | n/a                                                               |    yes   |



## Dependencies


> 💡 Upon execution, a SBOM (SPDX format) is auto-generated and stored in the VM's file system root directory (see /sbom.json).

| Name | Home URL |
|------|-------|
| ewc-ansible-role-htcondor-submit | https://github.com/ewcloud/ewc-ansible-role-htcondor-submit |


