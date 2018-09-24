##### Description

HDF5 is a data model, library, and file format for storing and managing data. It supports an unlimited variety of datatypes, and is designed for flexible and efficient I/O and for high volume and complex data. HDF5 is portable and is extensible, allowing applications to evolve in their use of HDF5. The HDF5 Technology suite includes tools and applications for managing, manipulating, viewing, and analyzing data in the HDF5 format. link: https://portal.hdfgroup.org/display/HDF5/HDF5

## Memory leak in H5O_dtype_decode_helper

Memory leak in the H5O_dtype_decode_helper() function in H5Odtype.c in HDF HDF5 through 1.10.3 allows attackers to cause a denial of service (memory consumption) via a crafted HDF5 file.

#### Affected version - 1.10.3 and before (compiled from source)

#### Command: ./h52gif $POC ~/output_h5gif/image1.gif -i image

##### Debugging: valgrind --leak-check=full ./h52gif $POC ~/output_h5gif/image1.gif -i image

###### Backtrace

```
valgrind --leak-check=full ./h52gif ~/output_h5gif/crashes/POC_H5O_dtype_decode_helper_H5Odtype ~/output_h5gif/image1.gif -i image
==2222== Memcheck, a memory error detector
==2222== Copyright (C) 2002-2017, and GNU GPL'd, by Julian Seward et al.
==2222== Using Valgrind-3.13.0 and LibVEX; rerun with -h for copyright info
==2222== Command: ./h52gif /home/ethan/output_h5gif/crashes/POC019 /home/ethan/output_h5gif/image1.gif -i image
==2222== 
==2222== Invalid read of size 1
==2222==    at 0x52A5316: H5O_dtype_decode_helper (H5Odtype.c:146)
==2222==    by 0x52A9620: H5O_dtype_decode (H5Odtype.c:1110)
==2222==    by 0x52A48B2: H5O_dtype_shared_decode (H5Oshared.h:82)
==2222==    by 0x528DA5A: H5O_attr_decode (H5Oattr.c:186)
==2222==    by 0x528CAED: H5O_attr_shared_decode (H5Oshared.h:82)
==2222==    by 0x52C781D: H5O__msg_iterate_real (H5Omessage.c:1288)
==2222==    by 0x50CCCCD: H5A__compact_build_table (H5Aint.c:1544)
==2222==    by 0x5292CE9: H5O_attr_iterate_real (H5Oattribute.c:1283)
==2222==    by 0x5293102: H5O__attr_iterate (H5Oattribute.c:1341)
==2222==    by 0x50D0542: H5A__iterate_common (H5Aint.c:2742)
==2222==    by 0x50D065B: H5A__iterate (H5Aint.c:2784)
==2222==    by 0x50BE186: H5Aiterate2 (H5A.c:1258)
==2222==  Address 0x63960f8 is 8 bytes after a block of size 80 alloc'd
==2222==    at 0x4C2FB0F: malloc (in /usr/lib/valgrind/vgpreload_memcheck-amd64-linux.so)
==2222==    by 0x527B5A0: H5MM_malloc (H5MM.c:322)
==2222==    by 0x51E6378: H5FL_malloc (H5FL.c:243)
==2222==    by 0x51E71AF: H5FL_blk_malloc (H5FL.c:921)
==2222==    by 0x5298713: H5O__chunk_deserialize (H5Ocache.c:1350)
==2222==    by 0x5296CF5: H5O__cache_chk_deserialize (H5Ocache.c:793)
==2222==    by 0x510FF4E: H5C_load_entry (H5C.c:6725)
==2222==    by 0x51056FC: H5C_protect (H5C.c:2357)
==2222==    by 0x50D42E9: H5AC_protect (H5AC.c:1624)
==2222==    by 0x52B77E8: H5O_protect (H5Oint.c:1137)
==2222==    by 0x52B90A1: H5O_obj_type (H5Oint.c:1706)
==2222==    by 0x5152B18: H5D__open_name (H5Dint.c:1209)
==2222== 
==2222== Invalid read of size 1
==2222==    at 0x52A5344: H5O_dtype_decode_helper (H5Odtype.c:146)
==2222==    by 0x52A9620: H5O_dtype_decode (H5Odtype.c:1110)
==2222==    by 0x52A48B2: H5O_dtype_shared_decode (H5Oshared.h:82)
==2222==    by 0x528DA5A: H5O_attr_decode (H5Oattr.c:186)
==2222==    by 0x528CAED: H5O_attr_shared_decode (H5Oshared.h:82)
==2222==    by 0x52C781D: H5O__msg_iterate_real (H5Omessage.c:1288)
==2222==    by 0x50CCCCD: H5A__compact_build_table (H5Aint.c:1544)
==2222==    by 0x5292CE9: H5O_attr_iterate_real (H5Oattribute.c:1283)
==2222==    by 0x5293102: H5O__attr_iterate (H5Oattribute.c:1341)
==2222==    by 0x50D0542: H5A__iterate_common (H5Aint.c:2742)
==2222==    by 0x50D065B: H5A__iterate (H5Aint.c:2784)
==2222==    by 0x50BE186: H5Aiterate2 (H5A.c:1258)
==2222==  Address 0x63960f9 is 9 bytes after a block of size 80 alloc'd
==2222==    at 0x4C2FB0F: malloc (in /usr/lib/valgrind/vgpreload_memcheck-amd64-linux.so)
==2222==    by 0x527B5A0: H5MM_malloc (H5MM.c:322)
==2222==    by 0x51E6378: H5FL_malloc (H5FL.c:243)
==2222==    by 0x51E71AF: H5FL_blk_malloc (H5FL.c:921)
==2222==    by 0x5298713: H5O__chunk_deserialize (H5Ocache.c:1350)
==2222==    by 0x5296CF5: H5O__cache_chk_deserialize (H5Ocache.c:793)
==2222==    by 0x510FF4E: H5C_load_entry (H5C.c:6725)
==2222==    by 0x51056FC: H5C_protect (H5C.c:2357)
==2222==    by 0x50D42E9: H5AC_protect (H5AC.c:1624)
==2222==    by 0x52B77E8: H5O_protect (H5Oint.c:1137)
==2222==    by 0x52B90A1: H5O_obj_type (H5Oint.c:1706)
==2222==    by 0x5152B18: H5D__open_name (H5Dint.c:1209)
==2222== 
==2222== Invalid read of size 1
==2222==    at 0x52A5375: H5O_dtype_decode_helper (H5Odtype.c:146)
==2222==    by 0x52A9620: H5O_dtype_decode (H5Odtype.c:1110)
==2222==    by 0x52A48B2: H5O_dtype_shared_decode (H5Oshared.h:82)
==2222==    by 0x528DA5A: H5O_attr_decode (H5Oattr.c:186)
==2222==    by 0x528CAED: H5O_attr_shared_decode (H5Oshared.h:82)
==2222==    by 0x52C781D: H5O__msg_iterate_real (H5Omessage.c:1288)
==2222==    by 0x50CCCCD: H5A__compact_build_table (H5Aint.c:1544)
==2222==    by 0x5292CE9: H5O_attr_iterate_real (H5Oattribute.c:1283)
==2222==    by 0x5293102: H5O__attr_iterate (H5Oattribute.c:1341)
==2222==    by 0x50D0542: H5A__iterate_common (H5Aint.c:2742)
==2222==    by 0x50D065B: H5A__iterate (H5Aint.c:2784)
==2222==    by 0x50BE186: H5Aiterate2 (H5A.c:1258)
==2222==  Address 0x63960fa is 10 bytes after a block of size 80 alloc'd
==2222==    at 0x4C2FB0F: malloc (in /usr/lib/valgrind/vgpreload_memcheck-amd64-linux.so)
==2222==    by 0x527B5A0: H5MM_malloc (H5MM.c:322)
==2222==    by 0x51E6378: H5FL_malloc (H5FL.c:243)
==2222==    by 0x51E71AF: H5FL_blk_malloc (H5FL.c:921)
==2222==    by 0x5298713: H5O__chunk_deserialize (H5Ocache.c:1350)
==2222==    by 0x5296CF5: H5O__cache_chk_deserialize (H5Ocache.c:793)
==2222==    by 0x510FF4E: H5C_load_entry (H5C.c:6725)
==2222==    by 0x51056FC: H5C_protect (H5C.c:2357)
==2222==    by 0x50D42E9: H5AC_protect (H5AC.c:1624)
==2222==    by 0x52B77E8: H5O_protect (H5Oint.c:1137)
==2222==    by 0x52B90A1: H5O_obj_type (H5Oint.c:1706)
==2222==    by 0x5152B18: H5D__open_name (H5Dint.c:1209)
==2222== 
==2222== Invalid read of size 1
==2222==    at 0x52A53A6: H5O_dtype_decode_helper (H5Odtype.c:146)
==2222==    by 0x52A9620: H5O_dtype_decode (H5Odtype.c:1110)
==2222==    by 0x52A48B2: H5O_dtype_shared_decode (H5Oshared.h:82)
==2222==    by 0x528DA5A: H5O_attr_decode (H5Oattr.c:186)
==2222==    by 0x528CAED: H5O_attr_shared_decode (H5Oshared.h:82)
==2222==    by 0x52C781D: H5O__msg_iterate_real (H5Omessage.c:1288)
==2222==    by 0x50CCCCD: H5A__compact_build_table (H5Aint.c:1544)
==2222==    by 0x5292CE9: H5O_attr_iterate_real (H5Oattribute.c:1283)
==2222==    by 0x5293102: H5O__attr_iterate (H5Oattribute.c:1341)
==2222==    by 0x50D0542: H5A__iterate_common (H5Aint.c:2742)
==2222==    by 0x50D065B: H5A__iterate (H5Aint.c:2784)
==2222==    by 0x50BE186: H5Aiterate2 (H5A.c:1258)
==2222==  Address 0x63960fb is 11 bytes after a block of size 80 alloc'd
==2222==    at 0x4C2FB0F: malloc (in /usr/lib/valgrind/vgpreload_memcheck-amd64-linux.so)
==2222==    by 0x527B5A0: H5MM_malloc (H5MM.c:322)
==2222==    by 0x51E6378: H5FL_malloc (H5FL.c:243)
==2222==    by 0x51E71AF: H5FL_blk_malloc (H5FL.c:921)
==2222==    by 0x5298713: H5O__chunk_deserialize (H5Ocache.c:1350)
==2222==    by 0x5296CF5: H5O__cache_chk_deserialize (H5Ocache.c:793)
==2222==    by 0x510FF4E: H5C_load_entry (H5C.c:6725)
==2222==    by 0x51056FC: H5C_protect (H5C.c:2357)
==2222==    by 0x50D42E9: H5AC_protect (H5AC.c:1624)
==2222==    by 0x52B77E8: H5O_protect (H5Oint.c:1137)
==2222==    by 0x52B90A1: H5O_obj_type (H5Oint.c:1706)
==2222==    by 0x5152B18: H5D__open_name (H5Dint.c:1209)
==2222== 
==2222== 
==2222== HEAP SUMMARY:
==2222==     in use at exit: 0 bytes in 0 blocks
==2222==   total heap usage: 3,226 allocs, 3,226 frees, 929,901 bytes allocated
==2222== 
==2222== All heap blocks were freed -- no leaks are possible
==2222== 
==2222== For counts of detected and suppressed errors, rerun with: -v
==2222== ERROR SUMMARY: 8 errors from 4 contexts (suppressed: 0 from 0)

```

