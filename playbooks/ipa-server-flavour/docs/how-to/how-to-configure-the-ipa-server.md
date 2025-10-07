# How to configure the IPA server (LDAP and DNS)

## Introduction

>💡 Some of the configuration changes might take a bit to sync in all machines. Therefore, if you don't see the effect immediately, it is recommended to wait 5 to 15 minutes before attempting further changes.

This guide covers essential tasks for LDAP management, from accessing your admin account to advanced configurations, utilizing both **CLI** and **web browser interface**.

## Using CLI

Accessing Your Admin Account

**Retrieve Admin User and Password:**

- Locate the admin username in Morpheus' Cypher → secret/ipaadmin\_username  
- Locate the password in Morpheus' Cypher → password/ipaadmin

![IPA Server secrets in Cypher](../images/ipa-secrets.png)

Then SSH into your LDAP instance with your admin account:

```bash
ssh -J admin@ssh-proxy-ip admin@ldap-ip
```

### Users and Groups

#### Check user info

```bash
ipa user-show <USERNAME>
```

#### Check existing groups

```bash
ipa group-find
```

#### Show group information

```bash
ipa group-show <GROUP_NAME>
```

#### Verify in which the group is the user

```bash
ipa group-find --user=<USERNAME>
```

#### Add a new user to a group

```bash
ipa group-add-member <GROUP_NAME> --users=<USERNAME>
```

#### Reset password for user

```bash
ipa passwd <USERNAME> # and interactively add a new password
```

#### Add ssh keys for user

```bash
ipa user-mod <USERNAME> --sshpubkey="ssh-rsa 12345abcde=ipaclient.example.com"

ipa user-mod <USERNAME> --sshpubkey="$(cat /home/alice/.ssh/id_rsa.pub)"

# To upload multiple keys, pass a comma-separated list of keys with a single --sshpubkey option:  
# --sshpubkey="12345abcde==,key2==,key3=="
```

#### List active users

```bash
ipa user-find --all
or 
ipa user-find --all --raw | grep -iE '(dn:|krbLastSuccessfulAuth)' | cut -d ',' -f1 | cut -d: -f2 | sed -re 's/([0-9]{4})([0-9]{2})([0-9]{2})([0-9]{2})([0-9]{2})([0-9]+)Z/\3-\2-\1 \4:\5:\6/'
```

#### Add new user

```bash
ipa user-add <USERNAME> --first=<FIRSTNAME> --last=<LASTNAME> --password
```

#### Change password expiration

```bash
ipa user-mod <user>--password-expiration='2050-10-25 19:18:30Z'
```

#### Delete user

```bash
ipa user-del <USERNAME>
```

### DNS

#### DNS server forwarders

check which DNS server are present in LDAP

```bash
ipa dnsserver-find
```

#### DNS server add a forwarder

Add a forwarder DNS server to LDAP (e.g. 8.8.8.8)

```bash
ipa dnsserver-mod --forwarder=8.8.8.8
```

for multiple forwarders

```bash
ipa dnsserver-mod --forwarder=208.67.222.222 --forwarder=208.67.220.220 --forward-policy=[only|first|none] ipa02.archyslife.lan
```

### Hosts

#### List hosts registered

```bash
ipa host-find
```

#### Show host

```bash
ipa host-show <host-name>
```

#### Remove host

```bash
ipa host-del <host-name>
```

#### List DNS zone

```bash
ipa dnszone-find
```

#### List DNS records in a DNS zone

```bash
ipa dnsrecord-find <zone-name>
```

#### Remove all associated DNS records from a DNS resource in a DNS zone

```bash
ipa dnsrecord-del <zone-name> <record-name> --del-all
```

### Advanced

#### Add sudo rule to a group

##### Verify if a group exists

```bash
ipa group-find <GROUP_NAME>
```

##### Check if sudo rules exist

```bash
ipa sudorule-find
```

##### Create a sudo rule

The following sudo rules give access to everything, it can be customized with specific commands and more, see help for more information

```bash
ipa sudorule-add <SUDO_RULE_NAME> --hostcat=all --runasusercat=all --runasgroupcat=all --cmdcat=all
```

##### Add a sudo rule to a group

```bash
ipa sudorule-add-user <SUDO_RULE> --group <GROUP_NAME>
```

  

**Add User to the Group**

```bash
ipa group-add-member <GROUP_NAME> --users=<USERNAME>
```

  

**Check Group Membership**

```bash
ipa group-show <GROUP_NAME>
```

  

**Verify Sudo Rule Application**

```bash
ipa sudorule-show <SUDO_RULE_NAME>
```

  

## Using web browser interface

Accessing Your Admin Account

**Retrieve Admin User and Password:**

- Locate the admin username in Morpheus' Cypher → secret/ipaadmin\_username  
- Locate the password in Morpheus' Cypher → password/ipaadmin
  

![IPA Server secrets in Cypher](../images/ipa-secrets.png)

Then SSH into your LDAP instance with your admin account:

```bash
ssh -J admin@ssh-proxy-ip admin@ldap-ip
```

Then start the IPA Server Control Interface by running:

```bash
ipactl start
```

Now reboot the LDAP VM. Find the IP address of your remote-desktop instance and use the x2goclient to access it.

When you're in, right-click the desktop, launch a terminal, and run:

```bash
firefox http://ldap.YOURDOMAIN
```

![IPA UI access](../images/ipa-ui-access.png)

Once Firefox has launched, log in with the admin account and head over to Policy.

Create two sudo commands as shown in the screenshot below.

![IPA UI Create Commands](../images/ipa-ui-create-commands.png)

Then click `Sudo` again and create a sudo rule called `allow_all`:

![IPA UI Allow All](../images/ipa-ui-allow-all.png)

Click the command itself and check the boxes as shown below:

  
![IPA UI Allow All Inspection Part 1](../images/ipa-ui-allow-all-inspect-1.png)
![IPA UI Allow All Inspection Part 2](../images/ipa-ui-allow-all-inspect-2.png)

Update the global policy for password expiration if required :

![IPA UI Global Policy](../images/ipa-ui-global-policy.png)
  

After updating the password policy, reset the admin password so it adheres to the new policy .

Update the default shell for users to bash instead of sh:

  
![IPA UI Default Shell](../images/ipa-ui-default-shell.png)

This guide is designed to help you efficiently manage LDAP tasks using both CLI and UI interfaces, ensuring a smooth and secure administration process.