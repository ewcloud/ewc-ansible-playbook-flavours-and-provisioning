# How to add a new backend server to HAProxy

## Initial configuration
After the machine is deployed, your initial configuration file is located at `/opt/ha-proxy-certbot/haproxy/config` and it contains the following structure:

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

* `defaults` section, which defines settings that are shared across all sections that follow. It sets (These are just good safety measures that will help avoid common problems.):
    * timeouts for how long HAProxy should wait for a client to send data (`timeout client`),
    * how long to wait when trying to connect to a backend server (`timeout connect`),
    * how long to wait for the server to send back data (`timeout server`),
    * how long to wait for the client to send a complete HTTP request (`timeout http-request`). 
    * mode sets how HAProxy treats requests. It can be either `http` or `tcp`, with the former used when working with HTTP web applications. In this mode, HAProxy is able to inspect metadata of an HTTP request, such as headers, cookies, and the URL path, when deciding how to route the request.


The load balancer is now listening on the localhost address, 127.0.0.1, at port 80. If you make a request using `curl`, you’ll get back a response:

```bash
curl 0.0.0.0:80
<html><body><h1>503 Service Unavailable</h1>
No server is available to handle this request.
</body></html>
```

Granted, there’s no reply from a server because we haven’t configured any servers yet. Nevertheless, you can see that HAProxy is functional.

## Add a backend server
> ⚠️ If you want to modify the config file, follow this procedure [https://confluence.ecmwf.int/display/EWCLOUDKB/How+to+modify+HAproxy+config+file](https://confluence.ecmwf.int/display/EWCLOUDKB/How+to+modify+HAproxy+config+file). You will find also more general info about the config and a way to validate your changes.

1. Open haproxy config file at `/opt/ha-proxy-certbot/haproxy/config` and start modifying the config

In HAProxy, a frontend receives traffic before dispatching it to a backend, which is a pool of web or application servers. One of the servers in the backend will receive the request, form a response, and then send the response back through HAProxy to the client.

In your HAProxy configuration, below the `frontend` section, add a backend section that contains a single server. Also, add a `default_backend` line to your frontend that points to this new backend. Your file now looks like this:

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
You can also add multiple backends:
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

With each new request that you make to port 80, HAProxy receives it and selects a server to handle it. HAProxy rotates through the list of servers, relaying the next request to the next server in line. You are now balancing the load across these servers.

2. After saving the changes, you need to restart the container

```bash
docker restart <CONTAINER_ID>
```

3. You can check the validity with the following command

```bash
docker exec -it <CONTAINER_ID> haproxy -c -f /config/haproxy.cfg
```

where `CONTAINER_ID`, can be found using docker ps

## Add health checks

[https://www.haproxy.com/blog/how-to-enable-health-checks-in-haproxy](https://www.haproxy.com/blog/how-to-enable-health-checks-in-haproxy)

## Resources

[https://www.haproxy.com/blog/haproxy-configuration-basics-load-balance-your-servers](https://www.haproxy.com/blog/haproxy-configuration-basics-load-balance-your-servers)