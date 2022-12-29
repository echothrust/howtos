---
author: Pantelis Roditis
date: 22/04/2022
tags:
  - Yii2
  - NGINX
---
# Yii2 + NGINX custom selective dynamic error pages
Ok this title makes no sense so i'll try to explain as best i can.

So we have a yii2 application served by nginx and fastcgi. Now as all clean url guides instruct out there, we've configured our nginx server in such a way to catch all requests and sent them to our application following a specific format.

So this means that when a user requests `/targets` the request internally gets translated to `/index.php?r=/targets`. So far so good.

## How it begun
Now nginx also allows you to configure custom error pages and not those god forsaken white pages that kill your eyes. Lets not kid our selfs, these error pages, aesthetically, they look nothing like the rest of your site which make it look unprofessional and in some rare cases attract the wrong type of people to pay attention at our website.

So we went to configure our nginx to use our existing application logic for non existent files and other request errors. The way we did that was by something like the following
```
error_page 404 /index.php?r=$request_uri;
```

One for each of the error codes that interest us, until we hit the error `50x` error status codes. Now usually these errors mean that there is something seriously wrong with the server trying to fulfill the request and as such we dont want to redirect it to back to our PHP application but rather to a static page that will hide our failures and embarrassment 😂

So we went and did something like this
```
error_page 500 501 502 503 ... /maintenance.html;
```

Now these may seem self explanatory until you realize that these error pages are only for the ones that NGINX generates. What that means is that if you try to access the `/something/nonexistant/file.blah`, nginx will detect that this file does not exist and redirect your to the page defined by your `error_page` directive. Similarly yii when producing its own errors uses the same pattern and thus you achieve a nice error page throughout.

However, one thing that we need to keep in mind is that errors that are produced by your fastcgi applications are not processed. This is actually a feature, and allows applications to provide their own status code along with proper error pages. This is how we can show the 404 page and send the same status code as well to the browser, when a user requests a target page that does not exist.

What happens however when your application "crashes" for whatever unexpected reason?
* Say you have a bug that causes your application to crash and cant serve the error pages like it normaly does
* Say your system got rebooted and your php files got corrupted somehow
* Say you wanted to make a change on your live system (you such if you did) and you have a typo
* Say you updated your php and now its missing modules that you're using

In all those situations, you would expect that as long as your application produces a proper status code (eg `Status: 500 Application error`) for nginx to pick up and use the page we defined. But this is not what happens. You get the dreaded white page with the error in `<H1>` saying your application made a 💩

Luckily nginx has a way to do that, with the parameter `fastcgi_intercept_errors [on|off];`. This tells nginx that if our application produces any statis code larger than 300, then nginx should handle the necessary actions for this code.

## The problem is...
So you go and put that into your config and suddenly all errors have turned into white pages except the ones that go to the static html (like `maintenance.html`).

Whats worse, your nice 404 pages are now back to their ugly white ones...

WTF?

Well this is the reason this is happening:
1. User visits invalid bage (eg `/target/invalid`)
2. The php application produces `404 target page not found`
3. nginx receives a status code of 404 from your application
4. `intercept_errors` kicks in which directs nginx to try and show the error page from your application (`/index.php?r=$request_uri`)
5. but your app, still returns proper status code (404) which go back to nginx again
6. nginx thinks that the `error_page` you defined doesnt actually exist (it returned 404) and shows you the default ugly and white one

So it seems that you cant have both ways, or can you?...

## There is a way...
Define two identical blocks of fastcgi proxy pass one with intercept and the other without.
The one with the intercept runs your normal application, when an error page is produced the error page is intercepted by nginx and is redirected to the other fastcgi block without intercept for your application to show and return proper error codes.

## Show me the 💶
The following snippet provides a few extra tricks such as:
* It only allows serving of`/index.php`
* Any other php file returns 404
* Removes duplicate `//` from the url
* Rate limits certain urls and sets the status code to 429 for rate and connection limits
* Doesnt allow direct access of static error pages
* Sets static html for error codes of `50x` to `/maintenance.html`
* All 4xx error codes are served by our application (from another fastcgi block)


