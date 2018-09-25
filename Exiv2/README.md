## Stack overflow due to excessive stack consumption (Recursive function) 

##### Description
A stack overflow exits in `CiffDirectory::readDirectory()` at `crwimage_int.cpp` due to a recursive function call causing the excessive stack consumption which leads to Denial of service.

 

###### **Affected version:**
 0.27.0.0 (64 bit build)



###### **Command:**
./exiv2 -pi $POC


### **Debugging**



```
   0x7ffff6f0d35a                  call   0x7ffff6e86bc0 <__stack_chk_fail@plt>
   0x7ffff6f0d35f                  nop    
   0x7ffff6f0d360                  sub    rsp, 0x8
 → 0x7ffff6f0d364                  call   0x7ffff6f06340
   ↳  0x7ffff6f06340                  cmp    BYTE PTR [rip+0x47b341], 0x0        # 0x7ffff7381688
      0x7ffff6f06347                  je     0x7ffff6f06358
      0x7ffff6f06349                  mov    edi, DWORD PTR [rip+0x47b33d]        # 0x7ffff738168c
      0x7ffff6f0634f                  jmp    0x7ffff6e86e00 <pthread_getspecific@plt>
      0x7ffff6f06354                  nop    DWORD PTR [rax+0x0]
      0x7ffff6f06358                  lea    rdx, [rip+0x34986]        # 0x7ffff6f3ace5
```



### ASAN Output

