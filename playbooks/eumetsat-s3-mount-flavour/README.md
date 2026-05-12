# EUMETSAT S3 Mount Flavour

This Ansible Playbook configures and existing virtual machine
within the [European Weather Cloud (EWC)](https://europeanweather.cloud/), enabling to access remote public (EUMETSAT data buckets)[https://confluence.ecmwf.int/x/FUEXHQ], stored in S3 buckets, as if it was kept on the local disc.


## Features

- Self-service access to EUMETSAT data publicly available within the EWC (no credentials required)
- User-defined mounting location and configurable idle timeout
- Strict file permissions for mounted directories
- Boot-safe design; does not block boot on `S3` outage
- Advanced per-bucket cache/refresh tuning (via tweaking of included defaults)

## Usage

The step-by-step described below assume your local file system follows the example structure below, with `ewc-ansible-role-eumetsat-s3-mount` being a clone of this repository:

```
.
├── roles
│   └── ewc-ansible-role-eumetsat-s3-mount
├── inventory.yml
└── playbook.yml
```

### 1. Specify the target host and SSH credentials
Create an inventory file to specify address/credentials that Ansible should use
to reach the virtual machine you wish to configure:
```yaml
# inventory.yml
---
ewcloud:
  hosts:
    ewc_host:
      ansible_python_interpreter: /usr/bin/python3
      ansible_host: <add the IPV4 address of the target host>
      ansible_ssh_private_key_file: <add the path to local SSH private key file>
      ansible_user: <add the username which owns the SSH private key >
```

### 2. Customize the template

Edit input values for the template [variables](./vars/main.yml) as needed (see
[Inputs](#inputs) section for details).
Then, proceed to create an Ansible Playbook file to load your customizations: 

```yaml
# playbook.yml
---
- name: Mount EUMETSAT S3 buckets
  hosts: ewc_host
  become: true
  become_user: root
  become_method: ansible.builtin.sudo

  roles:
    - ewc-ansible-role-eumetsat-s3-mount
```

### 3. Apply the template


You can apply changes on the target host by running:
```bash
ansible-playbook -i inventory.yml playbook.yml
```

## Inputs

>💡 Buckets are pre-defined by default (see [defaults/main.yml](./defaults/main.yml)).


| Name |  Description | Type | Default | Required |
|------|-------------|------|---------|:--------:|
| vfs_cache_mode | Cache mode. Example: `writes` | `string` | n/a | yes |
| vfs_cache_max_size | Max cache size. Example: `512Mi` | `string` | n/a | yes |


## Dependencies
> 💡 Upon execution, a SBOM (SPDX format) is auto-generated and stored in the VM's file system root directory (see `/sbom.json`).

The following components will be included in the resulting environment:

| Component | Home URL |
|------|---------|
| curl | https://curl.se/  |
| unzip | http://www.info-zip.org/UnZip.html |
| fuse3 | https://github.com/libfuse/libfuse |
| fuse-overlayfs | https://github.com/containers/fuse-overlayfs  |
| rclone |	https://github.com/rclone/rclone |

## Changelog
All notable changes (i.e. fixes, features and breaking changes) are documented 
in the [CHANGELOG.md](./CHANGELOG.md).


## Contributing

Thanks for taking the time to join our community and start contributing!
Please make sure to:
* Familiarize yourself with our [Code of Conduct](./CODE_OF_CONDUCT.md) before 
contributing.
* See [CONTRIBUTING.md](./CONTRIBUTING.md) for instructions on how to request 
or submit changes.

## Authors

[European Weather Cloud](http://support.europeanweather.cloud/) 
<[support@europeanweather.cloud](mailto:support@europeanweather.cloud)>