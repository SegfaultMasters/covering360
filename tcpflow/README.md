There exists a stack-based buffer overflow  in `setbit()` function at `iptree.h`, invoked  by `get_histogram()` in `src/netviz/address_histogram.cpp`. The issue gets triggered when the value of `depth` in  `get_histogram()` is set greater than 127 & being passed to `i` in `setbit()`.

The `setbit()` function sets the ith bit to 1, in case where the `i value` exceeds more than 127, the computation goes wrong & a stack overflow is triggered. 



### **Tested version** -
 1.5.0 (Master)

 
### **Command** -
tcpflow -a -D -b -m -o -Fk –r $POC




### **Debugging:** 

```
 /* set the ith bit to 1 */
    static void setbit(uint8_t *addr,size_t i){
        addr[i / 8] |= (1<<((7-i)&7));   //Buffer overflow
    }
```


```
gef➤  p/d i
$6 = 127


gef➤  p/d (1<<((7-i)&7))
$8 = 1

gef➤  p/d addr[127 / 8]
$14 = 0
gef➤  p/d addr[126 / 8]
$15 = 0
gef➤  p/d addr[128 / 8]
$16 = 96

gef➤  p/d (1<<((7-127)&7))
$18 = 1
gef➤  p/d (1<<((7-128)&7))
$19 = 128

```



### **Backtrace:**

```
.
.
.
.
.
#3 0x438649 in iptreet<unsigned long, 16ul>::get_histogram(std::vector<iptreet<unsigned long, 16ul>::addr_elem, std::allocator<iptreet<unsigned long, 16ul>::addr_elem> >&) const iptree.h:402
#4 0x437d92 in address_histogram::address_histogram(iptreet<unsigned long, 16ul> const&) netviz/address_histogram.cpp:30
#5 0x4489eb in one_page_report::render(std::__cxx11::basic_string<char, std::char_traits<char>, std::allocator<char> > const&) netviz/one_page_report.cpp:268
#6 0x4f7307 in scan_netviz /home/ace/Desktop/tcpflow/src/scan_netviz.cpp:74
#7 0x493ebd in be13::plugin::phase_shutdown(feature_recorder_set&, std::__cxx11::basic_stringstream<char, std::char_traits<char>, std::allocator<char> >*) be13_api/plugin.cpp:403
#8 0x4c75ec in main /home/ace/Desktop/tcpflow/src/tcpflow.cpp:909
#9 0x7ffff4ec082f in __libc_start_main (/lib/x86_64-linux-gnu/libc.so.6+0x2082f)
#10 0x408f58 in _start (/usr/bin/tcpflow+0x408f58)
```




### **ASAN Report**

```
==24797==ERROR: AddressSanitizer: stack-buffer-overflow on address 0x7fffffffb840 at pc 0x000000439d28 bp 0x7fffffffb710 sp 0x7fffffffb700
READ of size 1 at 0x7fffffffb840 thread T0
    #0 0x439d27 in iptreet<unsigned long, 16ul>::setbit(unsigned char*, unsigned long) iptree.h:245
    #1 0x438fda in iptreet<unsigned long, 16ul>::get_histogram(int, unsigned char const*, iptreet<unsigned long, 16ul>::node const*, std::vector<iptreet<unsigned long, 16ul>::addr_elem, std::allocator<iptreet<unsigned long, 16ul>::addr_elem> >&) const iptree.h:393
    #2 0x438649 in iptreet<unsigned long, 16ul>::get_histogram(std::vector<iptreet<unsigned long, 16ul>::addr_elem, std::allocator<iptreet<unsigned long, 16ul>::addr_elem> >&) const iptree.h:402
    #3 0x437d92 in address_histogram::address_histogram(iptreet<unsigned long, 16ul> const&) netviz/address_histogram.cpp:30
    #4 0x4489eb in one_page_report::render(std::__cxx11::basic_string<char, std::char_traits<char>, std::allocator<char> > const&) netviz/one_page_report.cpp:268
    #5 0x4f7307 in scan_netviz /home/ace/Desktop/tcpflow/src/scan_netviz.cpp:74
    #6 0x493ebd in be13::plugin::phase_shutdown(feature_recorder_set&, std::__cxx11::basic_stringstream<char, std::char_traits<char>, std::allocator<char> >*) be13_api/plugin.cpp:403
    #7 0x4c75ec in main /home/ace/Desktop/tcpflow/src/tcpflow.cpp:909
    #8 0x7ffff4ec082f in __libc_start_main (/lib/x86_64-linux-gnu/libc.so.6+0x2082f)
    #9 0x408f58 in _start (/usr/bin/tcpflow+0x408f58)

Address 0x7fffffffb840 is located in stack of thread T0 at offset 208 in frame
    #0 0x438ded in iptreet<unsigned long, 16ul>::get_histogram(int, unsigned char const*, iptreet<unsigned long, 16ul>::node const*, std::vector<iptreet<unsigned long, 16ul>::addr_elem, std::allocator<iptreet<unsigned long, 16ul>::addr_elem> >&) const iptree.h:379

  This frame has 3 object(s):
    [32, 72) '<unknown>'
    [128, 144) 'addr0'
    [192, 208) 'addr1' <== Memory access at offset 208 overflows this variable
HINT: this may be a false positive if your program uses some custom stack unwind mechanism or swapcontext
      (longjmp and C++ exceptions *are* supported)
SUMMARY: AddressSanitizer: stack-buffer-overflow iptree.h:245 iptreet<unsigned long, 16ul>::setbit(unsigned char*, unsigned long)
Shadow bytes around the buggy address:
  0x10007fff76b0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x10007fff76c0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x10007fff76d0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x10007fff76e0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 f1 f1
  0x10007fff76f0: f1 f1 00 00 00 00 00 f4 f4 f4 f2 f2 f2 f2 00 00
=>0x10007fff7700: f4 f4 f2 f2 f2 f2 00 00[f4]f4 f3 f3 f3 f3 00 00
  0x10007fff7710: 00 00 00 00 00 00 f1 f1 f1 f1 00 00 f4 f4 f3 f3
  0x10007fff7720: f3 f3 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x10007fff7730: f1 f1 f1 f1 00 f4 f4 f4 f2 f2 f2 f2 00 00 00 f4
  0x10007fff7740: f3 f3 f3 f3 00 00 00 00 00 00 00 00 00 00 00 00
  0x10007fff7750: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
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
==24797==ABORTING
[Inferior 1 (process 24797) exited with code 01]

```



**Note:** Issue reproducible when compiled with AddressSanitizer

[Reproducer file](https://github.com/SegfaultMasters/covering360/blob/master/tcpflow/setbit_00)

