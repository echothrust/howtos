# curl examples for a variety of purposes


## Cloudflare update DNS through API
```sh
curl -X PUT "https://api.cloudflare.com/client/v4/zones/<ZONE_IDENTIFIER>/dns_records/<RECORD_ID>" \
     -H "X-Auth-Email: user@example.com" \
     -H "X-Auth-Key: c2547eb745079dac9320b638f5e225cf483cc5cfdda41" \
     -H "Content-Type: application/json" \
     --data '{"type":"A","name":"example.com","content":"127.0.0.1","ttl":{},"proxied":false}'
```
