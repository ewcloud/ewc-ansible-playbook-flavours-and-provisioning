# How to configure the NGINX Proxy manager initial setup

Once the VM is provisioned, you need to configure NGINX Proxy manager initially.

In order to do that you need to access the admin UI.

  

# Access NGINX Proxy Admin UI

The NGINX Proxy Manager is protected by only being accessible on the private internal network, therefore you first need to get into a position to connect to it in order to access the admin UI.

![](../images/n-p-m_home.png)

### Easy: connect via remote-desktop

If you have a remote desktop server already set up in the tenancy, connect to it (e.g. via x2go) and open a web browser (e.g. firefox). Check x2go remote desktop section here: .

At the end you should be able to access this URL from the browser: **http://<VM\_PUBLIC\_IP>:8200** or **http://<vm\_name>.<tenancy\_name>.<host\_parameter>ewcloud.host:8200** (for more info on the DNS record for your VM, check )

  

## Initial setup

When the NGINX Proxy Manager first starts, log in with the following username and password:

*   Default Proxy Manager username: **[admin@example.com](mailto:admin@example.com)**
*   Default Proxy Manager password: **changeme**

The default NGINX Proxy manager username and password can only be used once. When you log in, you will be asked to update and change your credentials.

Now you are ready to create proxy host: [How to add a new proxy host for specific domain](../how-to/how-to-add-a-new-proxy-host-for-specific-domain.md)