===============================================================================

## Stack overflow in H5S_extent_get_dims

An issue was discovered in the HDF HDF5 1.10.3 library. There is a stack-based buffer overflow in the function H5S_extent_get_dims() in H5S.c. Specifically, this issue occur while converting an HDF5 file to a GIF file. 

#### Affected version - 1.10.3 and before (compiled from source)

### Command: ./h52gif  $POC ~/output_h5gif/image1.gif -i image


```
 Source: 

  1054	             ret_value = (int)ext->rank;
   1055	             for(i = 0; i < ret_value; i++) {
   1056	                 if(dims)
		// ext=0x00007fffffffd3e8  →  [...]  →  0x0000000000000000, dims=0x00007fffffffd3e0  →  [...]  →  0x00000000000000c8, i=0x12
 → 1057	                     dims[i] = ext->size[i];
   1058	                 if(max_dims) {
   1059	                     if(ext->max)
   1060	                         max_dims[i] = ext->max[i];
```

### ASAN Report

```

./h52gif  ~/output_h5gif/crashes/POC005 ~/output_h5gif/image1.gif -i image
=================================================================
==2267==ERROR: AddressSanitizer: stack-buffer-overflow on address 0x7fffa796ba38 at pc 0x7febcd0e4a2d bp 0x7fffa796b820 sp 0x7fffa796b810
WRITE of size 8 at 0x7fffa796ba38 thread T0
    #0 0x7febcd0e4a2c in H5S_extent_get_dims /home/ethan/hdf5-develop/src/H5S.c:1057
    #1 0x7febcd0e4f39 in H5S_get_simple_extent_dims /home/ethan/hdf5-develop/src/H5S.c:1106
    #2 0x7febcd0e44cc in H5Sget_simple_extent_dims /home/ethan/hdf5-develop/src/H5S.c:1015
    #3 0x7febcdc95467 in H5IMget_image_info /home/ethan/hdf5-develop/hl/src/H5IM.c:304
    #4 0x55b76d7fd979 in main /home/ethan/hdf5-develop/hl/tools/gif2h5/hdf2gif.c:153
    #5 0x7febcbfc91c0 in __libc_start_main (/lib/x86_64-linux-gnu/libc.so.6+0x211c0)
    #6 0x55b76d7fd159 in _start (/home/ethan/hdf5-develop/hdf5/bin/h52gif+0x12159)

Address 0x7fffa796ba38 is located in stack of thread T0 at offset 56 in frame
    #0 0x7febcdc951db in H5IMget_image_info /home/ethan/hdf5-develop/hl/src/H5IM.c:252

  This frame has 1 object(s):
    [32, 56) 'dims' <== Memory access at offset 56 overflows this variable
HINT: this may be a false positive if your program uses some custom stack unwind mechanism or swapcontext
      (longjmp and C++ exceptions *are* supported)
SUMMARY: AddressSanitizer: stack-buffer-overflow /home/ethan/hdf5-develop/src/H5S.c:1057 in H5S_extent_get_dims
Shadow bytes around the buggy address:
  0x100074f256f0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x100074f25700: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x100074f25710: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 f1 f1
  0x100074f25720: f1 f1 00 f2 f2 f2 00 00 00 00 00 00 00 00 00 00
  0x100074f25730: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
=>0x100074f25740: f1 f1 f1 f1 00 00 00[f2]00 00 00 00 00 00 00 00
  0x100074f25750: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
  0x100074f25760: 00 00 00 00 f1 f1 f1 f1 04 f2 f2 f2 f2 f2 f2 f2
  0x100074f25770: 00 f2 f2 f2 f2 f2 f2 f2 00 f2 f2 f2 f2 f2 f2 f2
  0x100074f25780: 00 f2 f2 f2 f2 f2 f2 f2 00 f2 f2 f2 f2 f2 f2 f2
  0x100074f25790: 00 f2 f2 f2 f2 f2 f2 f2 00 f2 f2 f2 f2 f2 f2 f2
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
==2267==ABORTING

```

===================================================================================================================
