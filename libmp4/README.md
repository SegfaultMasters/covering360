
##### Description
The libmp4v2 library provides an abstraction layer for working with files
using the mp4 container format. This library is developed by mpeg4ip project
and is an exact copy of the library distributed in the mpeg4ip package.             
Vendor: -


## Heap Overflow in mp4v2::impl::MP4Track::FinishSdtp()
  
The function `mp4v2::impl::MP4Track::FinishSdtp()` in `mp4track.cpp`  (libmp4v2 - 2.1.0)  mishandles compatibleBrand while processing a crafted mp4 file which leads to heap overflow, causing denial of service.
  
 
###### **Affected version:**
MP4v2 2.1.0



###### **Command**: 
mp4art --add image.png --art-any $POC
 


### Debugging

 ```
  →  601	         const uint32_t max = ftyp->compatibleBrands.GetCount();
```

  
  
 ``` 
gef➤  p ftyp->compatibleBrands
$4 = (mp4v2::impl::MP4StringProperty &) @0x60d00000ce90: {
  <mp4v2::impl::MP4Property> = {
    _vptr.MP4Property = 0x14ffffff00000002, 
    m_parentAtom = @0x4380000100000083, 
    m_name = 0x45534f425245560a <error: Cannot access memory at address 0x45534f425245560a>,    
    m_readOnly = 0x20, 
    m_implicit = 0x4c
  }, 
  members of mp4v2::impl::MP4StringProperty: 
  m_arrayMode = 0x45, 
  m_useCountedFormat = 0x56, 
  m_useExpandedCount = 0x45, 
  m_useUnicode = 0x4c, 
  m_fixedLength = 0x20302020, 
  m_values = {
    <mp4v2::impl::MP4Array> = {
      m_numElements = 0x676e696e, 
      m_maxNumElements = 0x6e612073
    }, 
    members of mp4v2::impl::MP4StringArray: 
    m_elements = 0x73726f7272652064
  }
}
```



### **ASAN output**

```
=================================================================
==5236==ERROR: AddressSanitizer: heap-buffer-overflow on address 0x60d00000ce68 at pc 0x7ffff6a083be bp 0x7fffffffcb60 sp 0x7fffffffcb50
READ of size 8 at 0x60d00000ce68 thread T0
    #0 0x7ffff6a083bd in mp4v2::impl::MP4Track::FinishSdtp() src/mp4track.cpp:601
    #1 0x7ffff6a07c6f in mp4v2::impl::MP4Track::FinishWrite(unsigned int) src/mp4track.cpp:521
    #2 0x7ffff69d02d0 in mp4v2::impl::MP4File::FinishWrite(unsigned int) src/mp4file.cpp:574
    #3 0x7ffff69d09a2 in mp4v2::impl::MP4File::Close(unsigned int) src/mp4file.cpp:619
    #4 0x7ffff69adc47 in MP4Close src/mp4.cpp:261
    #5 0x7ffff6a8d49b in mp4v2::util::Utility::job(std::__cxx11::basic_string<char, std::char_traits<char>, std::allocator<char> >) libutil/Utility.cpp:307
    #6 0x7ffff6a8b9eb in mp4v2::util::Utility::batch(int) libutil/Utility.cpp:104
    #7 0x7ffff6a8fa67 in mp4v2::util::Utility::process_impl() libutil/Utility.cpp:568
    #8 0x7ffff6a8ee84 in mp4v2::util::Utility::process() libutil/Utility.cpp:453
    #9 0x40a2a3 in main util/mp4art.cpp:437
    #10 0x7ffff5ea582f in __libc_start_main (/lib/x86_64-linux-gnu/libc.so.6+0x2082f)
    #11 0x4056a8 in _start (/home/woot/Desktop/mp4v2-master/.libs/mp4art+0x4056a8)

0x60d00000ce68 is located 16 bytes to the right of 136-byte region [0x60d00000cdd0,0x60d00000ce58)
allocated by thread T0 here:
    #0 0x7ffff6f03592 in operator new(unsigned long) (/usr/lib/x86_64-linux-gnu/libasan.so.2+0x99592)
    #1 0x7ffff69c5745 in mp4v2::impl::MP4Atom::factory(mp4v2::impl::MP4File&, mp4v2::impl::MP4Atom*, char const*) src/mp4atom.cpp:1019
    #2 0x7ffff69bf0cd in mp4v2::impl::MP4Atom::CreateAtom(mp4v2::impl::MP4File&, mp4v2::impl::MP4Atom*, char const*) src/mp4atom.cpp:78
    #3 0x7ffff69bf9b7 in mp4v2::impl::MP4Atom::ReadAtom(mp4v2::impl::MP4File&, mp4v2::impl::MP4Atom*) src/mp4atom.cpp:168
    #4 0x7ffff69c1592 in mp4v2::impl::MP4Atom::ReadChildAtoms() src/mp4atom.cpp:429
    #5 0x7ffff69c0085 in mp4v2::impl::MP4Atom::Read() src/mp4atom.cpp:235
    #6 0x7ffff69cf450 in mp4v2::impl::MP4File::ReadFromFile() src/mp4file.cpp:429
    #7 0x7ffff69ccd81 in mp4v2::impl::MP4File::Modify(char const*) src/mp4file.cpp:165
    #8 0x7ffff69ad5b7 in MP4Modify src/mp4.cpp:205
    #9 0x40727c in mp4v2::util::ArtUtility::actionAdd(mp4v2::util::Utility::JobContext&) util/mp4art.cpp:149
    #10 0x409a8d in mp4v2::util::ArtUtility::utility_job(mp4v2::util::Utility::JobContext&) util/mp4art.cpp:369
    #11 0x7ffff6a8d44d in mp4v2::util::Utility::job(std::__cxx11::basic_string<char, std::char_traits<char>, std::allocator<char> >) libutil/Utility.cpp:297
    #12 0x7ffff6a8b9eb in mp4v2::util::Utility::batch(int) libutil/Utility.cpp:104
    #13 0x7ffff6a8fa67 in mp4v2::util::Utility::process_impl() libutil/Utility.cpp:568
    #14 0x7ffff6a8ee84 in mp4v2::util::Utility::process() libutil/Utility.cpp:453
    #15 0x40a2a3 in main util/mp4art.cpp:437
    #16 0x7ffff5ea582f in __libc_start_main (/lib/x86_64-linux-gnu/libc.so.6+0x2082f)

SUMMARY: AddressSanitizer: heap-buffer-overflow src/mp4track.cpp:601 mp4v2::impl::MP4Track::FinishSdtp()
Shadow bytes around the buggy address:
  0x0c1a7fff9970: 00 00 00 00 00 00 00 00 00 00 00 00 00 fa fa fa
  0x0c1a7fff9980: fa fa fa fa fa fa 00 00 00 00 00 00 00 00 00 00
  0x0c1a7fff9990: 00 00 00 00 00 00 00 fa fa fa fa fa fa fa fa fa
  0x0c1a7fff99a0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x0c1a7fff99b0: 00 fa fa fa fa fa fa fa fa fa 00 00 00 00 00 00
=>0x0c1a7fff99c0: 00 00 00 00 00 00 00 00 00 00 00 fa fa[fa]fa fa
  0x0c1a7fff99d0: fa fa fa fa 00 00 00 00 00 00 00 00 00 00 00 00
  0x0c1a7fff99e0: 00 00 00 00 03 fa fa fa fa fa fa fa fa fa fd fd
  0x0c1a7fff99f0: fd fd fd fd fd fd fd fd fd fd fd fd fd fd fd fa
  0x0c1a7fff9a00: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x0c1a7fff9a10: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
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
==5236==ABORTING
```



