# EUMETSAT S3 Mount Flavour

This Ansible Playbook configures a virtual machine existing
within the [European Weather Cloud (EWC)](https://europeanweather.cloud/), enabling to access remote public [EUMETSAT data buckets](https://confluence.ecmwf.int/x/FUEXHQ) as if they were stored on a local disk.

Supports deployment to instances on both the ECMWF and the EUMETSAT compute sites of the EWC.

## Features

- Self-service access to EUMETSAT data publicly available within the EWC (no credentials required)
- Strict file permissions for mounted directories under the `/mnt` directory
- Boot-safe design; does not block boot on `S3` outage

## Prerequisites

* Verify the `ssh-https` OpenStack Security Group exists in your EWC tenancy
  > 💡 You may create Security Groups via [this EWC Community Hub Item](https://europeanweather.cloud/community-hub/openstack-compute-instance) if pre-required ones are missing.


## Usage

### Deploy via EWCCLI

>⚠️ Deployment via EWCCLI is only possible from a VM within your EWC private network.

#### 1. Setup working environment

```bash
pip install ewccli
```

#### 2. Configure access credentials

```bash
ewc login
```

#### 3. Deploy
>💡 To lean about EWCCLI deployment customization, checkout the [EWC User Stories: I want to use the ewccli](https://confluence.ecmwf.int/x/NlYiK) documentation pages.


```bash
ewc hub deploy eumetsat-s3-mount-flavour
```

### Deploy via native tooling (Ansible)

#### 1. Setup working environment

* Install [git](https://git-scm.com/downloads) (version 2.0 or higher )
* Install [python](https://www.python.org/downloads) (version 3.9 or higher)
* Install [ansible](https://pypi.org/project/ansible) (version 2.15 or higher)
* If you plan to configure an existing VM, ensure it meets the minium requirements before proceeding:
  * VM Image: Ubuntu or RockyLinux
  * VM Plan: 4 CPU cores, 8GB RAM, 40GB Disk
  * Network: Private
  * Security Groups: `ssh-https`
  * Floating IP: Optional (not recommended from a security standpoint)

  Otherwise, provision a new VM with above specifications before continuing (see [EWC Getting Started: Provision a VM](https://confluence.ecmwf.int/x/2RvEJg) for details).

#### 2. Clone the repository

```bash
git clone https://github.com/ewcloud/ewc-ansible-playbook-flavours-and-provisioning.git
```

##### 2.1. Change to the specific Item's subdirectory

```bash
cd playbooks/eumetsat-s3-mount-flavour
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
      target:
        ansible_python_interpreter: auto
        ansible_host: <add the PRIVATE IP address of the target host>
        ansible_ssh_private_key_file: <add the path to local SSH private key file>
        ansible_user: <add the default user according to your chosen VM image>
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
        ansible_python_interpreter: auto
        ansible_host: <add the PUBLIC IP address of the target host>
        ansible_ssh_private_key_file: <add the path to local SSH private key file>
        ansible_user: <add the default user according to your chosen VM image>
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
          ansible_ssh_user: <add the default user according to your chosen VM image>
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

* **Interactive Mode**

  By running the following command, you can trigger an interactive session that
  prompts you for the necessary user inputs, and then applies changes to your
  target EWC environment:


  ```bash
  ansible-playbook -i inventory.yml eumetsat-s3-mount-flavour.yml
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
        "vfs_cache_mode":"writes",
        "vfs_cache_max_size":"512Mi"
      }' \
    eumetsat-s3-mount-flavour.yml
  ```


## Inputs
> 💡 To learn more about the valid input values and their performance implications, checkout the [rclone official documentation](https://rclone.org/commands/rclone_mount/#vfs-file-caching)

| Name |  Description | Type | Default | Required |
|------|-------------|------|---------|:--------:|
| vfs_cache_mode | Cache mode | `string` | `writes` | yes |
| vfs_cache_max_size | Max total size of objects in the cache | `string` | `512Mi` | yes |


## Dependencies

> 💡 Upon execution, a SBOM (SPDX format) is auto-generated and stored in the VM's file system root directory (see `/sbom.json`).

| Component | Home URL |
|------|---------|
| ewc-ansible-role-eumetsat-s3-mount | https://github.com/ewcloud/ewc-ansible-role-eumetsat-s3-mount  |
