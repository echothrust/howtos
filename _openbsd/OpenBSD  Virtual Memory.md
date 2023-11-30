---
---

# [WIP] OpenBSD Virtual Memory Statistics with vmstat

## Introduction
As a subscriber of the openbsd-misc mailing list, often times i come across a mail (usually from someone far more knowledgeable) that ends up creating more questions than answers in my head. This case was no different a mail from Claudio Jeker on the thread with subject `OpenBSD SMP - BGPd - send_rtmsg: action 1, prefix A.B.C.D/24: No buffer space available - panic: malloc: out of space in kmem_map`

The mail that got me into all this was the following, which was in response to some attachments from vmstat output under different condition[^1]

```
So the problem is that the malloc space is filled by
a) 26540K of devbuf -- because of the multiqueue support in ixl
b) 63493K of ACPI -- what the heck ACPI?!?
and then there is not enough space for rtable. A full table requires
in your example 50816K of rtable malloc space.

Now on amd64 all of this needs to fit into 128MB which is impossible.

You can use config(8) and bsd.re-config(5) to adjust the nkmempg variable
to something like 131072 (which is 4 times the default size).
This can be verified with `sysctl vm.nkmempages`

Now ixl(4) and ACPI should not be such pigs but in the end 128MB of kernel
malloc space is just stupidly small on a system with 128GB of memory.

The alternative is to set "option NKMEMPAGES=131072" in your GENERIC
config file (or option NKMEMPAGES_MAX=131072). See also options(4).

Long term is the fix this proper. All of this was built when computers had
100MB of memory not 100GB.
```

So I went on a quest find out the following:
* where are these devbuf, ACPI, rtable spaces are
* where is this 128MB limit
* how is this 128MB limit estimated
* how can we see if we're hitting this limit in any of our systems
* how can we modify it to something else if needed

[^1]: [OpenBSD misc thread](https://marc.info/?l=openbsd-misc&m=170110432925974&w=2)

## `nkmempg`: kernel malloc area in PAGE_SIZE-sized logical pages
Display the current nkmempg number of PAGE_SIZE-sized logical pages for your system `sysctl vm.nkmempages` (check options(4))

The default PAGE_SIZE for amd64 is 4096 and is calculated under `amd64/param.h` as such
```
#define PAGE_SHIFT 12
#define PAGE_SIZE	 (1 << PAGE_SHIFT)
```

So in order to estimate how much memory will be allocated for the kernel pages we can do the following
```sh
PAGE_SIZE=$((1<<12)) # calculate the pagesize
PAGE_SIZE=$(sysctl -n hw.pagesize) # alternatively get the current pagesize as defined by the running hardware
NKMEMPAGES=$(sysctl -n vm.nkmempages) # get the current nkmempages
NKMEMPAGES_BYTES=$(($NKMEMPAGES*$PAGE_SIZE)) # multiply by PAGE_SIZE to total bytes
NKMEMPAGES_MB=$(($NKMEMPAGES_BYTES/1024/1024)) # divide by 1024 twice to get MB
```

This gives us the (default) size of 128MB.


## Checking virtual memory statistics with `vmstat`
The manual page for vmstat(8) states the following
> vmstat reports certain kernel statistics kept about process, virtual memory, disk, trap, and CPU activity. The default behavior is to print a one-line summary of these statistics. The -c and -w flags may be used to continually report summaries

among its options, the one that we will be focusing on is `-m` which the manual states the following
> -m   Report on the usage of kernel dynamic memory listed first by size of allocation and then by type of usage.

When running `vmstat -m` we get presented with 4 different views
* Memory statistics by bucket size
* Memory usage type by bucket size
* Memory statistics by type
* Memory resource pool statistics


The size for the `rtable` can be found at the original message and is derived by the output produced by vmstat[^2] with support for single processor, under the section `Memory statistics by type`. The sizes for `ACPI` and `devbuf` can be found on the output produced by vmstat[^3] with support for symmetric multi processor systems under the same section.

The columns of interest for deducing these numbers are `Type`, `InUse`, `MemUse` and `HighUse` which correspond to the following
* **`Type`**: Type/name of allocation category
* **`InUse`**: Number of pages[^4] currently in use
* **`MemUse`**: Memory used by those pages
* **`HighUse`**: Highest usage recorded for this `Type`

[^2]: [vmstat-m_SP_with_bgpd](https://marc.info/?l=openbsd-misc&m=170110432925974&q=p15)
[^3]: [vmstat-m_SMP_with_bgpd_011](https://marc.info/?l=openbsd-misc&m=170110432925974&q=p3)
[^4]: I am not 100% sure this is what this number represents atm


## `NKMEMPAGES` and `NKMEMPAGES_MAX`
Size of kernel malloc area in `PAGE_SIZE`-sized logical pages. This area is covered by the kernel submap `kmem_map`. The kernel attempts to auto-size this map based on the amount of physical memory in the system. Platform-specific code may place bounds on this computed size, which may be viewed with the `sysctl(8)` variable `vm.nkmempages`.

See `/usr/include/machine/param.h` for the default upper bound. The related option `NKMEMPAGES_MAX` allows the bounds to be overridden in the kernel configuration file in the event the computed value is insufficient resulting in an `out of space in kmem_map` panic.

## NMBCLUSTERS
The following is taken from a comment by Henning Brauer :man_bowing: on undeadly.org[^5]

> Network data is traversing through the system in so-called `mbufs`, memory buffers of a fixed size - 256 bytes if memory serves me right. Lots of them linked together (linked list) represent a packet then.
>
> As most stuff the kernel needs to do with network data only needs access to the header data and this is completely withing the first `mbuf` usually, that is very efficient.
>
> HOWEVER, the payload may be bigger, and it would be very inefficient to split it into 265 bytes chunks. That's why `mbuf` clusters exist, which are bigger - 2048 bytes if memory serves me right, and the mbufs only contain a reference to the mbuf cluster then.
>
> This memory space is only needed as long as the network data is traveling through the system. Once it is send out on some NIC or written to some socket where some deamon is listening, or dropped for whatever reason, the mbufs and mbuf clusters occupied by the packet are freed.
>
> If they are not freed then we have a mbuf leak, thus, the amount of `NMBCLUSTERS` you need is usually _very_ small. I have exactly 3 machines out of 60 or so that need increased `NMBCLUSTERS`...
>
> oh, and the `mbuf(9)` manpage is a very good read on that.

[^5]: [Running and tuning OpenBSD network servers in a production environment](https://www.undeadly.org/cgi?action=article;sid=20030208023845)

## References
* [The Linux kernel: Memory - 9.3 Pages](https://www.win.tue.nl/~aeb/linux/lk/lk-9.html#ss9.3)
* [CSCI.4210 Operating Systems - Virtual Memory](https://www.cs.rpi.edu/academics/courses/fall04/os/c12/)