# How to add an additional terrestrial service

`ter-1`, `ter-2`, `ter-3` `ter-4` and `ter-5` are enabled by default when you deploy the EUMETCast Terrestrial on AMT flavour.

Use this guide only when:
* You are unable to collect data from already available services

   **OR**

* EUMETSAT announces a new terrestrial services over the [EUMETSAT User Portal](https://user.eumetsat.int/news-events)

## Update your subscription
> ⚠️ Subscription changes become active on the next working day

1. Log in to the [EUMETSAT User Portal](https://user.eumetsat.int/dashboard) using your credentials

2. Navigate to the EUMETCast Terrestrial service and subscribe to the data channels required. 


## Update your reception station

1. Log into the reception station (via SSH).

2. Edit the main configuration file `/etc/tellicast-client.cfg` with your text editor of choice, by appending the new service, for example `ter-4` and `ter-5`, to the start order parameter:
   ```
   INSTANCE_START_ORDER=ter-1,ter-2,ter-3,ter-4,ter-5
   ```
   Save the file before existing the editor.

3. Copy or create the required configuration files for the new service (`cast-client_ter-4.ini`, `cast-client-channels_ter-4.ini` `cast-client_ter-5.ini` and `cast-client-channels_ter-5.ini`) in `/etc/`, following the pattern of the existing ter-X files.

4.  Insert your credentials (`user_name` and `user_key`) and the correct `interface_address`.

5. (Optional) Adjust the storage path in `cast-client-channels_ter-4.ini` and `cast-client-channels_ter-5.ini` if you do not want to use the default location under `/home/eumetuser/data`.

6. Restart the Tellicast client:
   ```bash
   sudo systemctl restart tellicast-client
   ```
7. Verify the changes by inspecting the log entries for the new service. They should appear in `/var/log/tellicast-client/recv_ter-4.log` and `/var/log/tellicast-client/recv_ter-5.log`

**Resources**
- [How to monitor and check connectivity of the EUMETCast client](./how-to-monitor-and-check-connectivity-of-the-eumetcast-client.md)
- [How to change the file storage path](./how-to-change-file-storage-path.md)
- [EUMETSAT User Helpdesk](mailto:ops@eumetsat.int)


