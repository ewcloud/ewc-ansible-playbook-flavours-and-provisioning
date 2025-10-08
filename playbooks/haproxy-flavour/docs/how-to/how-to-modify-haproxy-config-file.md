# How to Modify the HAProxy Configuration File

This guide provides steps to safely modify the HAProxy configuration file on the European Weather Cloud, ensuring changes are applied correctly and validated to avoid service disruptions.

## HAProxy Configuration File Location

The HAProxy configuration file is located at `/opt/ha-proxy-certbot/haproxy/config/haproxy.cfg` on the virtual machine (VM).

### Key Sections of the Configuration File
An HAProxy configuration file typically includes four essential sections:
- **Global**: Defines global settings, such as system limits, logging, and SSL parameters.
- **Defaults**: Sets default parameters for timeouts, modes (e.g., HTTP or TCP), and other behaviors applied to all frontends and backends unless overridden.
- **Frontend**: Configures how HAProxy receives client requests, including the ports and protocols it listens on.
- **Backend**: Specifies the pool of servers to which HAProxy routes traffic, including load balancing and health check settings.

For a detailed explanation, see [The Four Essential Sections of an HAProxy Configuration](https://www.haproxy.com/blog/the-four-essential-sections-of-an-haproxy-configuration#what-about-listen).

## Modify the Configuration File

Follow these steps to update the HAProxy configuration file:

1. **Gain root access** to the VM to edit the configuration file:

   ```bash
   sudo su
   ```

2. **Open and edit the configuration file** using a text editor (e.g., `vi` or `nano`):

   ```bash
   vi /opt/ha-proxy-certbot/haproxy/config/haproxy.cfg
   ```

   Make the necessary changes and save the file.

3. **Verify the configuration file** to ensure there are no syntax errors:

   ```bash
   docker exec -it <CONTAINER_ID> haproxy -c -f /config/haproxy.cfg
   ```

   Replace `<CONTAINER_ID>` with the actual container ID, which you can find by running:

   ```bash
   docker ps
   ```

   If the configuration is valid, you will see a message indicating success (e.g., `Configuration file is valid`). If errors are detected, review and correct them before proceeding.

4. **Restart the HAProxy container** to apply the changes:

   ```bash
   docker restart <CONTAINER_ID>
   ```

   Ensure `<CONTAINER_ID>` matches the ID from the `docker ps` command.

> ⚠️ Always validate the configuration before restarting the container to prevent downtime due to invalid settings. If the configuration is invalid, HAProxy may fail to start.

## Best Practices
- **Backup the configuration file** before making changes:

   ```bash
   cp /opt/ha-proxy-certbot/haproxy/config/haproxy.cfg /opt/ha-proxy-certbot/haproxy/config/haproxy.cfg.bak
   ```

- **Test changes in a non-production environment** if possible, to avoid impacting live services.
- **Document changes** to track modifications and their purpose, especially in a team environment.
- **Monitor logs** after restarting to confirm HAProxy is running correctly:

   ```bash
   docker logs <CONTAINER_ID>
   ```

## Resources
- [The Four Essential Sections of an HAProxy Configuration](https://www.haproxy.com/blog/the-four-essential-sections-of-an-haproxy-configuration)
- [HAProxy Configuration Manual](https://www.haproxy.com/documentation/haproxy/configuration/)