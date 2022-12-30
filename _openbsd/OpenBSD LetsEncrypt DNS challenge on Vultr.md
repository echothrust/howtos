---
---

# OpenBSD LetsEncrypt DNS challenge on Vultr

## Process summary

### Install required software
```
# pkg_add certbot
# pkg_add py-pip
# ln -sf /usr/local/bin/pip2.7 /usr/local/bin/pip
# pip install dns-lexicon
# pip install urllib3
```

### Create DNS automation script
Change the `API_KEY_GOES_HERE` with vultr API key and save the follwoing script to `/etc/letsencrypt/lexicon-vultr.sh`.
```
#!/usr/bin/env ksh
/usr/local/bin/lexicon vultr \
--auth-token=API_KEY_GOES_HERE \
"$1" "${CERTBOT_DOMAIN}" TXT \
--name "_acme-challenge.${CERTBOT_DOMAIN}" \
--content "${CERTBOT_VALIDATION}" || exit 255

if [ "$1" == "create" ]; then
  sleep 30
fi
```

### Set permissions
```
# chown root:wheel /etc/letsencrypt/lexicon-vultr.sh
# chmod 700 /etc/letsencrypt/lexicon-vultr.sh
```

### Command to run for renew/creation (*this will only work if our current IP address is whitelisted for Vultr API*)

```
# certbot certonly --manual \
--manual-public-ip-logging-ok \
--manual-auth-hook "/etc/letsencrypt/lexicon-vultr.sh create" \
--manual-cleanup-hook "/etc/letsencrypt/lexicon-vultr.sh delete" \
--preferred-challenges dns \
--agree-tos \
--email info@echothrust.dev \
-d example.org -d www.example.org
```
Renew is also possible by just `certbot renew`, but this will renew all the certificates under `/etc/letsencrypt/live`.

### Certificates and key location
* /etc/letsencrypt/live/example.org/cert.pem
* /etc/letsencrypt/live/example.org/privkey.pem
* /etc/letsencrypt/live/www.example.org/chain.pem
* /etc/letsencrypt/live/www.example.org/fullchain.pem
* ...


Assuming domain `www.example.org` copy the needed files
```
cp /etc/letsencrypt/live/www.example.org/fullchain.pem /etc/nginx/www.example.com.crt
cp /etc/letsencrypt/live/example.org/privkey.pem /etc/nginx/www.example.com.key
```