``` 
ASAN:SIGSEGV
=================================================================
==63879==ERROR: AddressSanitizer: stack-overflow on address 0x7ffd1c5e0ff8 (pc 0x7fce8fe99252 bp 0x000000000050 sp 0x7ffd1c5e1000 T0)
    #0 0x7fce8fe99251  (/usr/lib/x86_64-linux-gnu/libasan.so.2+0xb0251)
    #1 0x7fce8fe98d97  (/usr/lib/x86_64-linux-gnu/libasan.so.2+0xafd97)
    #2 0x7fce8fe0bf0f  (/usr/lib/x86_64-linux-gnu/libasan.so.2+0x22f0f)
    #3 0x7fce8fe8255e in operator new(unsigned long) (/usr/lib/x86_64-linux-gnu/libasan.so.2+0x9955e)
    #4 0x7fce8f79c94c in Exiv2::Internal::CiffDirectory::readDirectory(unsigned char const*, unsigned int, Exiv2::ByteOrder) /home/woot/Desktop/exiv2/src/crwimage_int.cpp:292
    #5 0x7fce8f79c733 in Exiv2::Internal::CiffDirectory::doRead(unsigned char const*, unsigned int, unsigned int, Exiv2::ByteOrder) /home/woot/Desktop/exiv2/src/crwimage_int.cpp:269
    #6 0x7fce8f79c318 in Exiv2::Internal::CiffComponent::read(unsigned char const*, unsigned int, unsigned int, Exiv2::ByteOrder) /home/woot/Desktop/exiv2/src/crwimage_int.cpp:227
    #7 0x7fce8f79ca54 in Exiv2::Internal::CiffDirectory::readDirectory(unsigned char const*, unsigned int, Exiv2::ByteOrder) /home/woot/Desktop/exiv2/src/crwimage_int.cpp:296
    #8 0x7fce8f79c733 in Exiv2::Internal::CiffDirectory::doRead(unsigned char const*, unsigned int, unsigned int, Exiv2::ByteOrder) /home/woot/Desktop/exiv2/src/crwimage_int.cpp:269
    #9 0x7fce8f79c318 in Exiv2::Internal::CiffComponent::read(unsigned char const*, unsigned int, unsigned int, Exiv2::ByteOrder) /home/woot/Desktop/exiv2/src/crwimage_int.cpp:227
    #10 0x7fce8f79ca54 in Exiv2::Internal::CiffDirectory::readDirectory(unsigned char const*, unsigned int, Exiv2::ByteOrder) /home/woot/Desktop/exiv2/src/crwimage_int.cpp:296
    #11 0x7fce8f79c733 in Exiv2::Internal::CiffDirectory::doRead(unsigned char const*, unsigned int, unsigned int, Exiv2::ByteOrder) /home/woot/Desktop/exiv2/src/crwimage_int.cpp:269
    #12 0x7fce8f79c318 in Exiv2::Internal::CiffComponent::read(unsigned char const*, unsigned int, unsigned int, Exiv2::ByteOrder) /home/woot/Desktop/exiv2/src/crwimage_int.cpp:227
    #13 0x7fce8f79ca54 in Exiv2::Internal::CiffDirectory::readDirectory(unsigned char const*, unsigned int, Exiv2::ByteOrder) /home/woot/Desktop/exiv2/src/crwimage_int.cpp:296
    #14 0x7fce8f79c733 in Exiv2::Internal::CiffDirectory::doRead(unsigned char const*, unsigned int, unsigned int, Exiv2::ByteOrder) /home/woot/Desktop/exiv2/src/crwimage_int.cpp:269
    #15 0x7fce8f79c318 in Exiv2::Internal::CiffComponent::read(unsigned char const*, unsigned int, unsigned int, Exiv2::ByteOrder) /home/woot/Desktop/exiv2/src/crwimage_int.cpp:227
    #16 0x7fce8f79ca54 in Exiv2::Internal::CiffDirectory::readDirectory(unsigned char const*, unsigned int, Exiv2::ByteOrder) /home/woot/Desktop/exiv2/src/crwimage_int.cpp:296
    #17 0x7fce8f79c733 in Exiv2::Internal::CiffDirectory::doRead(unsigned char const*, unsigned int, unsigned int, Exiv2::ByteOrder) /home/woot/Desktop/exiv2/src/crwimage_int.cpp:269
    #18 0x7fce8f79c318 in Exiv2::Internal::CiffComponent::read(unsigned char const*, unsigned int, unsigned int, Exiv2::ByteOrder) /home/woot/Desktop/exiv2/src/crwimage_int.cpp:227
    #19 0x7fce8f79ca54 in Exiv2::Internal::CiffDirectory::readDirectory(unsigned char const*, unsigned int, Exiv2::ByteOrder) /home/woot/Desktop/exiv2/src/crwimage_int.cpp:296
    #20 0x7fce8f79c733 in Exiv2::Internal::CiffDirectory::doRead(unsigned char const*, unsigned int, unsigned int, Exiv2::ByteOrder) /home/woot/Desktop/exiv2/src/crwimage_int.cpp:269
    #21 0x7fce8f79c318 in Exiv2::Internal::CiffComponent::read(unsigned char const*, unsigned int, unsigned int, Exiv2::ByteOrder) /home/woot/Desktop/exiv2/src/crwimage_int.cpp:227
    #22 0x7fce8f79ca54 in Exiv2::Internal::CiffDirectory::readDirectory(unsigned char const*, unsigned int, Exiv2::ByteOrder) /home/woot/Desktop/exiv2/src/crwimage_int.cpp:296
    #23 0x7fce8f79c733 in Exiv2::Internal::CiffDirectory::doRead(unsigned char const*, unsigned int, unsigned int, Exiv2::ByteOrder) /home/woot/Desktop/exiv2/src/crwimage_int.cpp:269
.
.
.
.
.
.
.
.
    #68 0x7fce8f79c733 in Exiv2::Internal::CiffDirectory::doRead(unsigned char const*, unsigned int, unsigned int, Exiv2::ByteOrder) /home/woot/Desktop/exiv2/src/crwimage_int.cpp:269
    #69 0x7fce8f79c318 in Exiv2::Internal::CiffComponent::read(unsigned char const*, unsigned int, unsigned int, Exiv2::ByteOrder) /home/woot/Desktop/exiv2/src/crwimage_int.cpp:227
    #70 0x7fce8f79ca54 in Exiv2::Internal::CiffDirectory::readDirectory(unsigned char const*, unsigned int, Exiv2::ByteOrder) /home/woot/Desktop/exiv2/src/crwimage_int.cpp:296
    #71 0x7fce8f79c733 in Exiv2::Internal::CiffDirectory::doRead(unsigned char const*, unsigned int, unsigned int, Exiv2::ByteOrder) /home/woot/Desktop/exiv2/src/crwimage_int.cpp:269
    #72 0x7fce8f79c318 in Exiv2::Internal::CiffComponent::read(unsigned char const*, unsigned int, unsigned int, Exiv2::ByteOrder) /home/woot/Desktop/exiv2/src/crwimage_int.cpp:227
    #73 0x7fce8f79ca54 in Exiv2::Internal::CiffDirectory::readDirectory(unsigned char const*, unsigned int, Exiv2::ByteOrder) /home/woot/Desktop/exiv2/src/crwimage_int.cpp:296
    #74 0x7fce8f79c733 in Exiv2::Internal::CiffDirectory::doRead(unsigned char const*, unsigned int, unsigned int, Exiv2::ByteOrder) /home/woot/Desktop/exiv2/src/crwimage_int.cpp:269
    #75 0x7fce8f79c318 in Exiv2::Internal::CiffComponent::read(unsigned char const*, unsigned int, unsigned int, Exiv2::ByteOrder) /home/woot/Desktop/exiv2/src/crwimage_int.cpp:227
    #76 0x7fce8f79ca54 in Exiv2::Internal::CiffDirectory::readDirectory(unsigned char const*, unsigned int, Exiv2::ByteOrder) /home/woot/Desktop/exiv2/src/crwimage_int.cpp:296
    #77 0x7fce8f79c733 in Exiv2::Internal::CiffDirectory::doRead(unsigned char const*, unsigned int, unsigned int, Exiv2::ByteOrder) /home/woot/Desktop/exiv2/src/crwimage_int.cpp:269
    #78 0x7fce8f79c318 in Exiv2::Internal::CiffComponent::read(unsigned char const*, unsigned int, unsigned int, Exiv2::ByteOrder) /home/woot/Desktop/exiv2/src/crwimage_int.cpp:227
    #79 0x7fce8f79ca54 in Exiv2::Internal::CiffDirectory::readDirectory(unsigned char const*, unsigned int, Exiv2::ByteOrder) /home/woot/Desktop/exiv2/src/crwimage_int.cpp:296
    #80 0x7fce8f79c733 in Exiv2::Internal::CiffDirectory::doRead(unsigned char const*, unsigned int, unsigned int, Exiv2::ByteOrder) /home/woot/Desktop/exiv2/src/crwimage_int.cpp:269
    #81 0x7fce8f79c318 in Exiv2::Internal::CiffComponent::read(unsigned char const*, unsigned int, unsigned int, Exiv2::ByteOrder) /home/woot/Desktop/exiv2/src/crwimage_int.cpp:227
    #82 0x7fce8f79ca54 in Exiv2::Internal::CiffDirectory::readDirectory(unsigned char const*, unsigned int, Exiv2::ByteOrder) /home/woot/Desktop/exiv2/src/crwimage_int.cpp:296
    #83 0x7fce8f79c733 in Exiv2::Internal::CiffDirectory::doRead(unsigned char const*, unsigned int, unsigned int, Exiv2::ByteOrder) /home/woot/Desktop/exiv2/src/crwimage_int.cpp:269
    #84 0x7fce8f79c318 in Exiv2::Internal::CiffComponent::read(unsigned char const*, unsigned int, unsigned int, Exiv2::ByteOrder) /home/woot/Desktop/exiv2/src/crwimage_int.cpp:227
    #85 0x7fce8f79ca54 in Exiv2::Internal::CiffDirectory::readDirectory(unsigned char const*, unsigned int, Exiv2::ByteOrder) /home/woot/Desktop/exiv2/src/crwimage_int.cpp:296
    #86 0x7fce8f79c733 in Exiv2::Internal::CiffDirectory::doRead(unsigned char const*, unsigned int, unsigned int, Exiv2::ByteOrder) /home/woot/Desktop/exiv2/src/crwimage_int.cpp:269
    #87 0x7fce8f79c318 in Exiv2::Internal::CiffComponent::read(unsigned char const*, unsigned int, unsigned int, Exiv2::ByteOrder) /home/woot/Desktop/exiv2/src/crwimage_int.cpp:227
    #88 0x7fce8f79ca54 in Exiv2::Internal::CiffDirectory::readDirectory(unsigned char const*, unsigned int, Exiv2::ByteOrder) /home/woot/Desktop/exiv2/src/crwimage_int.cpp:296
    #89 0x7fce8f79c733 in Exiv2::Internal::CiffDirectory::doRead(unsigned char const*, unsigned int, unsigned int, Exiv2::ByteOrder) /home/woot/Desktop/exiv2/src/crwimage_int.cpp:269
    #90 0x7fce8f79c318 in Exiv2::Internal::CiffComponent::read(unsigned char const*, unsigned int, unsigned int, Exiv2::ByteOrder) /home/woot/Desktop/exiv2/src/crwimage_int.cpp:227
    #91 0x7fce8f79ca54 in Exiv2::Internal::CiffDirectory::readDirectory(unsigned char const*, unsigned int, Exiv2::ByteOrder) /home/woot/Desktop/exiv2/src/crwimage_int.cpp:296
    #92 0x7fce8f79c733 in Exiv2::Internal::CiffDirectory::doRead(unsigned char const*, unsigned int, unsigned int, Exiv2::ByteOrder) /home/woot/Desktop/exiv2/src/crwimage_int.cpp:269
    #93 0x7fce8f79c318 in Exiv2::Internal::CiffComponent::read(unsigned char const*, unsigned int, unsigned int, Exiv2::ByteOrder) /home/woot/Desktop/exiv2/src/crwimage_int.cpp:227
    #94 0x7fce8f79ca54 in Exiv2::Internal::CiffDirectory::readDirectory(unsigned char const*, unsigned int, Exiv2::ByteOrder) /home/woot/Desktop/exiv2/src/crwimage_int.cpp:296
    #95 0x7fce8f79c733 in Exiv2::Internal::CiffDirectory::doRead(unsigned char const*, unsigned int, unsigned int, Exiv2::ByteOrder) /home/woot/Desktop/exiv2/src/crwimage_int.cpp:269
    #96 0x7fce8f79c318 in Exiv2::Internal::CiffComponent::read(unsigned char const*, unsigned int, unsigned int, Exiv2::ByteOrder) /home/woot/Desktop/exiv2/src/crwimage_int.cpp:227
    #97 0x7fce8f79ca54 in Exiv2::Internal::CiffDirectory::readDirectory(unsigned char const*, unsigned int, Exiv2::ByteOrder) /home/woot/Desktop/exiv2/src/crwimage_int.cpp:296
    #98 0x7fce8f79c733 in Exiv2::Internal::CiffDirectory::doRead(unsigned char const*, unsigned int, unsigned int, Exiv2::ByteOrder) /home/woot/Desktop/exiv2/src/crwimage_int.cpp:269
    #99 0x7fce8f79c318 in Exiv2::Internal::CiffComponent::read(unsigned char const*, unsigned int, unsigned int, Exiv2::ByteOrder) /home/woot/Desktop/exiv2/src/crwimage_int.cpp:227
    #100 0x7fce8f79ca54 in Exiv2::Internal::CiffDirectory::readDirectory(unsigned char const*, unsigned int, Exiv2::ByteOrder) /home/woot/Desktop/exiv2/src/crwimage_int.cpp:296
    #101 0x7fce8f79c733 in Exiv2::Internal::CiffDirectory::doRead(unsigned char const*, unsigned int, unsigned int, Exiv2::ByteOrder) /home/woot/Desktop/exiv2/src/crwimage_int.cpp:269
    #102 0x7fce8f79c318 in Exiv2::Internal::CiffComponent::read(unsigned char const*, unsigned int, unsigned int, Exiv2::ByteOrder) /home/woot/Desktop/exiv2/src/crwimage_int.cpp:227
    #103 0x7fce8f79ca54 in Exiv2::Internal::CiffDirectory::readDirectory(unsigned char const*, unsigned int, Exiv2::ByteOrder) /home/woot/Desktop/exiv2/src/crwimage_int.cpp:296
    #104 0x7fce8f79c733 in Exiv2::Internal::CiffDirectory::doRead(unsigned char const*, unsigned int, unsigned int, Exiv2::ByteOrder) /home/woot/Desktop/exiv2/src/crwimage_int.cpp:269
.
.
.
.
.
.
.
.
.    #240 0x7fce8f79c318 in Exiv2::Internal::CiffComponent::read(unsigned char const*, unsigned int, unsigned int, Exiv2::ByteOrder) /home/woot/Desktop/exiv2/src/crwimage_int.cpp:227
    #241 0x7fce8f79ca54 in Exiv2::Internal::CiffDirectory::readDirectory(unsigned char const*, unsigned int, Exiv2::ByteOrder) /home/woot/Desktop/exiv2/src/crwimage_int.cpp:296
    #242 0x7fce8f79c733 in Exiv2::Internal::CiffDirectory::doRead(unsigned char const*, unsigned int, unsigned int, Exiv2::ByteOrder) /home/woot/Desktop/exiv2/src/crwimage_int.cpp:269
    #243 0x7fce8f79c318 in Exiv2::Internal::CiffComponent::read(unsigned char const*, unsigned int, unsigned int, Exiv2::ByteOrder) /home/woot/Desktop/exiv2/src/crwimage_int.cpp:227
    #244 0x7fce8f79ca54 in Exiv2::Internal::CiffDirectory::readDirectory(unsigned char const*, unsigned int, Exiv2::ByteOrder) /home/woot/Desktop/exiv2/src/crwimage_int.cpp:296
    #245 0x7fce8f79c733 in Exiv2::Internal::CiffDirectory::doRead(unsigned char const*, unsigned int, unsigned int, Exiv2::ByteOrder) /home/woot/Desktop/exiv2/src/crwimage_int.cpp:269
    #246 0x7fce8f79c318 in Exiv2::Internal::CiffComponent::read(unsigned char const*, unsigned int, unsigned int, Exiv2::ByteOrder) /home/woot/Desktop/exiv2/src/crwimage_int.cpp:227
    #247 0x7fce8f79ca54 in Exiv2::Internal::CiffDirectory::readDirectory(unsigned char const*, unsigned int, Exiv2::ByteOrder) /home/woot/Desktop/exiv2/src/crwimage_int.cpp:296
    #248 0x7fce8f79c733 in Exiv2::Internal::CiffDirectory::doRead(unsigned char const*, unsigned int, unsigned int, Exiv2::ByteOrder) /home/woot/Desktop/exiv2/src/crwimage_int.cpp:269
    #249 0x7fce8f79c318 in Exiv2::Internal::CiffComponent::read(unsigned char const*, unsigned int, unsigned int, Exiv2::ByteOrder) /home/woot/Desktop/exiv2/src/crwimage_int.cpp:227
    #250 0x7fce8f79ca54 in Exiv2::Internal::CiffDirectory::readDirectory(unsigned char const*, unsigned int, Exiv2::ByteOrder) /home/woot/Desktop/exiv2/src/crwimage_int.cpp:296
    #251 0x7fce8f79c733 in Exiv2::Internal::CiffDirectory::doRead(unsigned char const*, unsigned int, unsigned int, Exiv2::ByteOrder) /home/woot/Desktop/exiv2/src/crwimage_int.cpp:269

SUMMARY: AddressSanitizer: stack-overflow ??:0 ??
```


[Reproducer file](https://github.com/SegfaultMasters/covering360/blob/master/Exiv2/readDirectory_stackoverflow_16)
