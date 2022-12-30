---
---

# Catalyst Cheatsheet

* Display information about the ports, including those in private VLANs `show interface switchport`
* Display summary information for all VLANs `show vlan`
* Display summary information for all private VLANs `show vlan private-vlan`
* Display mac addresses `show mac address-table` or `show mac-address-table`
* `show interface vlan vlan-id [ brief | private-vlan mapping ]`
* `show vlan id vlan-id` example: `show vlan id 120`
* `show interface status err-disabled` or `show interfaces status err-disabled`
* show port security `show port-security`
* `clear errdisable intrfacde gigabitethernet 0/2 vlan`
* `clear errdisable intrfacde gigabitethernet 0/2 vlan 103`
* `set port enable`
* `errdisable recovery cause`
* `sh int g1/0/1`
* shut no shut
```
conf t
int g1/0/1
shut
no shut
```
