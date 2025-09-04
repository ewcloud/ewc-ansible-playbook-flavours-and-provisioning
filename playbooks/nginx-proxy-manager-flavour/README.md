# Nginx Proxy Manager flavour
>💡 Not to be confused with Nginx, the web server and reverse proxy.

This Ansible Playbook configures an existing virtual machine running
within the [European Weather Cloud (EWC)](https://europeanweather.cloud/), to operate as a [Nginx Proxy Manager](https://nginxproxymanager.com/) server.

Nginx Proxy Manager is full-featured tool that helps to lower the barriers to entry for users who are interested in learning and working with [Nginx](https://nginx.org/en/) servers.

## Functionality
* Virtual host management
* Ability to cache assets
* Blocking of common exploits
* Websocket support
* Access list configuration
* SSL and HTTP/2 support
* Host redirection with HTTP code configuration
* TCP and UDP stream support
* User management
* Nginx Proxy
* Manager log auditing

## Usage

### 1. Specify the target host and SSH credentials
Create an inventory file to specify address/credentials that Ansible should use
to reach the virtual machine you wish to configure:

```yaml
# inventory.yml
---
ewcloud:
  hosts:
    nginx_proxy_manager:
      ansible_python_interpreter: /usr/bin/python3
      ansible_host: <add the IPV4 address of the target host>
      ansible_ssh_private_key_file: <add the path to local SSH private key file>
      ansible_user: ubuntu
      ansible_ssh_common_args: -o StrictHostKeyChecking=accept-new
```

### 2. Configure and apply the template

#### 2.1. Interactive Mode

By running the following command, you can trigger an interactive session that
prompts you for the necessary user inputs, and then applies changes to your
target EWC environment:

```bash
ansible-playbook -i inventory.yml nginx-proxy-manager-flavour.yml
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
  -e "npm_admin_ui_port=8080" \
  -e "os_security_group_name=nginx-proxy-manager" \
  nginx-proxy-manager-flavour.yml
```

## Inputs
> ⛔ If deploying to an instance on the ECMWF site, using a high port numbers such as the in the examples, will prevent you from accessing the Nginx Proxy Manager UI from the pubic internet, even when a valid security group is attached to the instance. This is due to the outer perimeter firewall of the ECMWF site. For details see [EWC Security guidelines - Restrictive firewall (allow-listing)](https://confluence.ecmwf.int/display/EWCLOUDKB/EWC+Security+guidelines#EWCSecurityguidelines-Restrictivefirewall(allow-listing)).

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|:--------:|
| npm_admin_ui_port | port number at which the Nginx Proxy Manager admin UI is served. Example: `8080` | `number` | n/a | yes |
| os_security_group_name | OpenStack security group containing all firewall rules required for Nginx Proxy Manager operation. Example: `nginx-proxy-manager` | `string` | n/a | yes |


## Dependencies
> ⚠️ Only Ubuntu 22.04 images are currently supported.
This is due to constrains imposed by the required ewc-ansible-role-nginx-proxy-manager
Ansible Role.

> 💡 A VM plan with at least 8GB of RAM is recommended for successful setup and
stable operation.

| Name | Version | License | Home URL |
|------|---------|------|------|
| ewc-ansible-role-nginx-proxy-manager | 1.0 |  MIT | https://github.com/ewcloud/ewc-ansible-role-nginx-proxy-manager |
