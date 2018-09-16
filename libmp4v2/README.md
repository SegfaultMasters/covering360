
##### Description
The libmp4v2 library provides an abstraction layer for working with files
using the mp4 container format. This library is developed by mpeg4ip project
and is an exact copy of the library distributed in the mpeg4ip package.             
Vendor: -


## Heap Overflow in mp4v2::impl::MP4Track::FinishSdtp()
  
The function `mp4v2::impl::MP4Track::FinishSdtp()` in `mp4track.cpp`  (libmp4v2 - 2.1.0)  mishandles compatibleBrand while processing a crafted mp4 file which leads to heap overflow, causing denial of service.
  
 
###### **Affected version:**
libmp4v2 - 2.1.0 (Tested against - Fedora, EPEL, Debian)


###### **Command**: 
./mp4art --add image.png --art-any $POC
 


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





## Invalid free in MP4Free()



##### Description
The function `MP4Free()` in `mp4property.cpp` (libmp4v2 - 2.1.0) internally calls free() on a invalid pointer, raising a SIGABRT signal.


###### **Affected version:**
libmp4v2 - 2.1.0 (Tested against - Fedora, EPEL, Debian)


###### **Command**: 
./mp4art --add image.png --art-any $POC



### Debugging

```
→ 4392	             free(p);
```

