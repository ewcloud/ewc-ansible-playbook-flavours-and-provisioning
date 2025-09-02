# Data tailor flavour

This Ansible Playbook configures an existing virtual machine running
within the [European Weather Cloud (EWC)](https://europeanweather.cloud/), to equip it with the Data Tailor Standalone and EUMETSAT Data Access Client (EUMDAC).

## Functionality
> 💡 Unlike the Data Tailor Web Services (DTWS), which can be used with EUMDAC or via https://tailor.eumetsat.int, the standalone version is generally faster and does not have limitations such as maximum concurrent jobs or workspace size.

The Data Tailor is a product customization toolbox designed to:
* Enable users to tailor satellite data to their specific needs. 
* Offers the ability to subset and aggregate data products across space and time, filter layers, generate quick looks, reproject data onto new coordinate reference systems, and reformat data into widely used Geographic Information System (GIS) formats such as netCDF and GeoTIFF, as well as image formats like JPEG and PNG. 
* Customize data from various satellite collections, including METOP, MFG ,MSG, MTG (Meteosat Third Generation) and Sentinel-3. 

For more information on capabilities of the Data Tailor, please refer to [Data Tailor Standalone Guide on User Portal](https://user.eumetsat.int/resources/user-guides/data-tailor-standalone-guide) and for more information about the available products and customisations inside the Data Tailor, please go to [Products and Customisations Available in the Data Tailor](https://user.eumetsat.int/resources/user-guides/data-store-detailed-guide#ID-Products-and-customisation-available-in-the-Data-Tailor) page.

## Usage

### 1. Specify the target host and SSH credentials
Create an inventory file to specify address/credentials that Ansible should use
to reach the virtual machine you wish to configure:

```yaml
# inventory.yml
---
ewcloud:
  hosts:
    data_tailor:
      ansible_python_interpreter: /usr/bin/python3
      ansible_host: <add the IPV4 address of the target host>
      ansible_ssh_private_key_file: <add the path to local SSH private key file>
      ansible_user: <add the default user according to your chosen VM image>
      ansible_ssh_common_args: -o StrictHostKeyChecking=accept-new
```

### 2. Configure and apply the template

#### 2.1. Interactive Mode

By running the following command, you can trigger an interactive session that
prompts you for the necessary user inputs, and then applies changes to your
target EWC environment:

```bash
ansible-playbook -i inventory.yml data-tailor-flavour.yml
```

#### 2.2. Non-Interactive Mode

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
  data-tailor-flavour.yml
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
> 💡 A VM plan with at least 16GB of RAM is recommended for successful setup and
stable operation.

| Name | Version | License | Package Info |
|------|---------|------|------|
| ewc-ansible-role-conda | 1.1 |  Apache-2.0 | https://github.com/ewcloud/ewc-ansible-role-conda |
| ewc-ansible-role-data-tailor | 1.0 |  MIT | https://github.com/ewcloud/ewc-ansible-role-data-tailor |
