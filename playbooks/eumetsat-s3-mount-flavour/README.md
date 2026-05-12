# EUMETSAT S3 Mount Flavour

This Ansible Playbook configures a virtual machine existing
within the [European Weather Cloud (EWC)](https://europeanweather.cloud/), enabling to access remote public [EUMETSAT data buckets](https://confluence.ecmwf.int/x/FUEXHQ) as if it was stored on a local disk.

## Features

- Self-service access to EUMETSAT data publicly available within the EWC (no credentials required)
- Strict file permissions for mounted directories
- Boot-safe design; does not block boot on `S3` outage

## Prerequisites
> ⚠️ Only Ubuntu version 24 and 22, or RockyLinux versions 9 or 8 supported due to constrains imposed by [dependencies](#dependencies).

* Install [git](https://git-scm.com/downloads) (version 2.0 or higher )
* Install [python](https://www.python.org/downloads) (version 3.9 or higher) 
* Install [ansible](https://pypi.org/project/ansible) (version 2.15 or higher)
* If you plan to configure an existing VM, then skip to the [Usage](#usage) section below
* If you have not yet provisioned a VM, it is required to do so. You may choose one of the following approaches:
  * **A) Provision a new VM via UI:**
    * Create an SSH keypair (see [Creating the keys](https://confluence.ecmwf.int/display/EWCLOUDKB/Add+your+SSH+key+pair+to+Morpheus#AddyourSSHkeypairtoMorpheus-Creatingthekeys) section of the EWC documentation)
    * Import the SSH public key into Morpheus (see [Adding the keys in Morpheus](https://confluence.ecmwf.int/display/EWCLOUDKB/Add+your+SSH+key+pair+to+Morpheus#AddyourSSHkeypairtoMorpheus-AddingthekeysinMorpheus) section of the EWC documentation)
    * Provision a new VM through the web portal (see [Provision a new Instance - Web](https://confluence.ecmwf.int/display/EWCLOUDKB/Provision+a+new+instance+-+web) section of the EWC) documentation

    OR 
  * **B) Provision a new VM via CLI:**
    * Create an SSH keypair (see [Creating the keys](https://confluence.ecmwf.int/display/EWCLOUDKB/Add+your+SSH+key+pair+to+Morpheus#AddyourSSHkeypairtoMorpheus-Creatingthekeys) section of the EWC documentation)
    * Add you SSH public key to OpenStack (see [Import SSH Key](https://confluence.ecmwf.int/display/EWCLOUDKB/EWC+-+OpenStack+Command-Line+client#EWCOpenStackCommandLineclient-ImportSSHkey) section of the EWC documentation).
    * Provision a new VM via the OpenStack CLI (see [How to create a VM using the OpenStack CLI](https://confluence.ecmwf.int/display/EWCLOUDKB/EWC+-+How+to+create+a+VM+using+the+Openstack+CLI) section of the EWC documentation)
  
    OR
  * **C) Deploy this template, together with a new VM, via the [EWCCLI](https://pypi.org/project/ewccli/)**

## Usage

### 1. Clone the repository

```bash
git clone https://github.com/ewcloud/ewc-ansible-playbook-flavours-and-provisioning.git
```

#### 1.1. Change to the specific Item's subdirectory

```bash
cd playbooks/eumetsat-s3-mount-flavour
```

#### 1.2. (Optional) Checkout an specific Item's version
>⚠️ Make sure to replace `x.y.z` in the command below, with your version of preference.

```bash
git checkout x.y.z
```


### 2. Download Ansible dependencies
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
    eumetsat_s3_mount:
      ansible_python_interpreter: /usr/bin/python3
      ansible_host: <add the IPV4 address of the target host>
      ansible_ssh_private_key_file: <add the path to local SSH private key file>
      ansible_user: <add the default user according to your chosen VM image>
      ansible_ssh_common_args: -o StrictHostKeyChecking=accept-new
```

### 4. Configure and apply the template

#### 4.1. Interactive Mode

By running the following command, you can trigger an interactive session that
prompts you for the necessary user inputs, and then applies changes to your
target EWC environment:

```bash
ansible-playbook \
  -i inventory.yml \
  -e '{
      "vfs_cache_mode":"writes",
      "vfs_cache_max_size":"writes"
    }' \
  eumetsat-s3-mount-flavour.yml
```

#### 4.2. Non-Interactive Mode

>💡 To learn more about defining variables at runtime, checkout the
[official Ansible documentation](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_variables.html).

You can also run in non-interactive mode by passing the
`--extra-vars` or `-e` flag, followed by a map of  key-value pairs; one for
each and every available input (see [inputs section](#inputs) below). For
example:

```bash
ansible-playbook -i inventory.yml eumetsat-s3-mount-flavour.yml
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
