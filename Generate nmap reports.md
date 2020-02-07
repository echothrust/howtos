# Generate nmap Reports
The following document describes a simple procedure to perform and compare network scans to detect anomalies.

Nmap has the ability to save output file in XML format, by using the '-oX' parameter.

* Create a list of networks that will be scanned (eg: `/etc/nmap/networks_weekly`)
```
172.16.0.0/24
172.16.10.0/24
172.16.22.0/24
```

* Scan the networks for hosts that are alive on these ranges
```sh
nmap -sP -iL /etc/nmap/networks_weekly -oG /tmp/grepable_hosts
```

* Separate `Up` and `Down` hosts
```sh
egrep -v "Down|#" /tmp/grepable_hosts > /tmp/alive-$(date +%d%m%y)
egrep -v "Up|\(\)|#" /tmp/grepable_hosts > /tmp/dead-$(date +%d%m%y)
```

* Create list of `Up` only hosts to be used by nmap's `-iL`
```sh
awk '{ print $2 }' /tmp/alive-$(date +%d%m%y) > /tmp/ips_alive
```

* Use the list to perform your scan on the hosts with your desired options and generate an XML report
```sh
nmap -iL ips_alive -sS -oX /root/nmap-reports/scan_alive-$(date +%d%m%y).xml
rm /tmp/dead-$(date +%d%m%y)
rm /tmp/alive-$(date +%d%m%y)
rm /tmp/grepable_hosts
```

Now you can use your favorite tools to parse and compare XML files.
