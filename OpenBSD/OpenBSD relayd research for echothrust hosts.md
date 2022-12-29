# OpenBSD relayd research for echothrust hosts
The following document will outline the findings with regards to our research
on the subject of utilizing `relayd` for infrastructure. The needs for usage in
particularly on `puffy.echothrust.com` in order to serve `www.echothrust.com`,
`gitlab.echothrust.com` for both HTTP and HTTPS accelerator along with the
ability to serve a separate content on cases where a downtime is required.

At the same time we want to filter certain requests based on protocol specific
criteria (eg when the `Host` http header is not set, deny specific URL patterns
and other such cases).

**NOTE**: The configuration directives `interval`, `timeout` and `prefork` must
be before the table definitions otherwise they might not work. This is as of
16/03/2016 (relayd - prefork option seems to be ignored).

Relayd makes the following distinctions with regards to the relaying capabilities:

* **`Redirections`** Redirections are translated to pf(4) rdr-to rules for stateful forwarding to a target host from a health-checked table on layer 3.
* **`Relays`** Relays allow application layer load balancing, TLS acceleration, and general purpose TCP proxying on layer 7.
* **`Routers`** Routers are used to insert routes with health-checked gateways for (WAN) link balancing.

## Preparing for relayd
You'd have to pump up the your `ulimit` a bit in order for `relayd` to be able
to start. The best option is to create a specific section on your
`/etc/login.conf` for the daemon to use.

Depending on your requirements you may need to tweak this a bit
```
relayd:\
        :openfiles-cur=1024:\
        :openfiles-max=2048:\
        :tc=daemon:
```

Futhermore for SSL off-loading to be work we will have to create a private key
and a certificate for each individual IP's that relay will serve.

The file locations are `/etc/ssl/private/IP:port.key` and `/etc/ssl/IP.ctt`

## Configuring www.echothrust.com

```
ext_if="re0"
www_echothrust_com="172.20.3.240"
web_echothrust_com="10.5.0.2"

gitlabpub_echothrust_com="172.20.3.240"
gitlab_echothrust_com="10.5.0.3"

interval 10
timeout 1000
prefork 10

table <www_echothrust_com> { $web_echothrust_com }
table <gitlab_echothrust_com> { $gitlab_echothrust_com }
table <fallback> disable { 127.0.0.1 }

http protocol www_echothrust_com_filter {
        include "/etc/www_echothrust_com_filters-relayd.conf"

        # Various TCP performance options
        tcp { nodelay, sack, socket buffer 65536, backlog 128 }
}

http protocol www_echothrust_com_sslfilter {
        include "/etc/www_echothrust_com_filters-relayd.conf"
        match request header set "HTTPS"         value "on"
        match request header append "X-Forwarded-Proto" value "https"

        # Various TCP performance options
        tcp { nodelay, sack, socket buffer 65536, backlog 128 }

        ssl { no sslv2, sslv3, tlsv1, ciphers HIGH }
        ssl session cache disable

}

relay www_echothrust_com_proxy {
        listen on $www_echothrust_com port 8880
        protocol www_echothrust_com_filter

        forward to <www_echothrust_com> port http check http "/" code 200
        forward to <fallback> port http check http "/" code 200
}

relay www_echothrust_com_ssl {
        # Run as a SSL accelerator
        listen on $www_echothrust_com port 443 ssl
        protocol www_echothrust_com_sslfilter

        forward to <www_echothrust_com> port http check http "/" code 200
        forward to <fallback> port http check http "/" code 200
}

http protocol gitlab_echothrust_com_sslfilter {
        match request header set "HTTPS"         value "on"
        match request header append "X-Forwarded-Proto" value "https"

        # Various TCP performance options
        tcp { nodelay, sack, socket buffer 65536, backlog 128 }

        ssl { no sslv2, sslv3, tlsv1, ciphers HIGH }
        ssl session cache disable
}


relay gitlab_echothrust_com_ssl {
        # Run as a SSL accelerator
        listen on $gitlabpub_echothrust_com port 443 ssl
        protocol gitlab_echothrust_com_sslfilter

        forward to <gitlab_echothrust_com> port http check http "/" code 200
        forward to <fallback> port http check http "/" code 200
}
```

