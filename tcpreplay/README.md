##### Description
Tcpreplay is a suite of free Open Source utilities for editing and replaying previously captured network traffic           
Vendor: https://tcpreplay.appneta.com/

## Heap overflow in get_next_packet()

A `memcpy()` in function `get_next_packet()` causes a buffer overflow due to source `pktdata` & destination `(*prev_packet)->pktdata` overlap in `send_packets.c`. 



###### **Affected version:**
  4.3 branch

  
###### **Command**: 
 sudo tcpreplay -i eno1 -t -K --loop 4 --unique-ip $POC


### **Debugging**

```
  1044                      (*prev_packet)->pktdata = safe_malloc(pktlen);
                // prev_packet=0xbfffef60 -> [...] -> 0x00000000, pktdata=0xbfffef24 -> [...] -> 0x07290c00
->1045                       memcpy((*prev_packet)->pktdata, pktdata, pktlen);   //Buffer overflow
   1046                      memcpy(&((*prev_packet)->pkthdr), pkthdr, sizeof(struct pcap_pkthdr));
   1047                  }
   1048              }
   1049          }

```



```
[#0] 0x8052f3a->Name: get_next_packet(ctx=0xb6403280, pcap=0xb4203280, pkthdr=0xbffff000, idx=0x0, prev_packet=0xbfffefc0)
[#1] 0x804e922->Name: preload_pcap_file(ctx=0xb6403280, idx=0x0)
[#2] 0x805615c->Name: main(argc=0x1, argv=0xbffff724)
```


```
gef> info locals
options = 0xb6200200
pktdata = 0xb3514800 ""
pktlen = 0x80003e
__PRETTY_FUNCTION__ = "get_next_packet"
__FUNCTION__ = "get_next_packet"

gef> ptype (*prev_packet)->pktdata
type = unsigned char *
gef> p pktdata
$30 = (u_char *) 0xb3514800 ""


gef> p (*prev_packet)->pktdata
$27 = (u_char *) 0xb2afe800 ""
gef> x (*prev_packet)->pktdata
0xb2afe800:     0

gef> ptype pktlen
type = unsigned int
gef> p/d pktlen
$25 = 8388670
```



```
gef> i r
eax            0xb4800320          0xb4800320
ecx            0x3                 0x3
edx            0x0                 0x0
ebx            0xb4800310          0xb4800310
esp            0xbfffef00          0xbfffef00
ebp            0xbfffef48          0xbfffef48
esi            0x0                 0x0
edi            0xb2afe800          0xb2afe800
eip            0x8052f3a           0x8052f3a <get_next_packet+1725>
eflags         0x246               [ PF ZF IF ]
cs             0x73                0x73
ss             0x7b                0x7b
ds             0x7b                0x7b
es             0x7b                0x7b
fs             0x0                 0x0
gs             0x33                0x33
```




### **ASAN output**

```
=================================================================
==22604==ERROR: AddressSanitizer: heap-buffer-overflow on address 0xb35247ff at pc 0xb7adba75 bp 0xbfffeec8 sp 0xbfffea9c
READ of size 8388670 at 0xb35247ff thread T0
    #0 0xb7adba74 in __asan_memcpy (/usr/lib/i386-linux-gnu/libasan.so.2+0x8aa74)
    #1 0xb7adbc2f in memcpy (/usr/lib/i386-linux-gnu/libasan.so.2+0x8ac2f)
    #2 0x8052fb6 in get_next_packet /home/loginsoft/ACE/tcpreplay/src/send_packets.c:1045
    #3 0x804e921 in preload_pcap_file /home/loginsoft/ACE/tcpreplay/src/send_packets.c:445
    #4 0x805615b in main /home/loginsoft/ACE/tcpreplay/src/tcpreplay.c:126
    #5 0xb784c636 in __libc_start_main (/lib/i386-linux-gnu/libc.so.6+0x18636)
    #6 0x804a7a0  (/usr/local/bin/tcpreplay+0x804a7a0)

0xb35247ff is located 0 bytes to the right of 65535-byte region [0xb3514800,0xb35247ff)
allocated by thread T0 here:
    #0 0xb7ae7dee in malloc (/usr/lib/i386-linux-gnu/libasan.so.2+0x96dee)
    #1 0xb7a28e7c  (/usr/lib/i386-linux-gnu/libpcap.so.0.8+0x1ce7c)

SUMMARY: AddressSanitizer: heap-buffer-overflow ??:0 __asan_memcpy
Shadow bytes around the buggy address:
  0x366a48a0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x366a48b0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x366a48c0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x366a48d0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x366a48e0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
=>0x366a48f0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00[07]
  0x366a4900: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x366a4910: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x366a4920: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x366a4930: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x366a4940: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
Shadow byte legend (one shadow byte represents 8 application bytes):
  Addressable:           00
  Partially addressable: 01 02 03 04 05 06 07
  Heap left redzone:       fa
  Heap right redzone:      fb
  Freed heap region:       fd
  Stack left redzone:      f1
  Stack mid redzone:       f2
  Stack right redzone:     f3
  Stack partial redzone:   f4
  Stack after return:      f5
  Stack use after scope:   f8
  Global redzone:          f9
  Global init order:       f6
  Poisoned by user:        f7
  Container overflow:      fc
  Array cookie:            ac
  Intra object redzone:    bb
  ASan internal:           fe
==22604==ABORTING
[Inferior 1 (process 22604) exited with code 01]
```



