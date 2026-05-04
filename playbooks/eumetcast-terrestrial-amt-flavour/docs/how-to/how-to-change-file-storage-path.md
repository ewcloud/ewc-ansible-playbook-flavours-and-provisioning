# How to change the file storage path

By default, all EUMETCast Terrestrial data is stored under `/home/eumetuser/data` (owned by the `eumetuser` account).

To change the storage location:

1. Log into the instance where EUMETCast Terrestrial on AMT Item has been successfully installed.

2. Open the channel configuration file with your text editor of choice for the desired service, for example `/etc/cast-client-channels_ter-1.ini`

3. Modify the `target_directory` and `tmp_directory` parameters to point to the new absolute path.

   The line `name=*` stores all data (regardless of channel) in a single directory. Save the file chenges before existing.

4. (Optional) For per-channel storage, uncomment the relevant channel sections and set individual `target_directory` paths.

5.  Restart the client:
   ```bash
   sudo systemctl restart tellicast-client
   ```

> 💡 Changes made directly to the configuration files will persist until you re-run the Ansible flavour playbook with different variables.

**Resources**
- [How to add an additional terrestrial service](./how-to-add-an-additional-service.md)