```
gef➤  x 0x60d00000ce68
0x60d00000ce68:	52880
gef➤  x/32w 0x60d00000ce68
0x60d00000ce68:	52880	24784	0	0
0x60d00000ce78:	0	0	0	0
0x60d00000ce88:	0	0	2	352321535
0x60d00000ce98:	131	1132462081	1380275722	1163087682
0x60d00000cea8:	1447382048	173231173	540024864	1918990112
0x60d00000ceb8:	1735289198	1851859059	1919230052	1936879474
0x60d00000cec8:	824188938	1869488160	1818324338	1718511904
0x60d00000ced8:	1634562671	1702259060	1936026912	1701273971
gef➤  x/32w 0x7fffffffcb50
0x7fffffffcb50:	-13344	32767	52688	24784
0x7fffffffcb60:	-13392	32767	-157252732	32767
0x7fffffffcb70:	-13264	32767	65088	24896
0x7fffffffcb80:	-13408	32767	-157547794	32767
0x7fffffffcb90:	54944	24800	52688	24784
0x7fffffffcba0:	-13232	32767	-1668	4095
0x7fffffffcbb0:	-13152	32767	-157254544	32767
0x7fffffffcbc0:	1	0	65088	24896
```




```
gef➤  i r
rax            0x60d00000cdd0	0x60d00000cdd0
rbx            0xa6	0xa6
rcx            0x7fffffffc901	0x7fffffffc901
rdx            0x0	0x0
rsi            0x0	0x0
rdi            0x7ffff6b2a660	0x7ffff6b2a660
rbp            0x7fffffffcbb0	0x7fffffffcbb0
rsp            0x7fffffffcb70	0x7fffffffcb70
r8             0x10000000000	0x10000000000
r9             0xc1e7fff9d2c	0xc1e7fff9d2c
r10            0x60f00000e8c0	0x60f00000e8c0
r11            0x7ffff7f52bb8	0x7ffff7f52bb8
r12            0x7fffffffcc80	0x7fffffffcc80
r13            0x7fffffffcbe0	0x7fffffffcbe0
r14            0x7fffffffcbe0	0x7fffffffcbe0
r15            0x7fffffffcd00	0x7fffffffcd00
rip            0x7ffff6a08397	0x7ffff6a08397 <mp4v2::impl::MP4Track::FinishSdtp()+325>
eflags         0x202	[ IF ]
cs             0x33	0x33
ss             0x2b	0x2b
ds             0x0	0x0
es             0x0	0x0
fs             0x0	0x0
gs             0x0	0x0
```

