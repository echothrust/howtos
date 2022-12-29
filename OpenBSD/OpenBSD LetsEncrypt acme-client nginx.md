# OpenBSD LetsEncrypt acme-client nginx
## Process summary

### Configure filesystem locations

Create the directories

```
# mkdir -p /etc/acme /etc/ssl/acme/private /var/www/acme/.well-known/acme-challenge/
# chmod -R 700 /etc/acme /etc/ssl/acme/private
```

### Install acme-client (formerly letskencrypt)

Simple as download, extract, compile, install

```
# ftp https://kristaps.bsd.lv/acme-client/snapshots/acme-client.tgz
# tar zxf acme-client.tgz
# cd acme-client-XXX && make && make install
```

### Configure nginx for letsencrypt access

In `/etc/nginx/nginx.conf` add one include, before other location directives:

```
include acme-client.conf;
```

And subsequently create `/etc/nginx/acme-client.conf`:

```
# Rule for legitimate ACME Challenge requests (like /.well-known/acme-challenge/xxxxxxxxx)
# We use ^~ here, so that we don't check other regexes. We actually MUST cancel
# other regex checks, because other config files might have regex rule that denies access to files with dotted names.
location ^~ /.well-known/acme-challenge/ {

    # https://community.letsencrypt.org/t/using-the-webroot-domain-verification-method/1445/29
    # Current specification requires "text/plain" or no content header at all.
    default_type "text/plain";

    # This directory is used as prefix in -C option of acme-client(1)
    # as "challengedir" parameter, i.e. acme-client -C /var/www/acme/.well-known/acme-challenge/
    root         /var/www/acme;

}

# Hide /acme-challenge subdirectory and return 404 on all requests.
# Ending slash is important!
location = /.well-known/acme-challenge/ {
    return 404;
}
```

Write something to `/var/www/acme/.well-known/acme-challenge/test.txt` and attempt to access the URL, e.g. http:/MYDOMAIN//.well-known/acme-challenge/test.txt

### Configure PF

The server is not publicly accessible so the following was added to `/etc/pf.conf` to allow letsencrypt servers through the firewall:

```
#table <letsencrypt> persist counters { outbound1.letsencrypt.org, outbound2.letsencrypt.org }
table <letsencrypt> persist counters { 66.133.109.36, 64.78.149.164 }
pass in on egress inet proto tcp from <letsencrypt> to (egress) port { 80 443 }
```

### Initial certificate creation

```
# CERT_DOMAINS="www.domain.tld altname.domain.tld domain.tld"
CERT_DOMAINS="support.echothrust.dev"
acme-client -vNn -C /var/www/acme/.well-known/acme-challenge/ ${CERT_DOMAINS}
```

Option `-N` asks acme-client to create the private key for our web server, if one does not already exist.

Option `-n` asks acme-client to create the private key for our Let’s Encrypt account, if one does not already exist.

### Generate DH ephemeral params

nginx will use a 1024-bit key, per openssl defaults, so let's generate a stronger DHE parameter file for our 4096-bit keys:

```
cd /etc/ssl/private
openssl dhparam -out dhparam.pem 4096
```

### Adding the new cert to nginx

Add the following to `/etc/nginx/nginx.conf` to use the new certificate

```
        ssl_certificate /etc/ssl/acme/fullchain.pem;
        ssl_certificate_key /etc/ssl/acme/private/privkey.pem;
        ssl_session_timeout 1d;
        ssl_session_cache shared:SSL:50m;
        ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
        ssl_ciphers 'EECDH+AESGCM:EDH+AESGCM:AES256+EECDH:AES256+EDH';
        ssl_prefer_server_ciphers on;
        ssl_dhparam /etc/ssl/private/dhparam.pem;
```

### Maintenance

To manually check the cert validity and update (when cert expires in 30 days or less):

```
# acme-client -v -C /var/www/acme/.well-known/acme-challenge/ support.echothrust.dev
acme-client: /etc/ssl/acme/cert.pem: certificate valid: 89 days left
```

Create a script `acme-client-cron.sh` and a cronjob to automate the task:

```
#!/bin/ksh
# CERT_DOMAINS="www.domain.tld altname.domain.tld domain.tld"
CERT_DOMAINS="support.echothrust.dev"

# Make sure we allow letsencrypt's servers to connect
pfctl -t letsencrypt -T add outbound1.letsencrypt.org outbound2.letsencrypt.org > /dev/null 2>&1

# Update Lets Encrypt certificate
/usr/local/bin/acme-client -C /var/www/acme/.well-known/acme-challenge/ ${CERT_DOMAINS}

# acme-client returns 1 on failure, 2 if no_change, or 0 if cert was changed
if [ $? -eq 0 ]
then
        /usr/local/sbin/nginx -t && rcctl reload nginx
fi

# Revoke letsencrypt access to web server
pfctl -t letsencrypt -T flush > /dev/null 2>&1

```
And the relevant crontab:

```
# Update letsencrypt @ 4:20 on wednesdays
20  4  *  *  3  /usr/local/sbin/acme-client-cron.sh
```