The contents of the file "/etc/www_echothrust_com_filters-relayd.conf" has the following contents
```
        # Return HTTP/HTML error pages to the client
        return error style "body { background: #e7e9ec; color:#737373; font:normal 13px/21px \"Droid Sans\", sans-serif; }\nhr { border: 0; border-bottom: 1px dashed; }"

        match request label "Invalid Host"
        pass request quick header "Host" value "echothrust.com"
        pass request quick header "Host" value "www.echothrust.com"
        block request quick header "Host" value "*"

        # Block disallowed sites
        match request label "URL filtered!"
        block request quick url "www.echothrust.com/wp-login.php" value "*"
        block request quick path "/wp-login.php" value "*"

        # Add forwarder-for and by header details
        match request header append "X-Forwarded-For"   value "$REMOTE_ADDR"
        match request header append "X-Forwarded-By"    value "$SERVER_ADDR:$SERVER_PORT"
        match request header append "X-Real-IP"         value "$REMOTE_ADDR"

        match request header set "Connection"           value "close"
        match request header "Keep-Alive"               value "$TIMEOUT"
```

# Managing fallback
In order to initiate a downtime period you'll have to disable the primary table
`<www_echothrust_com>` and activate the fallback.

```
relayctl table disable www_echothrust_com:80
relayctl table disable www_echothrust_com:443
relayctl table enable fallback:80
relayctl table enable fallback:443
relayctl poll
```

# Modify the error pages
In order to modify your return pages (the error pages that relayd sends back to
the client in case of an error) you have to modfy the following details

* `relayd/relay_http.c` modify the function `relay_abort_http` where the code
reads like this
```
	/* A CSS stylesheet allows minimal customization by the user */
	style = (rlay->rl_proto->style != NULL) ? rlay->rl_proto->style :
	    "body { background-color: #a00000; color: white; font-family: "
	    "'Comic Sans MS', 'Chalkboard SE', 'Comic Neue', sans-serif; }\n"
	    "hr { border: 0; border-bottom: 1px dashed; }\n";

	/* Generate simple HTTP+HTML error document */
	if ((bodylen = asprintf(&body,
	    "<!DOCTYPE html>\n"
	    "<html>\n"
	    "<head>\n"
	    "<title>%03d %s</title>\n"
	    "<style type=\"text/css\"><!--\n%s\n--></style>\n"
	    "</head>\n"
	    "<body>\n"
	    "<h1>%s</h1>\n"
	    "<div id='m'>%s</div>\n"
	    "<div id='l'>%s</div>\n"
	    "<hr><address>%s at %s port %d</address>\n"
	    "</body>\n"
	    "</html>\n",
	    code, httperr, style, httperr, text,
	    label == NULL ? "" : label,
	    RELAYD_SERVERNAME, hbuf, ntohs(rlay->rl_conf.port))) == -1)
		goto done;

```

* `relayd/relayd.h` modify the line
```
#define RELAYD_SERVERNAME	"OpenBSD relayd"
```

## Per url relay
```
table <web0> { 10.0.0.0 }
table <web1> { 10.0.0.1 }
table <web2> { 10.0.0.2 }
table <web3> { 10.0.0.3 }

http protocol echolab {
	return error
	pass
	match request path "/web1*" forward to <web1>
	match request path "/web2*" forward to <web2>
	match request path "/web3*" forward to <web3>
}

relay echolab {
	listen on $www_echothrust_com port 80
	protocol echolab

	# Main server table
	forward to <web0> check tcp port 80

	# Additional server tables used by custom rules
	forward to <web1> check tcp port 80
	forward to <web2> check tcp port 80
	forward to <web3> check tcp port 80
}
```


## login.conf
relayd:\
        :maxproc-max=31:\
        :openfiles-cur=16384:\
        :openfiles-max=65536:\
        :tc=daemon:
