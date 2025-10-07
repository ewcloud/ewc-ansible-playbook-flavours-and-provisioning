# Add a new SSL certificate to a server in HAProxy

## Configure HaProxy

In order to request certificates you need to add letsencrypt backend exposed to port 8080 and have similar structure to the below section. Remember to configure the backend to your own server

```
#HA Proxy Config
global
  ulimit-n 500000
  maxconn 99999
  maxpipes 99999
  tune.maxaccept 500
 
  log 127.0.0.1 local0
  log 127.0.0.1 local1 notice
 
  ca-base /etc/ssl/certs
  crt-base /etc/ssl/private
 
  ssl-default-bind-ciphers ECDH+AESGCM:DH+AESGCM:ECDH+AES256:DH+AES256:ECDH+AES128:DH+AES:ECDH+3DES:DH+3DES:RSA+AESGCM:RSA+AES:RSA+3DES:!aNULL:!MD5:!DSS
  ssl-default-bind-options no-sslv3
 
defaults
  mode http
 
  timeout client 10s
  timeout connect 5s
  timeout server 10s
  timeout http-request 10s
 
frontend http-in
# Receive HTTP traffic on all IP addresses assigned to the server at port 80
  bind *:80
 
  option forwardfor
 
  http-request add-header "X-Forwarded-Proto" "http"
 
  acl letsencrypt_http_acl path_beg /.well-known/acme-challenge/
 
  redirect scheme https if !letsencrypt_http_acl { env(FORCE_HTTPS_REDIRECT) -m str true }
 
  use_backend letsencrypt_http if letsencrypt_http_acl
 
  default_backend webservers
 
 
frontend https_in
  bind *:443 ssl crt /usr/local/etc/haproxy/default.pem crt /usr/local/etc/haproxy/certs.d ciphers ECDHE-RSA-AES256-SHA:RC4-SHA:RC4:HIGH:!MD5:!aNULL:!EDH:!AESGCM
 
  option forwardfor
 
  http-request add-header "X-Forwarded-Proto" "https"
 
  default_backend webservers
 
 
backend letsencrypt_http
  server letsencrypt_http_srv 127.0.0.1:8080
 
 
backend webservers
  balance leastconn
 
  option tcp-check
  option log-health-checks
 
  server mapserver 10.0.0.96:8999 check inter 5s
  http-request set-header X-Forwarded-Port %[dst_port]
```

>⚠️ the docker compose needs to expose the correct port. Moreover the VMs needs to have the required ports open!

![](../images/haproxy.png)

## Add a new SSL certificate

This will add a new cert using a certbot config that is compatible with the haproxy config template below. After creating the cert, you should run the refresh script referenced below to initialize haproxy to use it. After adding the cert and running the refresh script, no further action is needed.

```bash
# request certificate from let's encrypt
docker exec haproxy-certbot certbot-certonly   --domain YOUR_DOMAIN  --email YOUREMAIL --dry-run
 
# create/update haproxy formatted certs in certs.d and then restart haproxy
docker exec haproxy-certbot haproxy-refresh
```

*After testing the setup, remove --dry-run to generate a live certificate*

## Resources

* [https://github.com/thingsboard/docker/blob/master/haproxy-certbot/README.md](https://github.com/thingsboard/docker/blob/master/haproxy-certbot/README.md)