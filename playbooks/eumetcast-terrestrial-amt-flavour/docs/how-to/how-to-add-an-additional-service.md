# How to add an additional terrestrial service

`ter-1`, `ter-2` and `ter-3` are enabled by default when you deploy the EUMETCast Terrestrial on AMT flavour.

Use this guide only when EUMETSAT announces a new terrestrial service (`ter-4`, `ter-5`, etc.) in the future.

> ⚠️ After any configuration change, restart the client with `sudo systemctl restart tellicast-client`.

1. Update your subscription in the [EUMETSAT User Portal](https://user.eumetsat.int/).

2. Log into the reception station (via SSH).

3. Edit the main configuration file `/etc/tellicast-client.cfg` with your text editor of choice, by appending the new service, for example `ter-4`, to the start order parameter:
   ```
   INSTANCE_START_ORDER=ter-1,ter-2,ter-3,ter-4
   ```
   Save the file before existing the editor.

5. Copy or create the required configuration files for the new service (`cast-client_ter-4.ini` and `cast-client-channels_ter-4.ini`) in `/etc/`, following the pattern of the existing ter-X files.

6.  Insert your credentials (`user_name` and `user_key`) and the correct `interface_address`.

7. (Optional) Adjust the storage path in `cast-client-channels_ter-4.ini` if you do not want to use the default location under `/home/eumetuser/data`.

8. Restart the Tellicast client:
   ```bash
   sudo systemctl restart tellicast-client
   ```
9. Verify the changes by inspecting the log entries for the new service. They should appear in `/var/log/tellicast-client/recv_ter-4.log`.

**Resources**
- [How to monitor and check connectivity of the EUMETCast client](./how-to-monitor-and-check-connectivity-of-the-eumetcast-client.md)
- [How to change the file storage path](./how-to-change-file-storage-path.md)
- EUMETSAT User Helpdesk: ops@eumetsat.int


