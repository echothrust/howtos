---
---

# Echothrust Solutions website maintenance HOWTO

This is a small howto on how we perform scheduled maintenance on our websites that require a database update or re-import.

The guide assumes you're running OpenBSD and nginx but the principles can be applied to a variety of applications and services.

* Configure pf
```
table <maintenance> persist counters { }
pass in quick inet proto tcp from <maintenance> to port 80 rdr-to port 8080 label "www-maintenance"
pass in quick inet proto tcp from <maintenance> to port 443 rdr-to port 8443 label "www-maintenance"
pass in quick inet proto tcp to port {80,443} label "www-normal"
```
* Configure nginx
 ```
 server {
        listen       8080;
        listen       8443 ssl;
        server_name  SERVER_NAME;
        root         /var/www/maintenance;

        # include acme-client.conf;
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

Now in when you want to add your website under maintenance you can simply do
```sh
doas pfctl -t maintenance -T add 0.0.0.0/0
doas pfctl -k label -k www-normal # flush existing sessions
```

Remove website from maintenance
```sh
doas pfctl -t maintenance -T flush
doas pfctl -k label -k www-maintenance
```