### **Valgrind report**

```
==13353== Source and destination overlap in memcpy(0x467d028, 0x4648c50, 8388670)
==13353==    at 0x4030D39: memcpy (in /usr/lib/valgrind/vgpreload_memcheck-x86-linux.so)
==13353==    by 0x804D24B: ??? (in /usr/local/bin/tcpreplay)
==13353==    by 0x804BCC0: ??? (in /usr/local/bin/tcpreplay)
==13353==    by 0x804E3FA: ??? (in /usr/local/bin/tcpreplay)
==13353==    by 0x40DD636: (below main) (libc-start.c:291)
==13353==
==13353== Invalid read of size 4
==13353==    at 0x4030DD0: memcpy (in /usr/lib/valgrind/vgpreload_memcheck-x86-linux.so)
==13353==    by 0x804D24B: ??? (in /usr/local/bin/tcpreplay)
==13353==    by 0x804BCC0: ??? (in /usr/local/bin/tcpreplay)
==13353==    by 0x804E3FA: ??? (in /usr/local/bin/tcpreplay)
==13353==    by 0x40DD636: (below main) (libc-start.c:291)
==13353==  Address 0x467d024 is 4 bytes before a block of size 8,388,670 alloc'd
==13353==    at 0x402C17C: malloc (in /usr/lib/valgrind/vgpreload_memcheck-x86-linux.so)
==13353==    by 0x8053B5B: ??? (in /usr/local/bin/tcpreplay)
==13353==    by 0x804D22E: ??? (in /usr/local/bin/tcpreplay)
==13353==    by 0x804BCC0: ??? (in /usr/local/bin/tcpreplay)
==13353==    by 0x804E3FA: ??? (in /usr/local/bin/tcpreplay)
==13353==    by 0x40DD636: (below main) (libc-start.c:291)
==13353==
==13353== Invalid read of size 4
==13353==    at 0x4030DDE: memcpy (in /usr/lib/valgrind/vgpreload_memcheck-x86-linux.so)
==13353==    by 0x804D24B: ??? (in /usr/local/bin/tcpreplay)
==13353==    by 0x804BCC0: ??? (in /usr/local/bin/tcpreplay)
==13353==    by 0x804E3FA: ??? (in /usr/local/bin/tcpreplay)
==13353==    by 0x40DD636: (below main) (libc-start.c:291)
==13353==  Address 0x467d020 is 8 bytes before a block of size 8,388,670 alloc'd
==13353==    at 0x402C17C: malloc (in /usr/lib/valgrind/vgpreload_memcheck-x86-linux.so)
==13353==    by 0x8053B5B: ??? (in /usr/local/bin/tcpreplay)
==13353==    by 0x804D22E: ??? (in /usr/local/bin/tcpreplay)
==13353==    by 0x804BCC0: ??? (in /usr/local/bin/tcpreplay)
==13353==    by 0x804E3FA: ??? (in /usr/local/bin/tcpreplay)
==13353==    by 0x40DD636: (below main) (libc-start.c:291)
==13353==
```

 [Reproducer](https://github.com/SegfaultMasters/covering360/blob/master/tcpreplay/get_next_paket_01)
