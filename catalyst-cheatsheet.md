# Catalyst Cheatsheet
* `show mac address-table` or `show mac-address-table`

* `show interface vlan vlan-id [ brief | private-vlan mapping ]`
* `show vlan id vlan-id` example: `show vlan id 120`

* show interface status err-disabled (or `show interfaces status err-disabled`)
* show port-security
* clear errdisable intrfacde gigabitethernet 0/2 vlan
* clear errdisable intrfacde gigabitethernet 0/2 vlan 103
* set port enable
* errdisable recovery cause
* sh int g1/0/1
```
conf t
int g1/0/1
shut
no shut
```

show interface switchport (Displays information about the ports, including those in private VLANs)
show vlan (Displays summary information for all VLANs)
show vlan private-vlan (Displays summary information for all private VLANs)
