# OpenBSD relayd reverse proxy

## Forward per url request
The following simplified example forwards urls to differrent backend servers
based on the Host requested.

```
table <service> { 127.0.0.1 }
table <someappservice> { 127.0.0.1 }

http protocol "protsomeapp" {
  match request quick header "Host" value "someapp.mydomain.*" forward to <someappservice>
}

relay "someapp" {
  listen on 0.0.0.0 port 80
  protocol "protsomeapp"

  forward to <service> port 8080
  forward to <someappservice> port 8081
}
```
