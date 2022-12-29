# ddb Commands

* `show panic`
* `show uvm`
* `trace`
* `call`

## Inspect address `call db_show_rtentry()`

```
icmp_mtudisc_timeout(fffffd807a50b070,0) at icmp_mtudisc_timeout+0x77
rt_timer_timer(ffffffff8235d668) at rt_timer_timer+0x1cc
softclock_thread(ffff8000fffff260) at softclock_thread+0x13b
end trace frame: 0x0, count: -3
ddb{0}> call db_show_rtentry(fffffd807a50b070, 0, 0)
```
Where the first argument is derived from the first argument of icmp_mtudisc_timeout from the trace.

If there is a fault and the kernel enters DDB then there will be no panic in `show panic`.
