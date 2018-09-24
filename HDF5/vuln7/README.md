##### Description

HDF5 is a data model, library, and file format for storing and managing data. It supports an unlimited variety of datatypes, and is designed for flexible and efficient I/O and for high volume and complex data. HDF5 is portable and is extensible, allowing applications to evolve in their use of HDF5. The HDF5 Technology suite includes tools and applications for managing, manipulating, viewing, and analyzing data in the HDF5 format. link: https://portal.hdfgroup.org/display/HDF5/HDF5

## Heap Overflow in H5O_attr_decode

A heap-buffer-overflow was discovered in H5O_attr_decode() in H5Oattr.c in HDF5 through 1.10.3 allows attackers to cause a denial of service via a crafted HDF5 file. This issue was triggered while converting HDF file to GIF file. 

#### Affected version - 1.10.3 and before (compiled from source)

##### Command: ./h52gif $POC ~/output_h5gif/image1.gif -i image

###### Debugging: 

```
Source: 

    234	     /* Go get the data */
    235	     if(attr->shared->data_size) {
    236	         if(NULL == (attr->shared->data = H5FL_BLK_MALLOC(attr_buf, attr->shared->data_size)))
    237	             HGOTO_ERROR(H5E_RESOURCE, H5E_NOSPACE, NULL, "memory allocation failed")
		// p=0x00007fffffffd040  →  [...]  →  0x0000004547414d49 ("IMAGE"?), attr=0x00007fffffffd080  →  [...]  →  0x0000000000000000
 →  238	         HDmemcpy(attr->shared->data, p, attr->shared->data_size);
    239	     } /* end if */
    240	 

gef➤  x/d attr->shared->data
0x5555557d03e8:	0

gef➤ p attr->shared->data_size
$1 = 0x6


Backtrace

[#0] 0x7ffff76e9f80 → Name: H5O_attr_decode(f=0x5555557c68d0, open_oh=0x5555557cbe60, mesg_flags=0x0, ioflags=0x7fffffffd158, p_size=0x28, p=0x5555557cc130 "IMAGE")
[#1] 0x7ffff76e8aee → Name: H5O_attr_shared_decode(f=0x5555557c68d0, open_oh=0x5555557cbe60, mesg_flags=0x0, ioflags=0x7fffffffd158, p_size=0x28, p=0x5555557cc110 "\001")
[#2] 0x7ffff772381e → Name: H5O__msg_iterate_real(f=0x5555557c68d0, oh=0x5555557cbe60, type=0x7ffff7b9cba0 <H5O_MSG_ATTR>, op=0x7fffffffd1c0, op_data=0x7fffffffd1d0)
[#3] 0x7ffff7528cce → Name: H5A__compact_build_table(f=0x5555557c68d0, oh=0x5555557cbe60, idx_type=H5_INDEX_NAME, order=H5_ITER_INC, atable=0x7fffffffd270)
[#4] 0x7ffff76eecea → Name: H5O_attr_iterate_real(loc_id=0x500000000000000, loc=0x5555557ccc80, idx_type=H5_INDEX_NAME, order=H5_ITER_INC, skip=0x0, last_attr=0x7fffffffd388, attr_op=0x7fffffffd3f0, op_data=0x7ffff7bcd0fd)
[#5] 0x7ffff76ef103 → Name: H5O__attr_iterate(loc_id=0x500000000000000, idx_type=H5_INDEX_NAME, order=H5_ITER_INC, skip=0x0, last_attr=0x7fffffffd388, attr_op=0x7fffffffd3f0, op_data=0x7ffff7bcd0fd)
[#6] 0x7ffff752c543 → Name: H5A__iterate_common(loc_id=0x500000000000000, idx_type=H5_INDEX_NAME, order=H5_ITER_INC, idx=0x0, attr_op=0x7fffffffd3f0, op_data=0x7ffff7bcd0fd)
[#7] 0x7ffff752c65c → Name: H5A__iterate(loc_id=0x500000000000000, idx_type=H5_INDEX_NAME, order=H5_ITER_INC, idx=0x0, op=0x7ffff7bbbce3 <find_attr>, op_data=0x7ffff7bcd0fd)
[#8] 0x7ffff751a187 → Name: H5Aiterate2(loc_id=0x500000000000000, idx_type=H5_INDEX_NAME, order=H5_ITER_INC, idx=0x0, op=0x7ffff7bbbce3 <find_attr>, op_data=0x7ffff7bcd0fd)
[#9] 0x7ffff7bbbdd7 → Name: H5LT_find_attribute(loc_id=0x500000000000000, attr_name=0x7ffff7bcd0fd "INTERLACE_MODE")


```

###### ASAN Report