The following is the full server block we use for our applications
```nginx
server {
    listen                    {{item.ip}}:443 ssl;
    server_name               {{item.domain}};
    root                      {{item.root}};
    ssl_prefer_server_ciphers on;
    ssl_session_timeout       5m;

    ssl_certificate           /etc/nginx/{{item.domain}}-server.crt;
    ssl_certificate_key       /etc/nginx/{{item.domain}}-server.key;
    ssl_dhparam               /etc/ssl/private/dhparam.pem;

    ssl_ciphers               'AES256+EECDH:AES256+EDH:!aNULL';
    ssl_protocols             TLSv1 TLSv1.1 TLSv1.2;
    ssl_session_cache         builtin:1000  shared:SSL:10m;

    add_header Strict-Transport-Security "max-age=63072000; includeSubDomains" always;
    add_header X-Content-Type-Options nosniff always;
    add_header X-XSS-Protection "1; mode=block" always;
    add_header X-Frame-Options DENY always;

    error_page 400 401 402 403 404 405 406 407 408 409 @errorphp;
    error_page 410 411 412 413 414 415 416 417 418 419 @errorphp;
    error_page 420 421 422 423 424 425 426 427 428 @errorphp;
    # serve flooders with static html
    error_page 429 @maintenance;

    error_page 430 431 432 433 434 435 436 437 438 439 @errorphp;
    error_page 440 441 442 443 444 445 446 447 448 449 @errorphp;
    error_page 450 451 452 453 454 455 456 457 458 459 @errorphp;
    error_page 460 461 462 463 464 465 466 467 468 469 @errorphp;
    error_page 470 471 472 473 474 475 476 477 478 479 @errorphp;
    error_page 480 481 482 483 484 485 486 487 488 489 @errorphp;
    error_page 490 491 492 493 494 495 496 497 498 @errorphp;
    # 499 is only allowed to be used by nginx
    # timeout and bad gateway errors
    error_page 502 503 @maintenance;
    # php errors
    error_page 500 @maintenance;
    # not implemented
    error_page 501 @maintenance;
    # Gateway timeout
    error_page 504 @maintenance;
    # version not supported
    error_page 505 @maintenance;


    # this is needed for redirect to work
    merge_slashes off;
    # this ensures our urls are cleaned of multiple slashes
    rewrite (.*)//(.*) $1/$2 permanent;

    # Our maintenance block
    location @maintenance {
            add_header Retry-After 600 always;
            # We need to include these again for the retry header to not remove them from above
            add_header Strict-Transport-Security "max-age=63072000; includeSubDomains" always;
            add_header X-Content-Type-Options nosniff always;
            add_header X-XSS-Protection "1; mode=block" always;
            add_header X-Frame-Options DENY always;
            # Map everything to /maintenance.html
            rewrite ^(.*)$ /maintenance.html break;
            # This is internal redirect
            internal;
    }

    # disable direct access to maintenance.html
    location = /maintenance.html {
        return 418;
    }

    # for letsencrypt
    location /.well-known/acme-challenge/ {
        rewrite ^/.well-known/acme-challenge/(.*) /$1 break;
        root /acme;
    }

    # Hide .htpasswd and .htaccess files as well
    # as any file starting with .ht
    location ~ /\.ht {
        return 404;
    }

    # avoid processing of calls to non-existing static files by yii
    # and if they exist serve them directly with some caching
    location ~* \.(?:ico|css|woff2|exe|fla|gif|jpe?g|jpg|js|mov|pdf|png|rar|svg|swf|tgz|txt|xml|zip)$ {
        try_files $uri =404;
        access_log off;
        log_not_found off;
        expires 1M;
        add_header Cache-Control "public";
        add_header Strict-Transport-Security "max-age=63072000; includeSubDomains" always;
        add_header X-Content-Type-Options nosniff always;
        add_header X-XSS-Protection "1; mode=block" always;
        add_header X-Frame-Options DENY always;
    }

    set $yii_bootstrap "/index.php";

    # Rate limit /api to 10 requests/sec
    # You need to add the following to your http { } nginx block
    # limit_req_zone $binary_remote_addr zone=Api:1m rate=10r/s;
    location /api/ {
        limit_req_status 429;
        limit_conn_status 429;
        limit_req zone=Api;
        index  index.php;
        try_files $uri $uri/ /index.php$is_args$args;
    }

    location / {
        index  index.html index.php;
        try_files $uri $uri/ /index.php$is_args$args;
    }

    # return 404 for direct access to php files
    location ~ \.php$ {
      return 404;
    }

    location = /index.php {
        try_files $uri =404;
        fastcgi_split_path_info ^(.+\.php)(.*)$;

        # let yii catch the calls to non-existing PHP files
        set $fsn /$yii_bootstrap;
        set $yiiargs r=$request_uri;
        if (-f $document_root$fastcgi_script_name){
            set $fsn $fastcgi_script_name;
            set $yiiargs $query_string;
        }

        fastcgi_pass {{item.fpm}};
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME  $document_root$fsn;

        # NGiNX allowed duplicate HOST headers to be set, the following is
        # a workaround for these cases.
        fastcgi_param HTTP_HOST "{{item.domain}}";

        #PATH_INFO and PATH_TRANSLATED can be omitted, but RFC 3875 specifies them for CGI
        fastcgi_param PATH_INFO        $fastcgi_path_info;
        fastcgi_param PATH_TRANSLATED  $document_root$fsn;
        fastcgi_param QUERY_STRING $yiiargs;
        fastcgi_intercept_errors on;
    }

    location @errorphp {
        internal;
        fastcgi_split_path_info ^(.+\.php)(.*)$;

        # let yii catch the calls to non-existing PHP files
        set $fsn /$yii_bootstrap;
        set $yiiargs r=$request_uri;
        if (-f $document_root$fastcgi_script_name){
            set $fsn $fastcgi_script_name;
            set $yiiargs $query_string;
        }

        fastcgi_pass {{item.fpm}};
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME  $document_root$fsn;

        # NGiNX allowed duplicate HOST headers to be set, the following is
        # a workaround for these cases.
        fastcgi_param HTTP_HOST "{{item.domain}}";

        #PATH_INFO and PATH_TRANSLATED can be omitted, but RFC 3875 specifies them for CGI
        fastcgi_param PATH_INFO        $fastcgi_path_info;
        fastcgi_param PATH_TRANSLATED  $document_root$fsn;
        fastcgi_param QUERY_STRING $yiiargs;
        fastcgi_intercept_errors off;
    }
}
```

This is still under a lot of development and fine tuning but it will save you the hair pulling 😀

## TODO
* Set proper pages for 429, 500, 501, 502
* https://en.wikipedia.org/wiki/List_of_HTTP_status_codes