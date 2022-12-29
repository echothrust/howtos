# OpenBSD php-fpm chroot

**Tags:** `OpenBSD`, `OPENBSD_6_0`, `php-fpm`, `chroot`

The following document is a guide on how to prepare a chroot environment for php
in order to be able to run most of the available applications (drupal etc)
without problems.

The guide makes the following assumsions
* We will use the default `/var/www` location for the chroot location.
* The setup should work on `nginx(8)` and default OpenBSD `httpd(8)`
* We will install more php packages than needed in order to provide
  configuration requirements for each php module, you can cherry pick the
  modules you use and configure accordingly.


## Package Installations

The following packages can be tweaked to your liking.

We start by installing the packages
```
pkg_add -vvi femail php-curl php-fpm php-gd php-intl php-ldap php-mcrypt
pkg_add -vvi php-mysqli php-pdo_mysql php-zip pecl-APC pecl-uploadprogress
```


## Packages configuration

Configure the php-fpm service to listen on a socket and chroot under our
standard `www` user's home directory under `/var/www`. The listening socket will be
located under `/var/www/var/run/php-fpm.sock`.
```sh
rcctl enable php_fpm
cat <<__EOF__> /etc/php-fpm.conf
[global]
; no global options at the moment

; pool www
[www]
catch_workers_output = yes
user = www
group = www
listen = /var/www/var/run/php-fpm.sock
;listen = 127.0.0.1:9000
listen.owner = www
listen.group = www
listen.mode = 0660
pm = dynamic
pm.max_children = 15
pm.start_servers = 2
pm.min_spare_servers = 1
pm.max_spare_servers = 3
chroot = /var/www
__EOF__
```


## Preparing the chroot

Create required folders and copy basic system files such as `/etc/hosts`,
`/etc/resolv.conf`, `/etc/services`, `/etc/protocols` and `/etc/pwd.db`.

```sh
install -d -o root /var/www/{dev,etc/ssl,var/run,usr/sbin,bin}
install -d -o root -m 1777 /var/www/tmp
cp /etc/{hosts,services,protocols,resolv.conf,pwd.db} /var/www/etc/
cp /bin/sh /var/www/bin/sh
```

Prepare standard devices for the chroot and make sure we have a
`/var/www/dev/log` socket by `syslogd`.

```sh
cd /var/www/dev
/dev/MAKEDEV std
rcctl set syslogd flags "-4 -a /var/www/dev/log"
rcctl restart syslogd
```


## Sending mail

The package `femail` acts as a replacement for the sendmail binary within the
chroot. However, we need to configure php to use it by appending the following
overwride to our php-fpm.conf.
```sh
echo 'php_admin_value[sendmail_path] = /bin/femail -t'>>/etc/php-fpm.conf
rcctl restart php56_fpm
```

**NOTE**: `femail` is lacking the **`-i`** option which states
```
 -i          Do not strip a leading dot from lines in incoming messages,
             and do not treat a dot on a line by itself as the end of an
             incoming message.  This should be set if you are reading data
             from a file.
```

You can use the following command to test your chroot is working and can send emails
```
chroot -g www -u www /var/www/ /bin/femail -v -t -i me@myaddress.com
```

## OpenSSL

OpenSSL from the chroot. In order for OpenSSL to verify certificates we need to
make sure the trusted certificate authorities file of our system is also
available on the chroot environment.
```sh
cp /etc/ssl/cert.pem /var/www/etc/ssl
rcctl restart php56_fpm
```

If you want to generate keys and certificates from within your PHP scripts you
will also need the following files to be made available under the chroot
environment.
```
mkdir /var/www/etc/ssl/{private,certs,lib}
cp /etc/ssl/{openssl.cnf,x509v3.cnf} /var/www/etc/ssl
```

**NOTE**: The restart is needed because the certificate trust store is searched
only once at the php_fpm service start.


## OpenLDAP

If any particular setup is needed in order to access ldap server from the chroot
we will have to copy the following files
```sh
cp /etc/openldap/ldap.conf /var/www/etc/openldap/ldap.conf
rcctl restart php56_fpm
```

**NOTE**: A restart is needed because the configuration is looked upon once at
startup of the `php_fpm` service.


## Local time, Date, Time and Timestamps

In order to enable correct localtime and datetime conversions to other
timezones, you'll have to make sure the `zoneinfo` is available under the
chroot.

Copy your existing `/etc/localtime` and the timezone database.
```
cp /etc/localtime /var/www/etc
mkdir -p /var/www/usr/share
cp -pr /usr/share/zoneinfo /var/www/usr/share
```