```
./h52gif  ~/output_h5gif/POC012 ~/output_h5gif/image1.gif -i image
=================================================================
==2702==ERROR: AddressSanitizer: heap-buffer-overflow on address 0x613000027950 at pc 0x7ff83ae0e733 bp 0x7fff44c837c0 sp 0x7fff44c82f68
READ of size 13154500 at 0x613000027950 thread T0
    #0 0x7ff83ae0e732  (/usr/lib/x86_64-linux-gnu/libasan.so.4+0x79732)
    #1 0x7ff839d350a0 in H5O_attr_decode /home/ethan/hdf5-develop/src/H5Oattr.c:238
    #2 0x7ff839d30738 in H5O_attr_shared_decode /home/ethan/hdf5-develop/src/H5Oshared.h:82
    #3 0x7ff839e0e6ea in H5O__msg_iterate_real /home/ethan/hdf5-develop/src/H5Omessage.c:1288
    #4 0x7ff8395e7d5b in H5A__compact_build_table /home/ethan/hdf5-develop/src/H5Aint.c:1466
    #5 0x7ff839d48078 in H5O_attr_iterate_real /home/ethan/hdf5-develop/src/H5Oattribute.c:1283
    #6 0x7ff839d48f9c in H5O__attr_iterate /home/ethan/hdf5-develop/src/H5Oattribute.c:1341
    #7 0x7ff8395f6e2b in H5A__iterate_common /home/ethan/hdf5-develop/src/H5Aint.c:2596
    #8 0x7ff8395f76d5 in H5A__iterate /home/ethan/hdf5-develop/src/H5Aint.c:2634
    #9 0x7ff8395afcde in H5Aiterate2 /home/ethan/hdf5-develop/src/H5A.c:1258
    #10 0x7ff83ab68905 in H5LT_find_attribute /home/ethan/hdf5-develop/hl/src/H5LT.c:2040
    #11 0x7ff83ab6236b in H5IMget_image_info /home/ethan/hdf5-develop/hl/src/H5IM.c:277
    #12 0x558f13308979 in main /home/ethan/hdf5-develop/hl/tools/gif2h5/hdf2gif.c:153
    #13 0x7ff838e961c0 in __libc_start_main (/lib/x86_64-linux-gnu/libc.so.6+0x211c0)
    #14 0x558f13308159 in _start (/home/ethan/hdf5-develop/hdf5/bin/h52gif+0x12159)

0x613000027950 is located 0 bytes to the right of 336-byte region [0x613000027800,0x613000027950)
allocated by thread T0 here:
    #0 0x7ff83ae73b50 in __interceptor_malloc (/usr/lib/x86_64-linux-gnu/libasan.so.4+0xdeb50)
    #1 0x7ff839cf6dd0 in H5MM_malloc /home/ethan/hdf5-develop/src/H5MM.c:292
    #2 0x7ff839a8e9b4 in H5FL_malloc /home/ethan/hdf5-develop/src/H5FL.c:243
    #3 0x7ff839a92945 in H5FL_blk_malloc /home/ethan/hdf5-develop/src/H5FL.c:921
    #4 0x7ff839d6271e in H5O__chunk_deserialize /home/ethan/hdf5-develop/src/H5Ocache.c:1350
    #5 0x7ff839d5527a in H5O__cache_deserialize /home/ethan/hdf5-develop/src/H5Ocache.c:355
    #6 0x7ff839703424 in H5C_load_entry /home/ethan/hdf5-develop/src/H5C.c:6722
    #7 0x7ff8396cc3fd in H5C_protect /home/ethan/hdf5-develop/src/H5C.c:2357
    #8 0x7ff839606665 in H5AC_protect /home/ethan/hdf5-develop/src/H5AC.c:1624
    #9 0x7ff839dd2c88 in H5O_protect /home/ethan/hdf5-develop/src/H5Oint.c:984
    #10 0x7ff839dd8264 in H5O_obj_type /home/ethan/hdf5-develop/src/H5Oint.c:1591
    #11 0x7ff839867bf4 in H5D__open_name /home/ethan/hdf5-develop/src/H5Dint.c:1167
    #12 0x7ff8397866d5 in H5Dopen2 /home/ethan/hdf5-develop/src/H5D.c:291
    #13 0x7ff83ab6233a in H5IMget_image_info /home/ethan/hdf5-develop/hl/src/H5IM.c:273
    #14 0x558f13308979 in main /home/ethan/hdf5-develop/hl/tools/gif2h5/hdf2gif.c:153
    #15 0x7ff838e961c0 in __libc_start_main (/lib/x86_64-linux-gnu/libc.so.6+0x211c0)

SUMMARY: AddressSanitizer: heap-buffer-overflow (/usr/lib/x86_64-linux-gnu/libasan.so.4+0x79732) 
Shadow bytes around the buggy address:
  0x0c267fffced0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x0c267fffcee0: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x0c267fffcef0: 00 fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x0c267fffcf00: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x0c267fffcf10: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
=>0x0c267fffcf20: 00 00 00 00 00 00 00 00 00 00[fa]fa fa fa fa fa
  0x0c267fffcf30: fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa fa
  0x0c267fffcf40: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x0c267fffcf50: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x0c267fffcf60: 00 00 00 00 00 fa fa fa fa fa fa fa fa fa fa fa
  0x0c267fffcf70: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
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
==2702==ABORTING

```
=============================================================

