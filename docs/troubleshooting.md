# Troubleshooting

When working with the default stack provisioning templates, you might encounter
various issues. Here are some common problems and how to troubleshoot them:

## 1. Execution failed due to an unknown issue and cleanup is desired before proceeding

> 💡 You may use this method to edit attributes of a existing instance,
instead of destroying it. This is useful if you have reason to believe that
updating the VM plan or VM image would resolve any issue you've encountered,
or if you simply wish to adjust the instance after you are done with this
template.

It is possible that this template fails to complete successfully due to reasons
specific to your EWC tenancy, and which are naturally not listed below.

### Solution
If you are unable to troubleshot the specific error message you observe, before
reaching out to the EWC support team, we recommend you delete any lingering
resources from the failed run and attempt to reproduce the error.

You can remove dangling resources by changing to the same directory you set
for `tf_project_path` at the start of the failed run. Withing that directory, you
should find a `main.tf` and a `outputs.tf` files, containing the Terraform definition
of your instance. Remove the contents of both files, and then execute:
```
truncate -s 0 maint.tf && \
truncate -s 0 outputs.tf && \
terraform apply --auto-approve
```

OR

To delete the resources without editing the files, simply run:
```
terraform apply --destroy --auto-approve
```

## 2. Terraform plan could not be created (`auth_url` or `cloud` must be specified)

To authenticate, this template attempts to read the OpenStack Application
credentials you received from the EWC support, either from environmental
variables or YAML configuration file in a standard location.

If the credentials cannot be read, you may encounter the following error:
```
Terraform plan could not be created\nSTDOUT: \nPlanning failed. Terraform encountered an error while generating this plan.\n\n\nSTDERR: \nError: Invalid provider configuration\n\nProvider \"registry.terraform.io/terraform-provider-openstack/openstack\"\nrequires explicit configuration
...
Error: One of 'auth_url' or 'cloud' must be specified
...
```

### Solution

Ensure the application credentials you exposed in your work environment.

## 3. Terraform plan could not be created (`data.openstack_networking_network_v2.external`)

To simplify configuration, this template inferrers the default external
(public-internet connected) network where the instance shall be provisioned,
based on your choice of for the `ewc_provider` input variable.

If the default external network is not available during instance provision planning,
you might encounter an error like:
```
Terraform plan could not be created\nSTDOUT: data.openstack_networking_network_v2.external: Reading
...
\n\nSTDERR: \nError: Your query returned no results. Please change your search criteria and try again.\n\n  with data.openstack_networking_network_v2.external,\n  on datasources.tf line 1, in data \"openstack_networking_network_v2\" \"external\":\n   1: data \"openstack_networking_network_v2\" \"external\"
...
```

### Solution

Ensure the application credentials you exposed in your work environment
belong to the correct EWC tenancy.

Also, make sure that the EWC provider of the tenancy matches your
choice of the `ewc_provider` input variable, before re-running the
playbook.

Finally, double-check that the default external network,
(`external-internet` or `external` for ECMWF or EUMETSAT tenancies
respectively), exists in your tenancy or have not been
renamed/removed.

## 4. Remote host identification has changed

In cases where the following sequence of events took place:
1. You successfully connected to the target VM via SSH at least once (by
running this playbook until the end, by using the `ssh` utility in your shell, etc.)
2. The VM was destroy and the recreated after the initial connection
3. You run this playbook

Execution may fail at the start of the IPA server installation, with the following
error:
```
Failed to connect to the host via ssh:
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@    WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!     @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
IT IS POSSIBLE THAT SOMEONE IS DOING SOMETHING NASTY!
Someone could be eavesdropping on you right now (man-in-the-middle attack)!
It is also possible that a host key has just been changed.
The fingerprint for the ED25519 key sent by the remote host is
SHA256:aBcdE1f2GHh3iJKlMNOpqRstuV3Wxyzab4cdE+fGH45i.
Please contact your system administrator.
Add correct host key in /home/ubuntu/.ssh/known_hosts to get rid of this message.
Offending ECDSA key in /home/ubuntu/.ssh/known_hosts:21
Host key for 192.168.1.122 has changed and you have requested strict checking.
Host key verification failed.
```

