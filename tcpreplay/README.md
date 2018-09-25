##### Description
Tcpreplay is a suite of free Open Source utilities for editing and replaying previously captured network traffic           
Vendor: https://tcpreplay.appneta.com/

## Heap overflow in get_next_packet()

tcpreplay contains a heap-based buffer overflow vulnerability. The `get_next_packet()` function in the `send_packets.c` file uses the `memcpy()` function to copy sequences from the source buffer `pktdata` to the destination `(*prev_packet)->pktdata`. However, there are no checks in place to ensure that `dst` is a non-zero value. An attacker can exploit this vulnerability by submitting a malicious file that exploits this issue. This will result in a Denial of Service (DoS) and potentially Information Exposure when the application attempts to process the file.

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
 
 





## Heap Overflow in fast_edit_packet()


There exists a heap-buffer-overflow in function `fast_edit_packet()` in the file `send_packets.c` of tcpreplay (v4.3).
The issue can be reproduced when provided with an crafted pcap file as an input to the tcpreplay binary. 



###### **Affected version:**
  4.3 branch

  
###### **Command**: 
 sudo tcpreplay -i eno1 -t -K --loop 4 --unique-ip $POC
 
 

### **Debugging**

```
    287      switch (ether_type) {
    288      case ETHERTYPE_IP:
    289          ip_hdr = (ipv4_hdr_t *)(packet + l2_len);
                // ip_hdr=0xbfffeb3c -> [...] -> 0x00000000
-> 290           src_ip_orig = src_ip = ntohl(ip_hdr->ip_src.s_addr); //Overflow triggered 
    291          dst_ip_orig = dst_ip = ntohl(ip_hdr->ip_dst.s_addr);
    292          break;
    293
```



```
gef> p/d ip_hdr->ip_src.s_addr
$33 = 43200
gef> p/d src_ip
$34 = 727806
gef> p/d src_ip_orig
$35 = 28
```



### **ASAN output**

```
==3984==ERROR: AddressSanitizer: heap-buffer-overflow on address 0xb48002ca at pc 0x0804d6c6 bp 0xbfffeb08 sp 0xbfffeaf8
READ of size 4 at 0xb48002ca thread T0
    #0 0x804d6c5 in fast_edit_packet /home/loginsoft/ACE/tcpreplay/src/send_packets.c:290
    #1 0x804f9c0 in send_packets /home/loginsoft/ACE/tcpreplay/src/send_packets.c:569
    #2 0x8060aa4 in replay_file /home/loginsoft/ACE/tcpreplay/src/replay.c:188
    #3 0x805f8c1 in tcpr_replay_index /home/loginsoft/ACE/tcpreplay/src/replay.c:61
    #4 0x805e791 in tcpreplay_replay /home/loginsoft/ACE/tcpreplay/src/tcpreplay_api.c:1135
    #5 0x8056186 in main /home/loginsoft/ACE/tcpreplay/src/tcpreplay.c:139
    #6 0xb784c636 in __libc_start_main (/lib/i386-linux-gnu/libc.so.6+0x18636)
    #7 0x804a7a0  (/usr/local/bin/tcpreplay+0x804a7a0)

0xb48002cc is located 0 bytes to the right of 28-byte region [0xb48002b0,0xb48002cc)
allocated by thread T0 here:
    #0 0xb7ae7dee in malloc (/usr/lib/i386-linux-gnu/libasan.so.2+0x96dee)
    #1 0x8065642 in _our_safe_malloc /home/loginsoft/ACE/tcpreplay/src/common/utils.c:50
    #2 0x8052efd in get_next_packet /home/loginsoft/ACE/tcpreplay/src/send_packets.c:1044
    #3 0x804e921 in preload_pcap_file /home/loginsoft/ACE/tcpreplay/src/send_packets.c:445
    #4 0x805615b in main /home/loginsoft/ACE/tcpreplay/src/tcpreplay.c:126
    #5 0xb784c636 in __libc_start_main (/lib/i386-linux-gnu/libc.so.6+0x18636)

SUMMARY: AddressSanitizer: heap-buffer-overflow /home/loginsoft/ACE/tcpreplay/src/send_packets.c:290 fast_edit_packet
Shadow bytes around the buggy address:
  0x36900000: fa fa 00 00 00 fa fa fa 00 00 00 fa fa fa 00 00
  0x36900010: 00 fa fa fa 00 00 00 fa fa fa 00 00 00 fa fa fa
  0x36900020: 00 00 00 fa fa fa 00 00 00 fa fa fa 00 00 00 fa
  0x36900030: fa fa 00 00 00 fa fa fa 00 00 00 fa fa fa 00 00
  0x36900040: 00 fa fa fa 00 00 00 fa fa fa 00 00 00 fa fa fa
=>0x36900050: 00 00 00 fa fa fa 00 00 00[04]fa fa 00 00 00 fa
  0x36900060: fa fa 00 00 00 fa fa fa 00 00 00 fa fa fa fd fd
  0x36900070: fd fd fa fa fd fd fd fd fa fa fd fd fd fa fa fa
  0x36900080: fd fd fd fa fa fa fd fd fd fa fa fa fd fd fd fa
  0x36900090: fa fa fd fd fd fa fa fa fd fd fd fa fa fa fd fd
  0x369000a0: fd fa fa fa fd fd fd fd fa fa fd fd fd fd fa fa
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
==3984==ABORTING

```