## CURL configuration
Depending on your installation of CURL, there will be some libraries needed in
order to be able to fully utilize its operations (eg HTTPS).

By performing and copying the needed files by the PHP module we have a preaty
safe list of candidate libraries for our chroot.

By running
```sh
ldd /usr/local/lib/php-5.6/modules/curl.so
```

we get an output like the following,
```
/usr/local/lib/php-5.6/modules/curl.so:
        Start            End              Type Open Ref GrpRef Name
        0000131b19cbc000 0000131b1a0d1000 dlib 2    0   0      /usr/local/lib/php-5.6/modules/curl.so
        0000131baa03f000 0000131baa4a8000 rlib 0    1   0      /usr/local/lib/libcurl.so.25.5
        0000131b94412000 0000131b94836000 rlib 0    2   0      /usr/local/lib/libnghttp2.so.0.6
        0000131adc88a000 0000131adccbd000 rlib 0    2   0      /usr/local/lib/libidn.so.17.2
        0000131ae64f6000 0000131ae6950000 rlib 0    2   0      /usr/lib/libssl.so.39.0
        0000131b3cfcd000 0000131b3d59c000 rlib 0    3   0      /usr/lib/libcrypto.so.38.0
        0000131af2047000 0000131af245c000 rlib 0    2   0      /usr/lib/libz.so.5.0
        0000131b2797b000 0000131b27d89000 rlib 0    1   0      /usr/lib/libpthread.so.22.0
        0000131b781c0000 0000131b786be000 rlib 0    3   0      /usr/local/lib/libiconv.so.6.0
        0000131b434b3000 0000131b438bd000 rlib 0    2   0      /usr/local/lib/libintl.so.6.0
```

We proceed by generating the folders and copying the libraries mentioned

```sh
mkdir -p /var/www/usr/local/lib /var/www/usr/lib

cp /usr/local/lib/php-5.6/modules/curl.so /var/www/usr/local/lib
cp /usr/local/lib/libcurl.so.25.5 /var/www/usr/local/lib
cp /usr/local/lib/libnghttp2.so.0.6 /var/www/usr/local/lib
cp /usr/local/lib/libidn.so.17.2 /var/www/usr/local/lib
cp /usr/local/lib/libiconv.so.6.0 /var/www/usr/local/lib
cp /usr/local/lib/libintl.so.6.0 /var/www/usr/local/lib

cp /usr/lib/libssl.so.39.0 /var/www/usr/lib
cp /usr/lib/libcrypto.so.38.0 /var/www/usr/lib
cp /usr/lib/libz.so.5.0 /var/www/usr/lib
cp /usr/lib/libpthread.so.22.0 /var/www/usr/lib
```

**NOTE** These commands can be generated (just printed on terminal, to copy-paste the output):
```
for i in $(ldd /usr/local/lib/php-5.6/modules/curl.so |awk '{print $7}'|grep ^/usr/lib); do echo cp $i /var/www/usr/lib; done
for i in $(ldd /usr/local/lib/php-5.6/modules/curl.so |awk '{print $7}'|grep ^/usr/local/lib); do echo cp $i /var/www/usr/local/lib; done
```

## Timeouts for executed scripts (nginx config)

XXX FIXME XXX

## Troubleshooting

### Test FPM from the CLI
* Debug applications from CLI needs the package `fcgi`. So first install the
package by executing `pkg_add -vvi fcgi` and then test your chroot by

```
export SCRIPT_NAME=/htdocs/index.php SCRIPT_FILENAME=/htdocs/index.php REQUEST_METHOD=GET
cgi-fcgi -bind -connect 127.0.0.1:9000 # if you listen tcp
cgi-fcgi -bind -connect /var/www/var/run/php-fpm.sock # if you listen socket
```

### Debug library issues
Debugging chroot library issues (`ldd` ). If you're having issues with a
library or application from within the chroot it is always a good idea to ldd
the binary and see what the files are needed to be copied.

For instanse in order to make `/usr/local/bin/zip` binary available to be
executed under the chroot we would do
```
mkdir -p /var/www/usr/local/bin
cp /usr/local/bin/zip /var/www/usr/local/bin
```

Check the libraries that it depends by executing `ldd /usr/local/bin/zip`.
```
/usr/local/bin/zip:
        Start            End              Type Open Ref GrpRef Name
        0000037403500000 0000037403984000 exe  1    0   0      /usr/local/bin/zip
        00000376a8f03000 00000376a93ec000 rlib 0    1   0      /usr/lib/libc.so.77.0
        0000037628c00000 0000037628c00000 rtld 0    1   0      /usr/libexec/ld.so
```