```
[#0] 0x7ffff7af8968 → Name: MP4Free(p=0x7ffff74aab78 <main_arena+88>)
[#1] 0x7ffff7b18e8b → Name: mp4v2::impl::MP4StringProperty::~MP4StringProperty(this=0x63a560, __in_chrg=<optimized out>)
[#2] 0x7ffff7b18f12 → Name: mp4v2::impl::MP4StringProperty::~MP4StringProperty(this=0x63a560, __in_chrg=<optimized out>)
[#3] 0x7ffff7af98a0 → Name: mp4v2::impl::MP4Atom::~MP4Atom(this=0x63a470, __in_chrg=<optimized out>)
[#4] 0x7ffff7acf3f2 → Name: mp4v2::impl::MP4FtypAtom::~MP4FtypAtom(this=0x63a470, __in_chrg=<optimized out>)
[#5] 0x7ffff7acf422 → Name: mp4v2::impl::MP4FtypAtom::~MP4FtypAtom(this=0x63a470, __in_chrg=<optimized out>)
[#6] 0x7ffff7afa1b8 → Name: mp4v2::impl::MP4Atom::ReadAtom(file=@0x639990, pParentAtom=0x63a2f0)
[#7] 0x7ffff7afaea6 → Name: mp4v2::impl::MP4Atom::ReadChildAtoms(this=0x63a2f0)
[#8] 0x7ffff7afa361 → Name: mp4v2::impl::MP4Atom::Read(this=0x63a2f0)
[#9] 0x7ffff7b02d3e → Name: mp4v2::impl::MP4File::ReadFromFile(this=0x639990)



[#0] 0x7ffff711b428 → Name: __GI_raise(sig=0x6)
[#1] 0x7ffff711d02a → Name: __GI_abort()
[#2] 0x7ffff715d7ea → Name: __libc_message(do_abort=0x2, fmt=0x7ffff7276ed8 "*** Error in `%s': %s: 0x%s ***\n")
[#3] 0x7ffff716637a → Name: malloc_printerr(ar_ptr=<optimized out>, ptr=<optimized out>, str=0x7ffff7273caf "free(): invalid pointer", action=0x3)
[#4] 0x7ffff716637a → Name: _int_free(av=<optimized out>, p=<optimized out>, have_lock=0x0)
[#5] 0x7ffff716a53c → Name: __GI___libc_free(mem=<optimized out>)
[#6] 0x7ffff7af8974 → Name: MP4Free(p=0x7ffff74aab78 <main_arena+88>)
[#7] 0x7ffff7b18e8b → Name: mp4v2::impl::MP4StringProperty::~MP4StringProperty(this=0x63a560, __in_chrg=<optimized out>)
[#8] 0x7ffff7b18f12 → Name: mp4v2::impl::MP4StringProperty::~MP4StringProperty(this=0x63a560, __in_chrg=<optimized out>)
[#9] 0x7ffff7af98a0 → Name: mp4v2::impl::MP4Atom::~MP4Atom(this=0x63a470, __in_chrg=<optimized out>)
```


```
gef➤  p p
$38 = (void *) 0x63a520


gef➤  p p
$39 = (void *) 0x63a540


gef➤  p p
$48 = (void *) 0x7ffff74aab78 <main_arena+88>
```




### Valgrind report 

```
==57947== Invalid read of size 8
==57947==    at 0x4EFEE80: mp4v2::impl::MP4StringProperty::~MP4StringProperty() (mp4property.cpp:338)
==57947==    by 0x4EFEF11: mp4v2::impl::MP4StringProperty::~MP4StringProperty() (mp4property.cpp:340)
==57947==    by 0x4EDF89F: mp4v2::impl::MP4Atom::~MP4Atom() (mp4atom.cpp:66)
==57947==    by 0x4EB53F1: mp4v2::impl::MP4FtypAtom::~MP4FtypAtom() (in /usr/local/lib/libmp4v2.so.2.1.0)
==57947==    by 0x4EB5421: mp4v2::impl::MP4FtypAtom::~MP4FtypAtom() (atoms.h:325)
==57947==    by 0x4EE01B7: mp4v2::impl::MP4Atom::ReadAtom(mp4v2::impl::MP4File&, mp4v2::impl::MP4Atom*) (mp4atom.cpp:199)
==57947==    by 0x4EE0EA5: mp4v2::impl::MP4Atom::ReadChildAtoms() (mp4atom.cpp:429)
==57947==    by 0x4EE0360: mp4v2::impl::MP4Atom::Read() (mp4atom.cpp:235)
==57947==    by 0x4EE8D3D: mp4v2::impl::MP4File::ReadFromFile() (mp4file.cpp:430)
==57947==    by 0x4EE7851: mp4v2::impl::MP4File::Modify(char const*) (mp4file.cpp:166)
==57947==    by 0x4ED333A: MP4Modify (mp4.cpp:231)
==57947==    by 0x406547: mp4v2::util::ArtUtility::actionAdd(mp4v2::util::Utility::JobContext&) (mp4art.cpp:149)
==57947==  Address 0x5e7dff8 is 0 bytes after a block of size 8 alloc'd
==57947==    at 0x4C2DB8F: malloc (in /usr/lib/valgrind/vgpreload_memcheck-amd64-linux.so)
==57947==    by 0x4C2FDEF: realloc (in /usr/lib/valgrind/vgpreload_memcheck-amd64-linux.so)
==57947==    by 0x4EAE618: mp4v2::impl::MP4Realloc(void*, unsigned int) (mp4util.h:80)
==57947==    by 0x4F03DA2: mp4v2::impl::MP4StringArray::Resize(unsigned int) (mp4array.h:136)
==57947==    by 0x4EFEF56: mp4v2::impl::MP4StringProperty::SetCount(unsigned int) (mp4property.cpp:346)
==57947==    by 0x4EFEDE1: mp4v2::impl::MP4StringProperty::MP4StringProperty(mp4v2::impl::MP4Atom&, char const*, bool, bool, bool) (mp4property.cpp:330)
==57947==    by 0x4EB5171: mp4v2::impl::MP4FtypAtom::MP4FtypAtom(mp4v2::impl::MP4File&) (atom_ftyp.cpp:32)
==57947==    by 0x4EE2DBC: mp4v2::impl::MP4Atom::factory(mp4v2::impl::MP4File&, mp4v2::impl::MP4Atom*, char const*) (mp4atom.cpp:868)
==57947==    by 0x4EDFA3A: mp4v2::impl::MP4Atom::CreateAtom(mp4v2::impl::MP4File&, mp4v2::impl::MP4Atom*, char const*) (mp4atom.cpp:78)
==57947==    by 0x4EDFF89: mp4v2::impl::MP4Atom::ReadAtom(mp4v2::impl::MP4File&, mp4v2::impl::MP4Atom*) (mp4atom.cpp:168)
==57947==    by 0x4EE0EA5: mp4v2::impl::MP4Atom::ReadChildAtoms() (mp4atom.cpp:429)
==57947==    by 0x4EE0360: mp4v2::impl::MP4Atom::Read() (mp4atom.cpp:235)
==57947== 
==57947== Invalid free() / delete / delete[] / realloc()
==57947==    at 0x4C2EDEB: free (in /usr/lib/valgrind/vgpreload_memcheck-amd64-linux.so)
==57947==    by 0x4EDE973: MP4Free (mp4.cpp:4392)
==57947==    by 0x4EFEE8A: mp4v2::impl::MP4StringProperty::~MP4StringProperty() (mp4property.cpp:338)
==57947==    by 0x4EFEF11: mp4v2::impl::MP4StringProperty::~MP4StringProperty() (mp4property.cpp:340)
==57947==    by 0x4EDF89F: mp4v2::impl::MP4Atom::~MP4Atom() (mp4atom.cpp:66)
==57947==    by 0x4EB53F1: mp4v2::impl::MP4FtypAtom::~MP4FtypAtom() (in /usr/local/lib/libmp4v2.so.2.1.0)
==57947==    by 0x4EB5421: mp4v2::impl::MP4FtypAtom::~MP4FtypAtom() (atoms.h:325)
==57947==    by 0x4EE01B7: mp4v2::impl::MP4Atom::ReadAtom(mp4v2::impl::MP4File&, mp4v2::impl::MP4Atom*) (mp4atom.cpp:199)
==57947==    by 0x4EE0EA5: mp4v2::impl::MP4Atom::ReadChildAtoms() (mp4atom.cpp:429)
==57947==    by 0x4EE0360: mp4v2::impl::MP4Atom::Read() (mp4atom.cpp:235)
==57947==    by 0x4EE8D3D: mp4v2::impl::MP4File::ReadFromFile() (mp4file.cpp:430)
==57947==    by 0x4EE7851: mp4v2::impl::MP4File::Modify(char const*) (mp4file.cpp:166)
==57947==  Address 0x50 is not stack'd, malloc'd or (recently) free'd
==57947== 
==57947== Mismatched free() / delete / delete []
==57947==    at 0x4C2EDEB: free (in /usr/lib/valgrind/vgpreload_memcheck-amd64-linux.so)
==57947==    by 0x4EDE973: MP4Free (mp4.cpp:4392)
==57947==    by 0x4EFEE8A: mp4v2::impl::MP4StringProperty::~MP4StringProperty() (mp4property.cpp:338)
==57947==    by 0x4EFEF11: mp4v2::impl::MP4StringProperty::~MP4StringProperty() (mp4property.cpp:340)
==57947==    by 0x4EDF89F: mp4v2::impl::MP4Atom::~MP4Atom() (mp4atom.cpp:66)
==57947==    by 0x4EB53F1: mp4v2::impl::MP4FtypAtom::~MP4FtypAtom() (in /usr/local/lib/libmp4v2.so.2.1.0)
==57947==    by 0x4EB5421: mp4v2::impl::MP4FtypAtom::~MP4FtypAtom() (atoms.h:325)
==57947==    by 0x4EE01B7: mp4v2::impl::MP4Atom::ReadAtom(mp4v2::impl::MP4File&, mp4v2::impl::MP4Atom*) (mp4atom.cpp:199)
==57947==    by 0x4EE0EA5: mp4v2::impl::MP4Atom::ReadChildAtoms() (mp4atom.cpp:429)
==57947==    by 0x4EE0360: mp4v2::impl::MP4Atom::Read() (mp4atom.cpp:235)
==57947==    by 0x4EE8D3D: mp4v2::impl::MP4File::ReadFromFile() (mp4file.cpp:430)
==57947==    by 0x4EE7851: mp4v2::impl::MP4File::Modify(char const*) (mp4file.cpp:166)
==57947==  Address 0x5e7df70 is 0 bytes inside a block of size 56 alloc'd
==57947==    at 0x4C2E0EF: operator new(unsigned long) (in /usr/lib/valgrind/vgpreload_memcheck-amd64-linux.so)
==57947==    by 0x4EB5147: mp4v2::impl::MP4FtypAtom::MP4FtypAtom(mp4v2::impl::MP4File&) (atom_ftyp.cpp:32)
==57947==    by 0x4EE2DBC: mp4v2::impl::MP4Atom::factory(mp4v2::impl::MP4File&, mp4v2::impl::MP4Atom*, char const*) (mp4atom.cpp:868)
==57947==    by 0x4EDFA3A: mp4v2::impl::MP4Atom::CreateAtom(mp4v2::impl::MP4File&, mp4v2::impl::MP4Atom*, char const*) (mp4atom.cpp:78)
==57947==    by 0x4EDFF89: mp4v2::impl::MP4Atom::ReadAtom(mp4v2::impl::MP4File&, mp4v2::impl::MP4Atom*) (mp4atom.cpp:168)
==57947==    by 0x4EE0EA5: mp4v2::impl::MP4Atom::ReadChildAtoms() (mp4atom.cpp:429)
==57947==    by 0x4EE0360: mp4v2::impl::MP4Atom::Read() (mp4atom.cpp:235)
==57947==    by 0x4EE8D3D: mp4v2::impl::MP4File::ReadFromFile() (mp4file.cpp:430)
==57947==    by 0x4EE7851: mp4v2::impl::MP4File::Modify(char const*) (mp4file.cpp:166)
==57947==    by 0x4ED333A: MP4Modify (mp4.cpp:231)
==57947==    by 0x406547: mp4v2::util::ArtUtility::actionAdd(mp4v2::util::Utility::JobContext&) (mp4art.cpp:149)
==57947==    by 0x407B13: mp4v2::util::ArtUtility::utility_job(mp4v2::util::Utility::JobContext&) (mp4art.cpp:369)
==57947== 
==57947== Invalid read of size 4
==57947==    at 0x4EADD6D: mp4v2::impl::MP4Array::ValidIndex(unsigned int) (mp4array.h:40)
==57947==    by 0x4EB9A72: mp4v2::impl::MP4StringArray::operator[](unsigned int) (in /usr/local/lib/libmp4v2.so.2.1.0)
==57947==    by 0x4EFEE7F: mp4v2::impl::MP4StringProperty::~MP4StringProperty() (mp4property.cpp:338)
==57947==    by 0x4EFEF11: mp4v2::impl::MP4StringProperty::~MP4StringProperty() (mp4property.cpp:340)
==57947==    by 0x4EDF89F: mp4v2::impl::MP4Atom::~MP4Atom() (mp4atom.cpp:66)
==57947==    by 0x4EB53F1: mp4v2::impl::MP4FtypAtom::~MP4FtypAtom() (in /usr/local/lib/libmp4v2.so.2.1.0)
==57947==    by 0x4EB5421: mp4v2::impl::MP4FtypAtom::~MP4FtypAtom() (atoms.h:325)
==57947==    by 0x4EE01B7: mp4v2::impl::MP4Atom::ReadAtom(mp4v2::impl::MP4File&, mp4v2::impl::MP4Atom*) (mp4atom.cpp:199)
==57947==    by 0x4EE0EA5: mp4v2::impl::MP4Atom::ReadChildAtoms() (mp4atom.cpp:429)
==57947==    by 0x4EE0360: mp4v2::impl::MP4Atom::Read() (mp4atom.cpp:235)
==57947==    by 0x4EE8D3D: mp4v2::impl::MP4File::ReadFromFile() (mp4file.cpp:430)
==57947==    by 0x4EE7851: mp4v2::impl::MP4File::Modify(char const*) (mp4file.cpp:166)
==57947==  Address 0x5e7df98 is 40 bytes inside a block of size 56 free'd
==57947==    at 0x4C2EDEB: free (in /usr/lib/valgrind/vgpreload_memcheck-amd64-linux.so)
==57947==    by 0x4EDE973: MP4Free (mp4.cpp:4392)
==57947==    by 0x4EFEE8A: mp4v2::impl::MP4StringProperty::~MP4StringProperty() (mp4property.cpp:338)
==57947==    by 0x4EFEF11: mp4v2::impl::MP4StringProperty::~MP4StringProperty() (mp4property.cpp:340)
==57947==    by 0x4EDF89F: mp4v2::impl::MP4Atom::~MP4Atom() (mp4atom.cpp:66)
==57947==    by 0x4EB53F1: mp4v2::impl::MP4FtypAtom::~MP4FtypAtom() (in /usr/local/lib/libmp4v2.so.2.1.0)
==57947==    by 0x4EB5421: mp4v2::impl::MP4FtypAtom::~MP4FtypAtom() (atoms.h:325)
==57947==    by 0x4EE01B7: mp4v2::impl::MP4Atom::ReadAtom(mp4v2::impl::MP4File&, mp4v2::impl::MP4Atom*) (mp4atom.cpp:199)
==57947==    by 0x4EE0EA5: mp4v2::impl::MP4Atom::ReadChildAtoms() (mp4atom.cpp:429)
==57947==    by 0x4EE0360: mp4v2::impl::MP4Atom::Read() (mp4atom.cpp:235)
==57947==    by 0x4EE8D3D: mp4v2::impl::MP4File::ReadFromFile() (mp4file.cpp:430)
==57947==    by 0x4EE7851: mp4v2::impl::MP4File::Modify(char const*) (mp4file.cpp:166)
==57947==  Block was alloc'd at
==57947==    at 0x4C2E0EF: operator new(unsigned long) (in /usr/lib/valgrind/vgpreload_memcheck-amd64-linux.so)
==57947==    by 0x4EB5147: mp4v2::impl::MP4FtypAtom::MP4FtypAtom(mp4v2::impl::MP4File&) (atom_ftyp.cpp:32)
==57947==    by 0x4EE2DBC: mp4v2::impl::MP4Atom::factory(mp4v2::impl::MP4File&, mp4v2::impl::MP4Atom*, char const*) (mp4atom.cpp:868)
==57947==    by 0x4EDFA3A: mp4v2::impl::MP4Atom::CreateAtom(mp4v2::impl::MP4File&, mp4v2::impl::MP4Atom*, char const*) (mp4atom.cpp:78)
==57947==    by 0x4EDFF89: mp4v2::impl::MP4Atom::ReadAtom(mp4v2::impl::MP4File&, mp4v2::impl::MP4Atom*) (mp4atom.cpp:168)
==57947==    by 0x4EE0EA5: mp4v2::impl::MP4Atom::ReadChildAtoms() (mp4atom.cpp:429)
==57947==    by 0x4EE0360: mp4v2::impl::MP4Atom::Read() (mp4atom.cpp:235)
==57947==    by 0x4EE8D3D: mp4v2::impl::MP4File::ReadFromFile() (mp4file.cpp:430)
==57947==    by 0x4EE7851: mp4v2::impl::MP4File::Modify(char const*) (mp4file.cpp:166)
==57947==    by 0x4ED333A: MP4Modify (mp4.cpp:231)
==57947==    by 0x406547: mp4v2::util::ArtUtility::actionAdd(mp4v2::util::Utility::JobContext&) (mp4art.cpp:149)
==57947==    by 0x407B13: mp4v2::util::ArtUtility::utility_job(mp4v2::util::Utility::JobContext&) (mp4art.cpp:369)
==57947== 
==57947== Invalid read of size 8
==57947==    at 0x4EB9A7E: mp4v2::impl::MP4StringArray::operator[](unsigned int) (in /usr/local/lib/libmp4v2.so.2.1.0)
==57947==    by 0x4EFEE7F: mp4v2::impl::MP4StringProperty::~MP4StringProperty() (mp4property.cpp:338)
==57947==    by 0x4EFEF11: mp4v2::impl::MP4StringProperty::~MP4StringProperty() (mp4property.cpp:340)
==57947==    by 0x4EDF89F: mp4v2::impl::MP4Atom::~MP4Atom() (mp4atom.cpp:66)
==57947==    by 0x4EB53F1: mp4v2::impl::MP4FtypAtom::~MP4FtypAtom() (in /usr/local/lib/libmp4v2.so.2.1.0)
==57947==    by 0x4EB5421: mp4v2::impl::MP4FtypAtom::~MP4FtypAtom() (atoms.h:325)
==57947==    by 0x4EE01B7: mp4v2::impl::MP4Atom::ReadAtom(mp4v2::impl::MP4File&, mp4v2::impl::MP4Atom*) (mp4atom.cpp:199)
==57947==    by 0x4EE0EA5: mp4v2::impl::MP4Atom::ReadChildAtoms() (mp4atom.cpp:429)
==57947==    by 0x4EE0360: mp4v2::impl::MP4Atom::Read() (mp4atom.cpp:235)
==57947==    by 0x4EE8D3D: mp4v2::impl::MP4File::ReadFromFile() (mp4file.cpp:430)
==57947==    by 0x4EE7851: mp4v2::impl::MP4File::Modify(char const*) (mp4file.cpp:166)
==57947==    by 0x4ED333A: MP4Modify (mp4.cpp:231)
==57947==  Address 0x5e7dfa0 is 48 bytes inside a block of size 56 free'd
==57947==    at 0x4C2EDEB: free (in /usr/lib/valgrind/vgpreload_memcheck-amd64-linux.so)
==57947==    by 0x4EDE973: MP4Free (mp4.cpp:4392)
==57947==    by 0x4EFEE8A: mp4v2::impl::MP4StringProperty::~MP4StringProperty() (mp4property.cpp:338)
==57947==    by 0x4EFEF11: mp4v2::impl::MP4StringProperty::~MP4StringProperty() (mp4property.cpp:340)
==57947==    by 0x4EDF89F: mp4v2::impl::MP4Atom::~MP4Atom() (mp4atom.cpp:66)
==57947==    by 0x4EB53F1: mp4v2::impl::MP4FtypAtom::~MP4FtypAtom() (in /usr/local/lib/libmp4v2.so.2.1.0)
==57947==    by 0x4EB5421: mp4v2::impl::MP4FtypAtom::~MP4FtypAtom() (atoms.h:325)
==57947==    by 0x4EE01B7: mp4v2::impl::MP4Atom::ReadAtom(mp4v2::impl::MP4File&, mp4v2::impl::MP4Atom*) (mp4atom.cpp:199)
==57947==    by 0x4EE0EA5: mp4v2::impl::MP4Atom::ReadChildAtoms() (mp4atom.cpp:429)
==57947==    by 0x4EE0360: mp4v2::impl::MP4Atom::Read() (mp4atom.cpp:235)
==57947==    by 0x4EE8D3D: mp4v2::impl::MP4File::ReadFromFile() (mp4file.cpp:430)
==57947==    by 0x4EE7851: mp4v2::impl::MP4File::Modify(char const*) (mp4file.cpp:166)
==57947==  Block was alloc'd at
==57947==    at 0x4C2E0EF: operator new(unsigned long) (in /usr/lib/valgrind/vgpreload_memcheck-amd64-linux.so)
==57947==    by 0x4EB5147: mp4v2::impl::MP4FtypAtom::MP4FtypAtom(mp4v2::impl::MP4File&) (atom_ftyp.cpp:32)
==57947==    by 0x4EE2DBC: mp4v2::impl::MP4Atom::factory(mp4v2::impl::MP4File&, mp4v2::impl::MP4Atom*, char const*) (mp4atom.cpp:868)
==57947==    by 0x4EDFA3A: mp4v2::impl::MP4Atom::CreateAtom(mp4v2::impl::MP4File&, mp4v2::impl::MP4Atom*, char const*) (mp4atom.cpp:78)
==57947==    by 0x4EDFF89: mp4v2::impl::MP4Atom::ReadAtom(mp4v2::impl::MP4File&, mp4v2::impl::MP4Atom*) (mp4atom.cpp:168)
==57947==    by 0x4EE0EA5: mp4v2::impl::MP4Atom::ReadChildAtoms() (mp4atom.cpp:429)
==57947==    by 0x4EE0360: mp4v2::impl::MP4Atom::Read() (mp4atom.cpp:235)
==57947==    by 0x4EE8D3D: mp4v2::impl::MP4File::ReadFromFile() (mp4file.cpp:430)
==57947==    by 0x4EE7851: mp4v2::impl::MP4File::Modify(char const*) (mp4file.cpp:166)
==57947==    by 0x4ED333A: MP4Modify (mp4.cpp:231)
==57947==    by 0x406547: mp4v2::util::ArtUtility::actionAdd(mp4v2::util::Utility::JobContext&) (mp4art.cpp:149)
==57947==    by 0x407B13: mp4v2::util::ArtUtility::utility_job(mp4v2::util::Utility::JobContext&) (mp4art.cpp:369)
==57947== 
==57947== Conditional jump or move depends on uninitialised value(s)
==57947==    at 0x4EDE966: MP4Free (mp4.cpp:4391)
==57947==    by 0x4EFEE8A: mp4v2::impl::MP4StringProperty::~MP4StringProperty() (mp4property.cpp:338)
==57947==    by 0x4EFEF11: mp4v2::impl::MP4StringProperty::~MP4StringProperty() (mp4property.cpp:340)
==57947==    by 0x4EDF89F: mp4v2::impl::MP4Atom::~MP4Atom() (mp4atom.cpp:66)
==57947==    by 0x4EB53F1: mp4v2::impl::MP4FtypAtom::~MP4FtypAtom() (in /usr/local/lib/libmp4v2.so.2.1.0)
==57947==    by 0x4EB5421: mp4v2::impl::MP4FtypAtom::~MP4FtypAtom() (atoms.h:325)
==57947==    by 0x4EE01B7: mp4v2::impl::MP4Atom::ReadAtom(mp4v2::impl::MP4File&, mp4v2::impl::MP4Atom*) (mp4atom.cpp:199)
==57947==    by 0x4EE0EA5: mp4v2::impl::MP4Atom::ReadChildAtoms() (mp4atom.cpp:429)
==57947==    by 0x4EE0360: mp4v2::impl::MP4Atom::Read() (mp4atom.cpp:235)
==57947==    by 0x4EE8D3D: mp4v2::impl::MP4File::ReadFromFile() (mp4file.cpp:430)
==57947==    by 0x4EE7851: mp4v2::impl::MP4File::Modify(char const*) (mp4file.cpp:166)
==57947==    by 0x4ED333A: MP4Modify (mp4.cpp:231)
==57947== 
==57947== 
==57947== Process terminating with default action of signal 11 (SIGSEGV): dumping core
==57947==  Access not within mapped region at address 0x6234000
==57947==    at 0x4EFEE80: mp4v2::impl::MP4StringProperty::~MP4StringProperty() (mp4property.cpp:338)
==57947==    by 0x4EFEF11: mp4v2::impl::MP4StringProperty::~MP4StringProperty() (mp4property.cpp:340)
==57947==    by 0x4EDF89F: mp4v2::impl::MP4Atom::~MP4Atom() (mp4atom.cpp:66)
==57947==    by 0x4EB53F1: mp4v2::impl::MP4FtypAtom::~MP4FtypAtom() (in /usr/local/lib/libmp4v2.so.2.1.0)
==57947==    by 0x4EB5421: mp4v2::impl::MP4FtypAtom::~MP4FtypAtom() (atoms.h:325)
==57947==    by 0x4EE01B7: mp4v2::impl::MP4Atom::ReadAtom(mp4v2::impl::MP4File&, mp4v2::impl::MP4Atom*) (mp4atom.cpp:199)
==57947==    by 0x4EE0EA5: mp4v2::impl::MP4Atom::ReadChildAtoms() (mp4atom.cpp:429)
==57947==    by 0x4EE0360: mp4v2::impl::MP4Atom::Read() (mp4atom.cpp:235)
==57947==    by 0x4EE8D3D: mp4v2::impl::MP4File::ReadFromFile() (mp4file.cpp:430)
==57947==    by 0x4EE7851: mp4v2::impl::MP4File::Modify(char const*) (mp4file.cpp:166)
==57947==    by 0x4ED333A: MP4Modify (mp4.cpp:231)
==57947==    by 0x406547: mp4v2::util::ArtUtility::actionAdd(mp4v2::util::Utility::JobContext&) (mp4art.cpp:149)
```
