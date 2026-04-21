# How to modify your EUMETCast Terrestrial subscription

User management and product subscriptions are handled exclusively through the [EUMETSAT User Portal](https://user.eumetsat.int/).

1. Log in to the portal and adjust the products you want to receive for the EUMETCast Terrestrial service.

2. By default, the Tellicast client accepts all subscribed products (`name=*` in the channel configuration file).

3. If you have configured per-channel storage (commented out `name=*`), ensure the channels corresponding to your new subscription are uncommented in `/etc/cast-client-channels_ter-1.ini` and similarly for `ter-2`, `ter-3`, etc.

4. Restart the client after changes:
   ```bash
   sudo systemctl restart tellicast-client
   ```

**Resources**
- [How to change the file storage path](./how-to-change-file-storage-path.md)
