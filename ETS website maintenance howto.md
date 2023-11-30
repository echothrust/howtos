---
---

# Echothrust Solutions website maintenance HOWTO

This is a small paper on a method we employ in order to perform scheduled maintenance on our web servers.

The guide assumes you're running `OpenBSD` and `nginx` but the principles can be applied to a variety of applications and services.

The method as its pros and cons but they depend on your own requirements and limitations. In our case we are quite happy with this method and we've been using ever since.
* It allows you to stop the web server for your site completely so you can perform updates. But this comes at the penalty of having to maintain an extra service
* It is fairly simple to "script" and automate
* It assumes that both web services run on the same host, if you dont have this limitation then other methods may be better

## The idea
The idea is as the following:
* Setup two different nginx servers on of them listens on ports 80,443 and the other on localhost:8080 and localhost:8443
* The server listening on localhost, serves a single static html file for every request and returns status 503 for every request
* We setup a pf table called `maintenance` which is initially empty
* We redirect any request from addresses found in `maintenance` towards ports 80 and 443 to our nginx listening on localhost at 8080 and 8443 respectively
* We allow the allow any remaining requests to access the nginx instance listening on our public interface

## Configuration

* Configure pf
```
table <maintenance> persist counters { }
pass in quick inet proto tcp from <maintenance> to port 80 rdr-to port 8080 label "www-maintenance"
pass in quick inet proto tcp from <maintenance> to port 443 rdr-to port 8443 label "www-maintenance"
pass in quick inet proto tcp to port {80,443} label "www-normal"
```

* Configure nginx that will be used when we are under maintenance, we use `/etc/nginx/maintenance.conf`
```
server {
  listen       localhost:8080;
  listen       localhost:8443 ssl;
  server_name  SERVER_NAME;
  root         /var/www/maintenance;

  ssl_certificate /etc/ssl/acme/fullchain.pem;
  ssl_certificate_key /etc/ssl/acme/private/privkey.pem;
  ssl_session_timeout 5m;
  ssl_session_cache shared:SSL:50m;
  ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
  ssl_ciphers 'EECDH+AESGCM:EDH+AESGCM:AES256+EECDH:AES256+EDH';
  ssl_prefer_server_ciphers on;
  ssl_dhparam /etc/ssl/private/dhparam.pem;

  return 503;
  error_page   503  @maintenance;

  location @maintenance {
      rewrite ^(.*)$ /maintenance.html break;
  }
}
```

* copy the nginx service so that it can be managed indipendently
```sh
cp /etc/rc.d/nginx /etc/rc.d/nginx_maintenance
rcctl enable nginx_maintenance
rcctl set nginx_maintenance flags -c /etc/nginx/maintenance.conf
rcctl start nginx_maintenance
```
Add your `maintenance.html` under `/var/www/html` or any other location you picked and you're good to give it a go.

## Setting your site in maintenance mode
Now in when we want to add our website under maintenance you can simply do
```sh
pfctl -t maintenance -T add 0.0.0.0/0
```

In order to kill any existing connections to the live nginx we use our defined label
```sh
pfctl -k label -k www-normal
```

At this point any request towards our normal webserver should display our nice `maintenance.html`.

## Setting your site back online
In order to finish the maintenance we first have to empty the `maintenance` table
```sh
doas pfctl -t maintenance -T flush
```

And again we kill any existing sessions that are left open at the maintenance nginx
```sh
doas pfctl -k label -k www-maintenance
```
