##### Description

HDF5 is a data model, library, and file format for storing and managing data. It supports an unlimited variety of datatypes, and is designed for flexible and efficient I/O and for high volume and complex data. HDF5 is portable and is extensible, allowing applications to evolve in their use of HDF5. The HDF5 Technology suite includes tools and applications for managing, manipulating, viewing, and analyzing data in the HDF5 format. link: https://portal.hdfgroup.org/display/HDF5/HDF5

## Heap Overflow in ReadGifImageDesc

A heap-buffer-overflow was discovered in ReadGifImageDesc() in gifread.c in HDF5 through 1.10.3 allows attackers to cause a denial of service via a crafted HDF5 file. This issue was triggered while converting GIF file to HDF file. 

#### Affected version - 1.10.3 and before (compiled from source)

##### Command: ./gif2h5 $POC ~/output/ex_image2.h5 

###### ASAN Report

```
./gif2h5 ~/output_gif2h5/crashes/POC013 ~/output/ex_image2.h5 
=================================================================
==2834==ERROR: AddressSanitizer: heap-buffer-overflow on address 0x61f000000d00 at pc 0x55d43756998b bp 0x7ffff747a670 sp 0x7ffff747a660
WRITE of size 1 at 0x61f000000d00 thread T0
    #0 0x55d43756998a in ReadGifImageDesc /home/ethan/hdf5-develop/hl/tools/gif2h5/gifread.c:201
    #1 0x55d437566d19 in Gif2Mem /home/ethan/hdf5-develop/hl/tools/gif2h5/gif2mem.c:180
    #2 0x55d437565da1 in main /home/ethan/hdf5-develop/hl/tools/gif2h5/gif2hdf.c:100
    #3 0x7efdcd4181c0 in __libc_start_main (/lib/x86_64-linux-gnu/libc.so.6+0x211c0)
    #4 0x55d4375657b9 in _start (/home/ethan/hdf5-develop/hdf5/bin/gif2h5+0x127b9)

0x61f000000d00 is located 0 bytes to the right of 3200-byte region [0x61f000000080,0x61f000000d00)
allocated by thread T0 here:
    #0 0x7efdcf3f5b50 in __interceptor_malloc (/usr/lib/x86_64-linux-gnu/libasan.so.4+0xdeb50)
    #1 0x55d4375697c5 in ReadGifImageDesc /home/ethan/hdf5-develop/hl/tools/gif2h5/gifread.c:191
    #2 0x55d437566d19 in Gif2Mem /home/ethan/hdf5-develop/hl/tools/gif2h5/gif2mem.c:180
    #3 0x55d437565da1 in main /home/ethan/hdf5-develop/hl/tools/gif2h5/gif2hdf.c:100
    #4 0x7efdcd4181c0 in __libc_start_main (/lib/x86_64-linux-gnu/libc.so.6+0x211c0)

SUMMARY: AddressSanitizer: heap-buffer-overflow /home/ethan/hdf5-develop/hl/tools/gif2h5/gifread.c:201 in ReadGifImageDesc
Shadow bytes around the buggy address:
  0x0c3e7fff8150: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x0c3e7fff8160: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x0c3e7fff8170: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x0c3e7fff8180: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x0c3e7fff8190: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
=>0x0c3e7fff81a0:[fa]fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x0c3e7fff81b0: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x0c3e7fff81c0: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x0c3e7fff81d0: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x0c3e7fff81e0: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x0c3e7fff81f0: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
Shadow byte legend (one shadow byte represents 8 application bytes):
  Addressable:           00
  Partially addressable: 01 02 03 04 05 06 07 
  Heap left redzone:       fa
  Freed heap region:       fd
  Stack left redzone:      f1
  Stack mid redzone:       f2
  Stack right redzone:     f3
  Stack after return:      f5
  Stack use after scope:   f8
  Global redzone:          f9
  Global init order:       f6
  Poisoned by user:        f7
  Container overflow:      fc
  Array cookie:            ac
  Intra object redzone:    bb
  ASan internal:           fe
  Left alloca redzone:     ca
  Right alloca redzone:    cb
==2834==ABORTING

```
=============================================================

