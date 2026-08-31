# EUMETCast Terrestrial AMT Flavour

This Ansible Playbook configures an existing virtual machine running
within the [European Weather Cloud (EWC)](https://europeanweather.cloud/), to equip it with the [EUMETCast Terrestrial](https://user.eumetsat.int/data-access/eumetcast-terrestrial) over [AMT software stack](https://gitlab.eumetsat.int/open-source/amt). 


EUMETCast is EUMETSAT’s primary dissemination mechanism for the near-real-time delivery of satellite data. EUMETCast serves data through two complementary delivery systems: EUMETCast Satellite and EUMETCast Terrestrial.

![EUMETCast overview](https://raw.githubusercontent.com/ewcloud/ewc-ansible-playbook-flavours-and-provisioning/refs/heads/main/playbooks/eumetcast-terrestrial-amt-flavour/docs/images/eumetcast-overview.png)

Terrestrial services for data distribution, available and supported, include:

>💡 Additional terrestrial services for data distribution will be created in the future for various missions.
If unsure which services your you access, please contact the EUMETSAT User Helpdesk ([ops@eumetsat.int](mailto:ops@eumetsat.int)).


| | | | |
| --- | --- | --- | --- |
| Terrestrial Service | **Total Bandwidth (Mbps)** | Data | Default |
| **ter-1** | 240 | **EPS, MSG, Sentinel-3A/B, Sentinel-5P, Third Party data, MTG** | yes |
| **ter-2** | 168 | **Sentinel-6 data** | |
| **ter-3** | 230 | **Sentinel-5P L1B, Sentinel-3A/B OLCI L1 FR, Sentinel-3A/B SLSTR L1B, FY3 HIRAS, FY4 GIIRS, GOSAT, MTG HRFI-FD (4 high-res full-disk bands)** | |

## Functionality

* Installs and configures the EUMETCast Terrestrial client over Automatic Multicast Tunnelling (AMT)
* Deploys the Tellicast terrestrial receiver runtime
* Configures network translation required for multicast reception over AMT
* Creates maintenance jobs for log and data retention
* Automatically cleans up disk space by keeping only the latest 5 minutes of transmitted data, under the `/home/eumetuser/data` subdirectory.

## Prerequisites
> ⛔ When deploying on ECMWF site, network traffic required by this Item may be blocked due to ECMWF Firewall. Please get in touch with us through the [EWC Support portal](https://support.europeanweather.cloud/).

* Register a user account on the [EUMETSAT User Portal](https://eoportal.eumetsat.int/)
* Subscribe to the service and data for EUMETCast Terrestrial
  * Ensure you select at least one EKU (temporal quota allocation)
* Request a EUMETCast Terrestrial user key for access from the EWC:
  * Write to EUMETSAT Helpdesk ([ops@eumetsat.int](mailto:ops@eumetsat.int)), making sure to provide them your EUMETSAT User Portal username alongside your request
* Verify the `eumetcast` OpenStack Security Group exists in your EWC tenancy
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
ewc hub deploy eumetcast-terrestrial-amt-flavour
```

### Deploy via native tooling (Ansible)

#### 1. Setup working environment

* Install [git](https://git-scm.com/downloads) (version 2.0 or higher )
* Install [python](https://www.python.org/downloads) (version 3.9 or higher) 
* Install [ansible](https://pypi.org/project/ansible) (version 2.15 or higher)
* If you plan to configure an existing VM, ensure it meets the minium requirements before proceeding:
  * VM Image: Ubuntu 22
  * VM Plan: 8 CPU cores, 64GB RAM, 265GB Disk + 1TB Disk (secondary)
    > ⚠️ Make sure the secondary disk is formatted and mounted by following this guide steps from the [EWC Knowledge Base](https://confluence.ecmwf.int/x/WHAEJQ#EWCCloudManagementUIStorageAddavolumetoaninstance-Formatthenewdisk).
  * Network: Private
  * Security Groups: `eumetcast`
  * Floating IP: Optional (not recommended from a security standpoint)
  
  Otherwise, provision a new VM with above specifications before continuing (see [EWC Getting Started: Provision a VM](https://confluence.ecmwf.int/x/2RvEJg) for details).

#### 2. Clone the repository

```bash
git clone https://github.com/ewcloud/ewc-ansible-playbook-flavours-and-provisioning.git
```

##### 2.1. Change to the specific Item's subdirectory

```bash
cd playbooks/eumetcast-terrestrial-amt-flavour
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
          ansible_ssh_user:  ubuntu
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
  ansible-playbook -i inventory.yml eumetcast-terrestrial-amt-flavour.yml
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
        "tellicast_license_user_name":"<redacted>",
        "tellicast_license_user_key":"<redacted>"
      }' \
    eumetcast-terrestrial-amt-flavour.yml
  ```

## Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|----------|
| tellicast_license_user_name | Tellicast license user identifier, equivalent to your EUMETSAT User Portal username. | `string` | n/a | yes |
| tellicast_license_user_key | Tellicast license activation key (i.e. the user key you obtained via from EUMETSAT Helpdesk). | `string` | n/a | yes |

## Dependencies


> 💡 Upon execution, a SBOM (SPDX format) is auto-generated and stored in the VM's file system root directory (see `/sbom.json`).

| Name  | Home URL |
|-------|-------|
| ewc-ansible-role-eumetcast-terrestrial-amt | https://github.com/ewcloud/ewc-ansible-role-eumetcast-terrestrial-amt |

## Operation

Checkout the [how-to guides](./docs/how-to/) to learn about management of the Item after initial setup.