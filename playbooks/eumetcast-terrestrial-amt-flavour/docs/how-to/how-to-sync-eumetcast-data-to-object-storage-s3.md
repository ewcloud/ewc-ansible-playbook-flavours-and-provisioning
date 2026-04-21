# How to sync EUMETCast data to Object Storage  (S3)

> ✅ Throught this example, we will use [s3cmd](https://s3tools.org/s3cmd) as tool for S3 oject storage management. However, common valid alternatives include the [awscli](https://docs.aws.amazon.com/cli/latest/reference/s3/) and [rclone](https://rclone.org/).

You can automatically synchronise received data to Object Storage.

1. Install and configure `s3cmd` by following the guidelines in the [EWC Knowledge Base](https://confluence.ecmwf.int/x/1LjLCg).

2. Create a bucket:
   ```bash
   s3cmd mb s3://eumetcast-example
   ```

3. Copy your `.s3cfg` file to a system-wide location:
   ```bash
   sudo cp ~/.s3cfg /etc/s3cfg
   ```

4. Edit root’s crontab:
   ```bash
   sudo crontab -e
   ```

   Add a line similar to the following (adjust the source path if you changed the storage location):
   ```
   */5 * * * * s3cmd -c /etc/s3cfg sync /home/eumetuser/data/ter-1/default/ s3://eumetcast-example/ --delete-removed
   ```

> 💡 Adjust the cron interval and source sub-directory according to your needs and the actual data layout under `/home/eumetuser/data`.

**Resources**
- [How to sync EUMETCast data to Shared File System (SFS)](./how-to-sync-eumetcast-data-to-shared-file-system-sfs.md)
