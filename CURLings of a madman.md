# curl examples for a variety of purposes


## Cloudflare update DNS through API
```sh
curl -X PUT "https://api.cloudflare.com/client/v4/zones/<ZONE_IDENTIFIER>/dns_records/<RECORD_ID>" \
     -H "X-Auth-Email: user@example.com" \
     -H "X-Auth-Key: c2547eb745079dac9320b638f5e225cf483cc5cfdda41" \
     -H "Content-Type: application/json" \
     --data '{"type":"A","name":"example.com","content":"127.0.0.1","ttl":{},"proxied":false}'
```

## Docker Registry
* Get catalog listing
```sh
curl http://<registry:port>/v2/_catalog
```

* Get image tags
```sh
curl http://<registry:port>/v2/<IMAGENAME>/tags/list
```

* Get manifests
```sh
curl -fsSL \
	-H 'Accept: application/vnd.docker.distribution.manifest.v2+json' \
	-H 'Accept: application/vnd.docker.distribution.manifest.list.v2+json' \
	-H 'Accept: application/vnd.docker.distribution.manifest.v1+json' \
	"http://<registry:port>/v2/<IMAGENAME>/manifests/<DIGEST|TAG>"
```
