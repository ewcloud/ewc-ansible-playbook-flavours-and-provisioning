# How to sync EUMETCast data to Shared File System (SFS)

You can synchronise received data to a mounted Shared File System (SFS) using [rsync](https://linux.die.net/man/1/rsync).

1. Set up and mount the SFS share to a local directory (e.g. `/mountpoint`) following the EWC Shared File System documentation.

2. Create a target directory on the share:
   ```bash
   mkdir /mountpoint/eumetcast-data
   ```

3. Edit root’s crontab:
   ```bash
   sudo crontab -e
   ```

   Add a line similar to:
   ```
   */1 * * * * rsync -rt --delete /home/eumetuser/data/ter-1/default/ /mountpoint/eumetcast-data/
   ```
   > ⚠️ Adjust permissions on the target directory if your applications need specific access rights. Change the source path if you modified the default storage location.
   
   The above command ensures changes done locally are always synced to the remote (`local` → `/home/eumetuser/data/ter-1/default/` OR `remote → /mountpoint/eumetcast-data`):
   * files added in local are added to remote
   * files removed from local are removed from remote
   * files added in remote are removed
   * files removed from remote are restored from local if they exist, else ignored

**Resources**
- [How to sync EUMETCast data to Object Storage (S3)](./how-to-sync-eumetcast-data-to-object-storage-s3.md)
