# Netmod ISDN BRI terminal
These instructions assume the most common setup of these devices as they were used in Greece by OTE. This is a primary (head) number and an additional number (MSN).
If you were used Smart Office offering then there will be a 3rd number provided to you for call forwarding and other operations provided by the service.
For the instructions bellow the numbers used are as follows:
* Head: `2101111111`
* MSN: `2101111112`
* Smart Office: `2101111114`
settings for the netmod bringing the circuit for 210-2230050.
This document is useful in the event of a reset or replacement of our netmod.

### Plug analog phone on **AB1 port**

Clean all (MSN) settings

```
**91*#
**92*#
**93*#
```

Add head number to slot-1 and Smart Office number to slot-2

```
**91*2101111111#
**92*2101111114#
```

Note: 2101111114 is some additional number we were handed only for smart office
to work. Call forwarding is essentially applied on that number (as stated by
OTE tech support person).

### Now, plug analog phone to **AB2 port**

Again, clean all MSN settings

```
**91*#
**92*#
**93*#
```

Attach MSN on a specific PSTN port (AB2)

```
**91*2101111112#
```

Note: Once done, make sure the FAX machine is plugged back on AB2
