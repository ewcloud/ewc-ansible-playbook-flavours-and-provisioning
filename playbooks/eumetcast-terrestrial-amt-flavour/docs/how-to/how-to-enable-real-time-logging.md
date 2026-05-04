# How to enable real-time logging

By default, log output from the Tellicast client is buffered. To see events in real time:

1. Log into the instance where EUMETCast Terrestrial on AMT Item has been successfully installed.

2. Edit the configuration file for the relevant service, for example `/etc/cast-client_ter-1.ini`, with your text editor of choice

3. Change the `log_file` line so it does **not** contain the double "grater than" character (`>>`) (i.e. remove any redirection):
   ```
   log_file=/var/log/tellicast-client/recv_ter-1.log
   ```

4. Save the file and restart the service:
   ```bash
   sudo systemctl restart tellicast-client
   ```

> ⚠️ Direct edits to Ansible-managed files may be overwritten if you re-run the flavour playbook. Repeat the change for every ter-X service you want to monitor in real time.

**Resources**
- [How to monitor and check connectivity of the EUMETCast client](./how-to-monitor-and-check-connectivity-of-the-eumetcast-client.md)