Copy the libraries into the chroot and you're done
```
mkdir -p /var/www/usr/lib /var/www/usr/libexec
cp /usr/lib/libc.so.77.0 /var/www/usr/lib
cp /usr/libexec/ld.so /var/www/usr/libexec
```

**NOTE**: Occasionaly you might also require to `ldd` the libraries for extra
dependencies.

### Debugging PHP scripts
If you want to see what kind of files are needed in order for a script to be
executed under the chroot, it is good idea to pass the php execution through
the `ktrace` & `kdump` utilities.

A sample invocation looks like the following
```
ktrace -f php.trace php-5.6 /var/www/htdocs/check_dns.php
kdump -f php.trace |grep -i nami
```

## Final words

Although the guide is based on OpenBSD and the OpenBSD provided packages, most
of the commands can be applied to any unix like system for similar purposes.

This guide is constantly updated as we come across new modules and issues with
our installations :).

## References

* Using fcgi from command line https://rtcamp.com/tutorials/php/directly-connect-php-fpm/


## Example code
### Test curl
```php
<?php
/*
 * Test php fpm for proper curl configuration
 */

 $options = array(
        CURLOPT_RETURNTRANSFER => 1,     // return web page
        CURLOPT_HEADER         => 1,    // don't return headers
        CURLOPT_FOLLOWLOCATION => true,     // follow redirects
        CURLOPT_ENCODING       => "",       // handle all encodings
        CURLOPT_USERAGENT      => "Mozilla/5.0 (Windows; U; Windows NT 5.1; en-US; rv:x.x.x) Gecko/20041107 Firefox/x.x", // who am i
        CURLOPT_AUTOREFERER    => true,     // set referer on redirect
        CURLOPT_CONNECTTIMEOUT => 120,      // timeout on connect
        CURLOPT_TIMEOUT        => 120,      // timeout on response
        CURLOPT_MAXREDIRS      => 10,       // stop after 10 redirects
        CURLOPT_SSL_VERIFYHOST => 2,
        CURLOPT_SSL_VERIFYPEER => true,
        CURLOPT_DNS_USE_GLOBAL_CACHE => false,
        CURLOPT_DNS_CACHE_TIMEOUT => 2,
        CURLOPT_HTTPGET => 1
    );

$url="https://graph.facebook.com";
$_h = curl_init($url);
foreach($options as $key =>$val)
	curl_setopt($_h,$key,$val);
curl_setopt($_h, CURLOPT_URL, $url);


var_dump(curl_exec($_h));
var_dump(curl_getinfo($_h));
var_dump(curl_error($_h));
```

### Test DNS
```php
<?php
/*
 * Check that all appropriate calls for name resolution work under the chroot.
*/

$domain="DOMAINTOLOOK";
$inaddr="IPADDR";
// If we run from the CLI we're fine
if(is_array($argv) && count($argv)>1)
{
	$domain=$argv[1];
	if (count($argv)>2) $inaddr=$argv[2];
}
// Set proper encoding so that the format is clean
if(isset($_GET) && is_array($_GET))
	header("Content-Type: text/plain");

// if we run from within fpm
if(isset($_GET['domain']) && !empty($_GET['domain']))
	$domain=$_GET['domain'];

if(isset($_GET['inaddr']) && !empty($_GET['inaddr']))
	$inaddr=$_GET['inaddr'];


check_call("gethostbyaddr",$inaddr);
check_call("gethostbyname","$domain");
check_call("checkdnsrr",$domain);
check_call("checkdnsrr",$domain,"A");
check_call("checkdnsrr",$domain,"NS");
check_call("checkdnsrr",$domain,"TXT");
check_call("checkdnsrr",$domain,"MX");
check_call("checkdnsrr",$domain,"ANY");
check_call("dns_get_record",$domain);

/**
 * Check the return of a call and log output for debuging
 * @param string $func the function to call
 * @param mixed $param1 the first parameter of the call
 * @param mixed $param2 the second parameter of the call if any
 */
function check_call($func,$param1=null,$param2=null)
{
	echo "checking $func with [param1=$param1] and [param2=$param2] => ";
	if($param2==null)
		$ret=call_user_func($func,$param1);
	else
		$ret=call_user_func($func,$param1,$param2);

	if($ret!==false) {
		echo "true ", is_bool($ret) ? "\n" : var_dump($ret);
	}
	else
		echo "false\n";

}
```