### Solution

Remove the line in your `~/.ssh/know_hosts` file that
shows the same IP address displayed in the error message before re-running
the playbook.

## 5. Quota exceeded for resource "Floating IP" although none was requested

This template assumes your working environment is running outside of the EWC
private network where your instance placed.
For convenience, it attempts to allocated a temporal public IP address
to enable configuration of the target instance via ssh over the public internet.

However, your EWC tenancy has a maximum number of available public IP addresses.
If you hit that limit during execution of this template, the following error
will be raised:

```
Error: Error creating openstack_networking_floatingip_v2: Expected HTTP response code [201 202] when accessing [POST https://<redacted>/v2.0/floatingips], but got 409 instead\n{\"NeutronError\": {\"type\": \"OverQuota\", \"message\": \"Quota exceeded for resources: ['floatingip'].\", \"detail\": \"\"}}\n\n  with openstack_networking_floatingip_v2.fip[0],\n  on main.tf line 64, in resource \"openstack_networking_floatingip_v2\" \"fip\":\n  64: resource \"openstack_networking_floatingip_v2\" \"fip\" {", "rc": 1, "stderr": "\nError: Error creating openstack_networking_floatingip_v2: Expected HTTP response code [201 202] when accessing [POST https://<redacted>/v2.0/floatingips], but got 409 instead\n{\"NeutronError\":
...
```

### Solution

If no public IP addresses in your tenancy can be released, at least temporarily,
get in touch with the EWC support team to request an clarify your available options.

## 6. Unable to find flavour with name ...

If you input an invalid value for `flavour_name`, you might get the following
error:

```
Error: Unable to find flavor with name eo2.medium\n\n  with openstack_compute_instance_v2.instance,\n  on main.tf line 1, in resource \"openstack_compute_instance_v2\" 
...
 ```

### Solution

Ensure the application credentials you exposed in your work environment
belong to the correct EWC tenancy.

Also, make sure that the EWC provider of the tenancy matches your
choice of the `ewc_provider` input variable, before re-running the
playbook.

Finally, double-check that the chosen flavour name exists in your
tenancy and has not been renamed/removed.

## 7. The role was not found

This playbook requires one or more Ansible Roles to be be available
in your working environment. Otherwise, an error such as the one
below might be thrown during execution:

 ```
 ERROR! the role '<role-name>-<MAJOR.MINOR>' was not found in ~/$(pwd)/roles:~/.ansible/roles:/usr/share/ansible/roles:/etc/ansible/roles:~/$(pwd)
 ...
 ```

### Solution

Make sure to download and place the required Ansible Roles within
one of the paths Ansible expects to find them. Check the "Getting Started"
section of the main [README.md](../../README.md) for details on
how to do so.

## 8. Could not resolve host `mirrors.rockylinux.org`

If you are provisioning a new IPA server instance after teardown a 
fully-operational one, it is possible that you encounter errors at the point
your new instance post-provisioning (i.e. package installation) starts, such
as:

```
Failed to download metadata for repo 'appstream': Cannot prepare internal mirrorlist: Curl error (6): Couldn't resolve host name for https://mirrors.rockylinux.org/mirrorlist?arch=x86_64&repo=AppStream-8 [Could not resolve host: mirrors.rockylinux.org]
...
```

### Solution

Double-check that the IP address(es) set for DNS nameservers in the subnet where
your instance is connected, are pointing to a valid DNS server or servers.

You can replace the `my-tenant-private-network` placeholder in the bash line below,
and execute it, within an authenticated working environment, to get the required 
information:

```bash
export PRIVATE_NETWORK_NAME=my-private-network && \
openstack subnet list --network "$PRIVATE_NETWORK_NAME" -f value -c ID -c Name | while read id name; do echo -n "$name ($id): "; openstack subnet show "$id" -f value -c dns_nameservers; done
```
If a valid network name was set, the output if the command should be in the format:

