---
---

# OpenBSD PF Cheatsheet

## General PFCTL Commands
* `pfctl -d` disable packet-filtering
* `pfctl -e` enable packet-filtering
* `pfctl -q` run quiet
* `pfctl -vv` run even more verbose

## Loading PF Rules
* `pfctl -f /etc/pf.conf` load /etc/pf.conf
* `pfctl -n -f /etc/pf.conf` parse /etc/pf.conf, but dont load it
* `pfctl -R -f /etc/pf.conf` load only the FILTER rules
* `pfctl -N -f /etc/pf.conf` load only the NAT rules
* `pfctl -O -f /etc/pf.conf` load only the OPTION rules

## Clearing PF Rules & Counters
* `pfctl -F all` flush ALL (rules, counters and states)
* `pfctl -F states` flush states (kills active connections)
* `pfctl -F rules` flush only the rules (connections stay open)
* `pfctl -F queue` flush only queue’s
* `pfctl -F nat` flush only NAT
* `pfctl -F info` flush all stats that are not part of any rule.
* `pfctl -z` clear all per-rule counters

note: flushing rules do not touch any existing stateful connections

## Output PF Information
* `pfctl -s rules` show filter information
* `pfctl -v -s rules` show filter information for what FILTER rules hit..
* `pfctl -vvsr` show filter information as above and prepend rule numbers
* `pfctl -v -s nat` show NAT information, for which NAT rules hit..
* `pfctl -s nat -i xl1` show NAT information for interface xl1
* `pfctl -s queue` show QUEUE information
* `pfctl -s label` show LABEL information
* `pfctl -s state` show contents of the STATE table
* `pfctl -s info` show statistics for state tables and packet normalization
* `pfctl -s all` show everything

## Maintaining PF Tables
* `pfctl -t vhosts -T show` show table vhosts
* `pfctl -vvsTables` view global information about all tables
* `pfctl -t vhosts -T add 192.168.1.50` add entry to table vhosts
* `pfctl -t vhosts -T add 192.168.1.0/16` add a network to table vhosts
* `pfctl -t vhosts -T delete 192.168.1.0/16` delete network from table vhosts
* `pfctl -t vhosts -T flush` remove all entries from table vhosts
* `pfctl -t vhosts -T kill` delete table vhosts entirely
* `pfctl -t vhosts -T replace -f /etc/vhosts` reload table vhosts on the fly
* `pfctl -t vhosts -T test 192.168.1.40` find ip address 192.168.1.40 in table vhosts
* `pfctl -T load -f /etc/pf.conf` load a new table definition only
* `pfctl -t vhosts -T show -v` output stats for each ip address in table vhosts
* `pfctl -t vhosts -T zero` reset all counters for table vhosts
