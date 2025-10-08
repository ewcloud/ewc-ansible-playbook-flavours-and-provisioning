# How to Connect to a VM Hidden Behind the SSH Bastion

This guide explains how to securely connect to a virtual machine (VM) on the European Weather Cloud that is hidden behind an SSH bastion host. The bastion acts as a secure gateway, allowing access to internal VMs that lack direct public IP addresses.

## Connect via SSH Using the Bastion

To connect to a VM behind the SSH bastion, use the `-J` (jump) flag with the `ssh` command to route through the bastion host:

```bash
ssh -J user@ssh-proxy user@internal-vm
```

- **user**: Your username for both the bastion and the target VM.
- **ssh-proxy**: The public IP address or hostname of the SSH bastion host.
- **internal-vm**: The private IP address or hostname of the target VM within the internal network.

For example, to connect to a VM at `10.0.0.5` via a bastion at `203.0.113.10`:

```bash
ssh -J user@203.0.113.10 user@10.0.0.5
```

> ⚠️ Ensure your SSH key is configured for both the bastion and the target VM. You may need to specify the key file using the `-i` flag (e.g., `ssh -i ~/.ssh/id_rsa -J ...`).

## Create SSH Tunnels for Secure Access

You can use the bastion to create SSH tunnels, enabling secure access to services running on the internal VM. There are two common approaches:

### 1. Tunnel Specific Ports
Use the `-L` option to forward a specific port from your local machine to a service on the internal VM via the bastion:

```bash
ssh -J user@ssh-proxy -L LOCAL_PORT:internal-vm:DESTINATION_PORT user@internal-vm
```

- **LOCAL_PORT**: The port on your local machine that will forward traffic.
- **DESTINATION_PORT**: The port on the internal VM where the service is running.

For example, to access a web server running on port 8080 on `10.0.0.5` through the bastion at `203.0.113.10`, binding to local port 8080:

```bash
ssh -J user@203.0.113.10 -L 8080:10.0.0.5:8080 user@10.0.0.5
```

Then, access the service locally at `http://localhost:8080`.

### 2. Create a Dynamic SOCKS5 Proxy
Use the `-D` option to create a dynamic [SOCKS5](https://en.wikipedia.org/wiki/SOCKS) proxy, allowing flexible connections to multiple services on the internal VM:

```bash
ssh -J user@ssh-proxy -D LOCAL_PORT user@internal-vm
```

- **LOCAL_PORT**: The port on your local machine that will act as the SOCKS5 proxy.

For example, to set up a SOCKS5 proxy on local port 1080:

```bash
ssh -J user@203.0.113.10 -D 1080 user@10.0.0.5
```

Configure your browser or application to use the SOCKS5 proxy at `localhost:1080` to route traffic through the bastion to the internal VM.

## Resources
- [OpenSSH Manual](https://man.openbsd.org/ssh)
- [SOCKS5 Proxy Overview](https://en.wikipedia.org/wiki/SOCKS)