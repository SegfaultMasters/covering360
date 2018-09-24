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
