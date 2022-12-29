# OpenBSD Generic Health checks


## TCP Statistics
There are some counters in netstat -sptcp output. In particular:
```
        532358 SYN packets dropped due to queue or memory full
        770174566 SYN cache entries added
                0 hash collisions
                768118448 completed
                6 aborted (no space to build PCB)
                905577 timed out
                0 dropped due to overflow
                0 dropped due to bucket overflow
                1150155 dropped due to RST
                0 dropped due to ICMP unreachable
        7036108 SYN,ACKs retransmitted
        13117384 duplicate SYNs received for entries already in the cache
        2580 SYNs dropped (no route or no space)
```

## Detect select() collisions
http://www.tedunangst.com/flak/post/select-works-poorly

```
sysctl kern.nselcoll
```
