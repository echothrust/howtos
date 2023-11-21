---
tags:
  - OpenBSD
  - WIP
---
# OpenBSD and findingsd
This is something that i've been wanting to write for so long, but every time i even tried to start putting down my thoughts, i changed my mind. 
Hopefully this time around will be different 🤞.

## echoCTF and `findingsd`
echoCTF.RED is a framework that allows one to manage the entire life-cycle of cybersecurity exercises or competitions (ctf). Part of its features,
is the ability to track the services that are running on a host and keep monitor who's accessing them as well as when. 

These are the echoCTF _findings_!

In oversimplified terms, findings are services that are listening on a target host, that we want to be able to know who is accessing them and when. 
At the same findings let the user know, that a particular service might warrant a further investigation.

So lets assume we have a target that is running an nginx web server (80/tcp), openssh (22/tcp) and snmpd (161/udp) among other things. So these 3 services 
will be our findings for this target. We want to know when a user is accessing these services for the first time so that we can give them hints (eg when they 
access the tcp port 80 for nginx we let them know that there is also udp services running).

## the requirements
So we want to be able to monitor specific services but we do have some things to consider.
* We need to be able to add and remove services and targets with ease
* We want to be able to limit the source and destination matches if possible. This means that we may want to monitor a given player but not another.
* We want to be able to perform the monitoring in a passive way, without having to interject into the players connection. So no proxy solutions.
* We want to be able to access a small set of packet metadata but not necessarily the entire packet
* We want to be able to monitor only specific types of packets and not process everything (eg only syn packets)
* We want to impose the smallest possible overhead, so it is preferable to avoid using pipes which involve multiple programs and scripts as the overhead may get too much.
* We want to be able to link these packets into our database which is MySQL
* We dont want to "kill" our database server with connections
* Access to the target services must not depend on our ability to log the packet. Even if our solution crashes, we want to allow users to access their intended destination (passive monitoring).

This means that the obvious solution of using `tcpdump` is out of the question but not entirely. Managing filtering rules is a pain even for the simplest cases not to mention having
to filter based on 100s of services and destination hosts. However `libpcap` might be an option.

## the `match`, the `log` and the `label`
In order to achieve this _monitoring_ of services we utilize two features from OpenBSD's PF. The ability to `match` a packet based on a set of given rules 
and the ability to `log` a packet. 

### `match`-ing
For those that use OpenBSD the most common use case is something like the following entry in the `/etc/pf.conf` file, which scrubs (performs cleanups) on incoming packets.
```
match in all scrub (no-df)
```
or when natting can be seen used in the following form
```
match out on tl0 from 192.168.1.0/24 to any nat-to 198.51.100.1
```
The `match` rules follow the same syntax as `pass`, however as can be seen from the examples, they usually need to perform something to the matched packets, even 
though its not a `pass` or `block`. 

### `log`-ing
So we've covered part of our requirements with regards to matching specific packets but we still dont do anything with them. How can we receive and process these
packets that matched our criteria?

OpenBSD already has a way to log packets that pass or block through the use of `log` pf rule parameter. By default, this logs packets to a virtual interface, `pflog0`.
However, we cant use the default interface if we also want to log firewall actions (eg blocked packets), since it will get extremely noisy for us when we want to 
analyze eg attacks. Well, it turns out OpenBSD has another neat feature, we can have as many virtual interfaces as we like and we can instruct PF to log only a specific 
set of packets to it.

So our previous match rules can be modified to log to a specific interface, like so:
```
match in (log pflog1) all scrub (no-df)
match out (log pflog1) on tl0 from 192.168.1.0/24 to any nat-to 198.51.100.1
```
This will log all packets that match the criteria to the pseudo interface `pflog1`. 

### `label`-ing
So now we've covered matching and loging but all the actions we saw already perform an unwanted action to our packets (`scrub` & `nat-to`). The last piece 
of the puzzle is another awesome feature of OpenBSD's PF, which is the labeling of packet. 

Labels are an internal PF functionality that allows you to add a symbolic a label to the rule, which can be used to identify the rule later on, 
and this has almost 0 overhead and no effect on the packet it self. Whats more we can have some macros for packet details which can later allow us even better control 
to the tracking of our rules. 

The following macros can be used in `label`:
* `$dstaddr`: The destination IP address
* `$dstport`: The destination port specification
* `$if`: The interface
* `$nr`: The rule number
* `$proto`: The protocol name
* `$srcaddr`: The source IP address
* `$srcport`: The source port specification

Again, using the previous rules as an example, we will add the destination address and destination port as labels
```
match in log (to pflog1) all scrub (no-df) label "$dstaddr:$dstport"
match out log (to pflog1) on tl0 from 192.168.1.0/24 to any nat-to 198.51.100.1 label "$dstaddr:$dstport"
```

### putting it all together
So now we can create rules that perform no action on the packets, other than logging them to an interface of our choice.
```
match log (to pflog1) inet proto tcp from 1.2.3.4 to 5.6.7.8 port 1337 label "$dstaddr:$dstport"
```
This could be further optimized by replacing source and destination IP's with tables, which will require only a single rule for any number of source/destination combinations.
```
match log (to pflog1) inet proto tcp from <players> to <targets> port 1337 label "$dstaddr:$dstport"
```