### **glibc detection**

```
*** Error in `tcpreplay': corrupted size vs. prev_size: 0x0825bd50 ***
======= Backtrace: =========
/lib/i386-linux-gnu/libc.so.6(+0x67377)[0xb7d0c377]
/lib/i386-linux-gnu/libc.so.6(+0x6d2f7)[0xb7d122f7]
/lib/i386-linux-gnu/libc.so.6(+0x6d6fe)[0xb7d126fe]
/lib/i386-linux-gnu/libc.so.6(+0x6e395)[0xb7d13395]
tcpreplay[0x8053cf6]
tcpreplay[0x804f7ea]
tcpreplay[0x804e56b]
/lib/i386-linux-gnu/libc.so.6(__libc_start_main+0xf7)[0xb7cbd637]
tcpreplay[0x804a671]
======= Memory map: ========
08048000-0804a000 r--p 00000000 08:01 2641698    /usr/local/bin/tcpreplay
0804a000-08059000 r-xp 00002000 08:01 2641698    /usr/local/bin/tcpreplay
08059000-08060000 r--p 00011000 08:01 2641698    /usr/local/bin/tcpreplay
08060000-08061000 r--p 00017000 08:01 2641698    /usr/local/bin/tcpreplay
08061000-08062000 rw-p 00018000 08:01 2641698    /usr/local/bin/tcpreplay
08062000-08065000 rw-p 00000000 00:00 0
08254000-08275000 rw-p 00000000 00:00 0          [heap]
b7b00000-b7b21000 rw-p 00000000 00:00 0
b7b21000-b7c00000 ---p 00000000 00:00 0
b7ca4000-b7ca5000 rw-p 00000000 00:00 0
b7ca5000-b7e55000 r-xp 00000000 08:01 786798     /lib/i386-linux-gnu/libc-2.23.so
b7e55000-b7e57000 r--p 001af000 08:01 786798     /lib/i386-linux-gnu/libc-2.23.so
b7e57000-b7e58000 rw-p 001b1000 08:01 786798     /lib/i386-linux-gnu/libc-2.23.so
b7e58000-b7e5b000 rw-p 00000000 00:00 0
b7e5b000-b7e7b000 r-xp 00000000 08:01 2241910    /usr/lib/i386-linux-gnu/libopts.so.25.16.1
b7e7b000-b7e7c000 r--p 0001f000 08:01 2241910    /usr/lib/i386-linux-gnu/libopts.so.25.16.1
b7e7c000-b7e7d000 rw-p 00020000 08:01 2241910    /usr/lib/i386-linux-gnu/libopts.so.25.16.1
b7e7d000-b7ebf000 r-xp 00000000 08:01 2230579    /usr/lib/i386-linux-gnu/libpcap.so.1.7.4
b7ebf000-b7ec0000 ---p 00042000 08:01 2230579    /usr/lib/i386-linux-gnu/libpcap.so.1.7.4
b7ec0000-b7ec1000 r--p 00042000 08:01 2230579    /usr/lib/i386-linux-gnu/libpcap.so.1.7.4
b7ec1000-b7ec2000 rw-p 00043000 08:01 2230579    /usr/lib/i386-linux-gnu/libpcap.so.1.7.4
b7ec9000-b7ee5000 r-xp 00000000 08:01 786836     /lib/i386-linux-gnu/libgcc_s.so.1
b7ee5000-b7ee6000 rw-p 0001b000 08:01 786836     /lib/i386-linux-gnu/libgcc_s.so.1
b7ee6000-b7ee8000 rw-p 00000000 00:00 0
b7ee8000-b7eeb000 r--p 00000000 00:00 0          [vvar]
b7eeb000-b7eed000 r-xp 00000000 00:00 0          [vdso]
b7eed000-b7f10000 r-xp 00000000 08:01 786770     /lib/i386-linux-gnu/ld-2.23.so
b7f10000-b7f11000 r--p 00022000 08:01 786770     /lib/i386-linux-gnu/ld-2.23.so
b7f11000-b7f12000 rw-p 00023000 08:01 786770     /lib/i386-linux-gnu/ld-2.23.so
bf8e8000-bf909000 rw-p 00000000 00:00 0          [stack]
Aborted
```

[Reproducer](https://github.com/SegfaultMasters/covering360/blob/master/tcpreplay/fast_edit_package_02)

