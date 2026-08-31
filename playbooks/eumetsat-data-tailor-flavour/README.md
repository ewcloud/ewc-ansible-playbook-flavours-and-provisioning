# EUMETSAT Data Tailor Flavour

This Ansible Playbook configures an existing virtual machine running
within the [European Weather Cloud (EWC)](https://europeanweather.cloud/), to equip it with the Data Tailor Standalone and EUMETSAT Data Access Client (EUMDAC).

## Functionality
> 💡 Unlike the Data Tailor Web Services (DTWS), which can be used with EUMDAC or via https://tailor.eumetsat.int, the standalone version is generally faster and does not have limitations such as maximum concurrent jobs or workspace size.

The Data Tailor is a product customization toolbox designed to:
* Enable users to tailor satellite data to their specific needs. 
* Offers the ability to subset and aggregate data products across space and time, filter layers, generate quick looks, reproject data onto new coordinate reference systems, and reformat data into widely used Geographic Information System (GIS) formats such as netCDF and GeoTIFF, as well as image formats like JPEG and PNG. 
* Customize data from various satellite collections, including METOP, MFG ,MSG, MTG (Meteosat Third Generation) and Sentinel-3. 

For more information on capabilities of the Data Tailor, please refer to [Data Tailor Standalone Guide on User Portal](https://user.eumetsat.int/resources/user-guides/data-tailor-standalone-guide) and for more information about the available products and customizations inside the Data Tailor, please go to [Products and Customizations Available in the Data Tailor](https://user.eumetsat.int/resources/user-guides/data-store-detailed-guide#ID-Products-and-customisation-available-in-the-Data-Tailor) page.

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
ewc hub deploy data-tailor-flavour
```

### Deploy via native tooling (Ansible)

#### 1. Setup working environment

* Install [git](https://git-scm.com/downloads) (version 2.0 or higher )
* Install [python](https://www.python.org/downloads) (version 3.9 or higher) 
* Install [ansible](https://pypi.org/project/ansible) (version 2.15 or higher)
* If you plan to configure an existing VM, ensure it meets the minium requirements before proceeding:
  * VM Image: Ubuntu 24 or 22
  * VM Plan: 4 CPU cores, 16GB RAM, 80GB Disk
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
cd ewc-ansible-playbook-flavours-and-provisioning/playbooks/eumetsat-data-tailor-flavour
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
        ansible_python_interpreter: auto
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

* **Interactive Mode**

  By running the following command, you can trigger an interactive session that
  prompts you for the necessary user inputs, and then applies changes to your
  target EWC environment:

  ```bash
  ansible-playbook -i inventory.yml eumetsat-data-tailor-flavour.yml
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
        "data_tailor_env_wipe": "no",
        "data_tailor_env_name": "epct-desktop",
        "conda_installer": "https://github.com/conda-forge/miniforge/releases/latest/download/Miniforge3-Linux-x86_64.sh",
        "conda_update_base": "false",
        "conda_prefix": "/opt/conda",
        "conda_user": "root"
      }' \
    eumetsat-data-tailor-flavour.yml
  ```

## Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|:--------:|
| data_tailor_env_wipe | flag to delete existing conda environment where data tailor was previously installed. Only `yes` will be accepted to approve | `string` | `no` | yes |
| data_tailor_env_name | name of conda environment where data tailor will be installed | `string` | `epct-desktop` | yes |
| conda_installer  | URI of the installer to use | `string` | `https://github.com/conda-forge/miniforge/releases/latest/download/Miniforge3-Linux-x86_64.sh` | yes |
| conda_update_base | boolean to decide wether base environment needs updating | `bool` | `false` | yes |
| conda_prefix | prefix where conda will be installed | `string` | `/opt/conda` | yes |
| conda_user | user that will own the conda installation | `string` | `root` | yes |

## Dependencies

> 💡 Upon execution, a SBOM (SPDX format) is auto-generated and stored in the VM's file system root directory (see `/sbom.json`).

| Name | Home URL |
|------|---------|
| ewc-ansible-role-conda | https://github.com/ewcloud/ewc-ansible-role-conda |
| ewc-ansible-role-data-tailor | https://github.com/ewcloud/ewc-ansible-role-data-tailor |

## Operation
Checkout the following how-to guides to learn about management of the Item after initial setup:
* [How to initialize the environment](./docs/how-to/how-to-initialize-environment.md)
