# xcube Viewer Flavour
This Ansible Playbook configures an existing virtual machine running within the [European Weather Cloud (EWC)](https://europeanweather.cloud/) with the [xcube Viewer](https://xcube.readthedocs.io/en/latest/viewer.html) to visualize Earth Observation data in a user-friendly graphical user interface.

![xcube viewer example](img/xcube-viewer-example.png)

## Functionality
The template is designed to:

* Configure a pre-existing virtual machine running Ubuntu 24.04, with public IP address, and a minimum recommended 8GB of RAM, with the xcube Viewer - a web-based visualization tool that enables users to:
  * Interactively explore and visualize multi-dimensional Earth Observation data
  * Display data from various sources including remote S3-compatible object storage
  * View time-series data with animation capabilities
  * Customize visualization with different color maps and overlays
  * Access data through a user-friendly web interface at `http://<your-vm>.<tenancy-name>.<s|f>.ewcloud.host:80/viewer`. More info about the standard for url creation [here](https://confluence.ecmwf.int/display/EWCLOUDKB/EWC+DNS)

For more information about xcube capabilities, please refer to the [xcube Documentation](https://xcube.readthedocs.io/).

## Prerequisites
> ⚠️ Only Ubuntu 24.04 is supported.

> 💡 A VM plan with at least 8GB of RAM is recommended for successful setup and stable operation. 

* Install [git](https://git-scm.com/downloads) (version 2.0 or higher)
* Install [python](https://www.python.org/downloads) (version 3.9 or higher)
* Install [ansible](https://pypi.org/project/ansible) (version 2.15 or higher)
* If you plan to configure an existing VM, jump to the [Usage](#usage) section below
* If you have not yet provisioned a VM, it is required to do so. You may choose one of the following approaches:

  * **A) Deploy this template, together with a new VM, via the [EWCCLI](https://pypi.org/project/ewccli/)**

    [→ Go to Usage - Deployment on VM with EWC CLI](#usage---deployment-on-vm-with-ewc-cli)
  
  OR
  * **B) Provision a new VM via CLI:**
    * Create an SSH keypair (see [Creating the keys](https://confluence.ecmwf.int/display/EWCLOUDKB/Add+your+SSH+key+pair+to+Morpheus#AddyourSSHkeypairtoMorpheus-Creatingthekeys) section of the EWC documentation)
    * Add your SSH public key to OpenStack (see [Import SSH Key](https://confluence.ecmwf.int/display/EWCLOUDKB/EWC+-+OpenStack+Command-Line+client#EWCOpenStackCommandLineclient-ImportSSHkey) section of the EWC documentation)
    * Provision a new VM via the OpenStack CLI (see [How to create a VM using the OpenStack CLI](https://confluence.ecmwf.int/display/EWCLOUDKB/EWC+-+How+to+create+a+VM+using+the+Openstack+CLI) section of the EWC documentation)
    * Create a machine with the following specs: 
      * image_name: Ubuntu-24.04-20251007115108
      * flavor: eo.large1
      * Networks: network-private-fixed  
      * Security Groups: ssh-http-https, xcube-viewer
      * assign a floating IP

    [→ Go to Usage - Deployment on VM without EWC CLI](#usage---deployment-on-vm-without-ewc-cli)

## Usage
### Usage - Deployment on VM with EWC CLI

```bash
ewc hub deploy ewc-xcube-viewer --security-groups http-https-ssh --item-inputs tenancy_name="your-tenancy-name" --item-inputs federee="EUMETSAT"
```

### Usage - Deployment on VM without EWC CLI

### 1. Clone the repository

```bash
git clone https://github.com/ewcloud/ewc-ansible-playbook-flavours-and-provisioning.git
```

#### 1.1. Change to the specific Item's subdirectory

```bash
cd playbooks/xcube-viewer-flavour
```

#### 1.2. (Optional) Checkout a specific Item's version
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
Create an inventory file to specify address/credentials that Ansible should use to reach the virtual machine you wish to configure:

```yaml
# inventory.yml
---
ewcloud:
  hosts:
    xcube_viewer:
      ansible_python_interpreter: /usr/bin/python3
      ansible_host: <add the IPV4 address of the target host>
      ansible_ssh_private_key_file: <add the path to local SSH private key file>
      ansible_user: <add-username>
      ansible_ssh_common_args: -o StrictHostKeyChecking=accept-new
```

### 4. Configure and apply the template

#### 4.1. Interactive Mode

By running the following command, you can trigger an interactive session that prompts you for the necessary user inputs, and then applies changes to your target EWC environment:

```bash
ansible-playbook -i inventory.yml xcube-viewer-flavour.yml
```

#### 4.2. Non-Interactive Mode

>💡 To learn more about defining variables at runtime, checkout the [official Ansible documentation](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_variables.html).

You can also run in non-interactive mode by passing the `--extra-vars` or `-e` flag, followed by a map of key-value pairs for each available input (see [inputs section](#inputs) below). For example:

```bash
ansible-playbook \
  -i inventory.yml \
  -e '{
      "tenancy_name": "your-tenancy",
      "federee": "EUMETSAT",
      "xcube_env_wipe": "no",
      "xcube_env_name": "xcube-demo",
      "conda_installer": "https://github.com/conda-forge/miniforge/releases/latest/download/Miniforge3-Linux-x86_64.sh",
      "conda_update_base": "false",
      "conda_prefix": "/opt/conda",
      "conda_user": "root",
      "xcube_config_location": "./templates/xcube-demo-config.yml"
    }' \
  xcube-viewer-flavour.yml
```

## Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|----------|
| tenancy_name | Tenancy name for URL construction | string | - | yes |
| federee | Federee identifier (EUMETSAT or ECMWF) | string | - | yes |
| xcube_env_wipe | Flag to delete existing conda environment where xcube was previously installed. Only "yes" will be accepted to approve | string | no | no |
| xcube_env_name | Name of conda environment where xcube will be installed | string | xcube-demo | no |
| conda_installer | URI of the installer to use | string | https://github.com/conda-forge/miniforge/releases/latest/download/Miniforge3-Linux-x86_64.sh | no |
| conda_update_base | Boolean to decide whether base environment needs updating | bool | false | no |
| conda_prefix | Prefix where conda will be installed | string | /opt/conda | no |
| conda_user | User that will own the conda installation | string | root | no |
| xcube_config_location | Path of the config file for the xcube viewer | string | ./templates/xcube-demo-config.yml | no |

## Dependencies

> 💡 Upon execution, a SBOM (SPDX format) is auto-generated and stored in the VM's file system root directory (see `/sbom.json`).

Applying this template will trigger the installation of the following open-source packages onto your desired target host:

| Name | Version | License | Package Info |
|------|---------|---------|--------------|
| numcodecs | 0.15.1 | MIT | https://anaconda.org/channels/conda-forge/packages/numcodecs/overview |
| python | 3.11 | PSF License | https://www.python.org/ |
| xcube | 1.13.0 | MIT | https://anaconda.org/channels/conda-forge/packages/xcube/files |