`Network name (Network ID): [dns nameserver IP 1, dns nameserver IP 2, ...,  dns nameserver IP N]`

Before proceeding with resource cleanup and re-deployment, in case none of the DNS nameserver IPs
points to a valid DNS server, you can add one by replacing `my-private-subnet` in the command
below and executing it:
```bash
export PRIVATE_SUBNET_NAME=my-private-subnet && \
openstack subnet set --dns-nameserver 8.8.8.8 "$PRIVATE_SUBNET_NAME"  && \
openstack subnet set --dns-nameserver 1.1.1.1 "$PRIVATE_SUBNET_NAME"
```

## 9. Failed to connect to the host via ssh... permission denied

After instance creation or updating is completed by Terraform, Ansible will
attempt to connect to the instance via SSH, to change configurations of its
operative system or application running on it. If your local private SSH key
does not correspond to the public SSH key that was copied into the machine
during creation, you will get the an error like:

```
Failed to connect to the host via ssh: Warning: Permanently added
...
to the list of known hosts.\r\n
...
Permission denied (publickey,gssapi-keyex,gssapi-with-mic).
```
### Solution

Make sure the value you set to `private_keypair_path` points to the correct private
SSH key, and that the public key name you set in `public_keypair_name` matches the
name of the public key you previously imported to OpenStack.
To learn more about how to import your public SSH key into OpenStack, checkout the
official [EWC documentation](https://confluence.ecmwf.int/display/EWCLOUDKB/EWC+-+OpenStack+Command-Line+client#EWCOpenStackCommandLineclient-ImportSSHkey).


## 10. Failed to connect to host via ssh... system is booting up
After instance creation or updating is completed by Terraform, it might take
a couple of minutes for its SSHD service to become fully operational.
A wait time of 300 seconds is configured on the templates, before
Ansible attempts to connect to the instance via SSH. In case the port
accepts traffic prematurely, you might get an error like:

```
Failed to connect to the host via ssh: Warning: Permanently added
...
to the list of known hosts.\r\n\"System is booting up. Unprivileged users are not permitted to log in yet. Please come back later.
For technical details, see pam_nologin(8).\"\nConnection closed by
...
```

### Solution

Wait for couple more minutes and re-run the playbook with the same inputs as in
the previous run. It should pickup right where it left when the timeout occurred.

## 11. Failed during Ansible execution due to Python "future" module syntax errors
If the Python version running on your localhost is to far ahead (three or more major releases) from the python version running on 
the the target host, Ansible may fail mid run with an error like:

```
"module_stdout": " ...  from __future__ import annotations\r\n ^\r\nSyntaxError: future feature annotations is not defined\r\n", 
...
"msg": "The following modules failed to execute: ansible.legacy.setup."
...
````

### Solution
>💡 Ansible attempts to run the playbook on the target host by using the Python binary specified in your inventory.yml file as `ansible_python_interpreter`.

On your target host, try to update the default Python version, for example by using [pyenv](https://github.com/pyenv/pyenv) to install and configure a 
new default, or by swapping the virtual image used by your instance with a newer one (which should include a newer Python version).

If technical or organizational reasons prevent you from updating the remote host, or updating its Python version does not solve the issue, try downgrading
the Python version used on your localhost to trigger the execution, such that it matches that of the remote host.

## 12. IPA Client Enroll Flavour copmleted without errors but LDAP users are unable to SSH into the VM

It is possible that the execution of the template goes from start to finish without errors, but you still find yourself unable to use an LDAP account to access the target host.
In such cases, take note of how fast the template runtime took to complete; anything less than 30 seconds is a clear sign that Ansible could not hit its target.

If that is indeed the case, the logs of the run will clearly state that no host was matched when attempting to install and configure the IPA client:
```
...
PLAY[Install IPA client and enroll into an IPA server's LDAP/DNS services] *******
skipping: no hosts matched
...
```

### Solution

Ensure the contents of your `inventory.yml` file includes the inventory group specified by `hosts: ipa_client` (see full example on the Item's [README.md](../playbooks/ipa-client-enroll-flavour/README.md)).
