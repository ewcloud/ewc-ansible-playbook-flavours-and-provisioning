# How to connect to a VM hidden behind the SSH Bastion

To connect to a VM hidden behind the proxy, specify the jump flag -J when connecting via SSH:
```bash
ssh -J user@ssh-proxy user@internal-vm
```
Where **ssh-proxy** is the public IP of your SSH proxy, and internal-vm is the private IP of the VM you want to connect to.

You may also use this as a gateway to create SSH tunnels to securely access services running instead those instances, either by:
* Tunnelling specific ports with ssh's -L option:
```
-L LOCAL_PORT:internal-vm:DESTINATION_PORT
```
* Or creating a Dynamic [SOCKS5](https://en.wikipedia.org/wiki/SOCKS) proxy that you can use locally to connect through your internal instance
```
-D LOCAL_PORT
```