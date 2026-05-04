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
   */5 * * * * rsync -rt --delete /home/eumetuser/data/ter-1/default/ /mountpoint/eumetcast-data/
   ```

   This command keeps the remote directory synchronised with the local one (added/removed files are mirrored).

> ⚠️ Adjust permissions on the target directory if your applications need specific access rights. Change the source path if you modified the default storage location.

**Resources**
- [How to sync EUMETCast data to Object Storage (S3)](./how-to-sync-eumetcast-data-to-object-storage-s3.md)
