# How to Add a New Backend Server to HAProxy

## Initial Configuration
After the machine is deployed, your initial configuration file is located at `/opt/ha-proxy-certbot/haproxy/config`. It contains the following structure:

```
defaults
  mode http
  timeout client 10s
  timeout connect 5s
  timeout server 10s
  timeout http-request 10s
 
frontend myfrontend
  bind *:80
```

This example includes:
* A `defaults` section, which defines settings that are shared across all sections that follow. It sets the following (these are good safety measures that help avoid common problems):
    * `timeout client`: How long HAProxy should wait for a client to send data.
    * `timeout connect`: How long to wait when trying to connect to a backend server.
    * `timeout server`: How long to wait for the server to send back data.
    * `timeout http-request`: How long to wait for the client to send a complete HTTP request.
* `mode`: Sets how HAProxy treats requests. It can be either `http` or `tcp`, with the former used when working with HTTP web applications. In this mode, HAProxy can inspect metadata of an HTTP request, such as headers, cookies, and the URL path, when deciding how to route the request.

The load balancer is now listening on the localhost address (127.0.0.1) at port 80. If you make a request using `curl`, you’ll get back a response:

```bash
curl 127.0.0.1:80
<html><body><h1>503 Service Unavailable</h1>
No server is available to handle this request.
</body></html>
```

Note that there’s no reply from a server because no servers are configured yet. However, this confirms that HAProxy is functional.

## Add a Backend Server
> ⚠️ If you want to modify the config file, follow this procedure: [How to modify HAproxy config file](../how-to/how-to-modify-haproxy-config-file.md). You will also find more general info about the config and a way to validate your changes.

In HAProxy, a frontend receives traffic before dispatching it to a backend, which is a pool of web or application servers. One of the servers in the backend will receive the request, form a response, and then send the response back through HAProxy to the client.

1. Open the HAProxy config file at `/opt/ha-proxy-certbot/haproxy/config` and modify it.

   Below the `frontend` section, add a `backend` section that contains a single server. Also, add a `default_backend` line to your frontend that points to this new backend. Your file now looks like this:

   ```
   defaults
     mode http
     timeout client 10s
     timeout connect 5s
     timeout server 10s
     timeout http-request 10s
    
   frontend myfrontend
     bind *:80
     default_backend myservers
    
   backend myservers
     server server1 127.0.0.1:8000
   ```

   You can also add multiple backends for load balancing:

   ```
   defaults
     mode http
     timeout client 10s
     timeout connect 5s
     timeout server 10s
     timeout http-request 10s
    
   frontend myfrontend
     bind *:80
     default_backend myservers
    
   backend myservers
     server server1 127.0.0.1:8000
     server server2 127.0.0.1:8001
     server server3 127.0.0.1:8002
   ```

   With each new request to port 80, HAProxy receives it and selects a server to handle it. By default, HAProxy rotates through the list of servers in a round-robin fashion, balancing the load across them.

2. After saving the changes, restart the container:

   ```bash
   docker restart <CONTAINER_ID>
   ```

   (Replace `<CONTAINER_ID>` with the actual ID, which you can find using `docker ps`.)

3. Validate the configuration with the following command:

   ```bash
   docker exec -it <CONTAINER_ID> haproxy -c -f /config/haproxy.cfg
   ```

## Add Health Checks
To ensure HAProxy only routes traffic to healthy servers, add health checks to your backend section. This prevents sending requests to failed or unresponsive servers.

For example, update your `backend` section like this:

```
backend myservers
  option httpchk
  http-check send meth GET uri /health
  server server1 127.0.0.1:8000 check
  server server2 127.0.0.1:8001 check
  server server3 127.0.0.1:8002 check
```

* `option httpchk`: Enables HTTP health checks.
* `http-check send meth GET uri /health`: Specifies a GET request to a health endpoint (e.g., `/health`) on each server. Adjust the URI based on your application's health check path.
* `check`: Enables checking for each server.

HAProxy will periodically probe the servers and mark them as down if they fail to respond correctly. For more details, see [How to Enable Health Checks in HAProxy](https://www.haproxy.com/blog/how-to-enable-health-checks-in-haproxy).

## Resources
For more on HAProxy basics and load balancing, refer to [HAProxy Configuration Basics: Load Balance Your Servers](https://www.haproxy.com/blog/haproxy-configuration-basics-load-balance-your-servers).