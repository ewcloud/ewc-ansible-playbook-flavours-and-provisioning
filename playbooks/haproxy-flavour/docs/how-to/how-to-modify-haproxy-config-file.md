# How to modify HAproxy config file

## HAProxy Config

The configuration file is stored on the VM under **/opt/ha-proxy-certbot/config/haproxy.cfg**

1.  **Become root** to modify it.
2.  Open the file, modify and save it.
3.  Check the container ID
```bash
docker ps
```
4.  Restart container for the changes to take effect
```bash
docker restart <CONTAINER_id>
```

There are four essential sections to an HAProxy configuration file. They are `global`, `defaults`, `frontend`, and `backend`. These four sections define how the server as a whole performs, what your default settings are, and how client requests are received and routed to your backend servers.

Check here: [https://www.haproxy.com/blog/the-four-essential-sections-of-an-haproxy-configuration#what-about-listen](https://www.haproxy.com/blog/the-four-essential-sections-of-an-haproxy-configuration#what-about-listen)

  

## Check if configuration file is valid

```bash
docker exec -it <CONTAINER_ID> haproxy -c -f /config/haproxy.cfg
```