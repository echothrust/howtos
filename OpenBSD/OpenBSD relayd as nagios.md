# OpenBSD relayd as nagios

The following document will outline the idea and findings for re-purposing
`relayd` as a service and server health check mechanism.

The checking mechanism that `relayd` provides, allows us to perform the most
common of tasks such as
 * Ping hosts
 * Connect to tcp port
 * Connect to http or https host
 * Execute an external script for checks
 * Send and Expect specific data to a port allowing us to perform tests for
 non-supported protocols such as `ftp`.


## Preparing for relayd
You'd have to pump up your `ulimit` a bit in order for `relayd` to be able
to start. The best option is to create a specific section on your
`/etc/login.conf` for the daemon to use.

Depending on your requirements you may need to tweak this a bit
```
relayd:\
        :openfiles-cur=1024:\
        :openfiles-max=2048:\
        :tc=daemon:
```

Furthermore for SSL off-loading to work we will have to create a private key
and a certificate for each individual IP's that relay will serve.

The file locations are `/etc/ssl/private/IP:port.key` and `/etc/ssl/IP.ctt`

# Define protocols
We will utilize the `table <name> {}` feature of `relayd` in order to group
common checks.

The following simple host groupings have been created
* `<http_hosts>` & `<https_hosts>` tests hosts that we need to check by simple
  `HEAD / HTTP/1.1` requests through http or https. **NOTE** that the check does
  not include the `HOST` request header.
* `<icmp_hosts>` Hosts that we will perform icmp ping requests to determine
  their status. **NOTE** that `relayd` sets a host as offline if the icmp reply
  takes longer than the `timeout` defined.
* `<smtp_hosts>` Hosts that we will perform a simple TCP connect on port 25.
  **NOTE** this test does not include protocol negotiation with the mail server
  (ehlo, rcpt to, mail from etc)


```
interval 60
timeout 3000
table <http_hosts> {  web.echothrust.dev }
table <https_hosts>  { www.google.com }
table <tcp_dns_hosts> { 172.20.0.254, 172.20.1.254, 172.20.2.254, 172.20.3.254, 10.5.0.254, 10.20.0.254 }
table <udp_dns_hosts> { 172.20.0.254, 172.20.1.254, 172.20.2.254, 172.20.3.254, 10.5.0.254, 10.20.0.254 }
table <icmp_hosts> { kerberus.echothrust.dev, printer.echothrust.dev, builder.echothrust.dev }
table <smtp_hosts> { mail.echothrust.dev  }
table <gitlab> { gitlab.echothrust.dev }
table <zeus> { zeus.echothrust.dev }

relay gitlab_tests {
        listen on 127.0.0.1 port 6
        forward to <gitlab> port http check http "/" host "gitlab.echothrust.dev" code 301
        forward to <gitlab> port https check https "/" host "gitlab.echothrust.dev" code 302
        forward to <zeus> port http check http "/pub/" host "zeus.echothrust.dev" code 200
        forward to <zeus> port https check https "/" host "zeus.echothrust.dev" code 400
}

relay http_tests {
        listen on 127.0.0.1 port 1
        forward to <http_hosts> port http check http "/" code 200
}

redirect https_tests {
       listen on 127.0.0.1 port 2
       forward to <https_hosts> port https check https "/" code 200
}

relay icmp_tests {
        listen on 127.0.0.1 port 3
        forward to <icmp_hosts> check icmp
}
redirect dns_tests {
        listen on 127.0.0.1 tcp port 4
        listen on 127.0.0.1 udp port 4
        forward to <tcp_dns_hosts> port 53 check tcp
#       forward to <udp_dns_hosts> port 53 check tcp
}
redirect smtp_tests {
        listen on 127.0.0.1 tcp port 5
        forward to <smtp_hosts> port 25 check tcp
}
```
