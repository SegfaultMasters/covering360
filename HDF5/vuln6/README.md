##### Description

HDF5 is a data model, library, and file format for storing and managing data. It supports an unlimited variety of datatypes, and is designed for flexible and efficient I/O and for high volume and complex data. HDF5 is portable and is extensible, allowing applications to evolve in their use of HDF5. The HDF5 Technology suite includes tools and applications for managing, manipulating, viewing, and analyzing data in the HDF5 format. link: https://portal.hdfgroup.org/display/HDF5/HDF5

## Null pointer dereference in H5O_sdspace_encode (CVE-2018-17432)

A NULL pointer dereference was discovered in H5O_sdspace_encode() in H5Osdspace.c in HDF5 through 1.10.3 allows attackers to cause a denial of service via a crafted HDF5 file. 

#### Affected version - 1.10.3 and before (compiled from source)

##### Command: ./h5repack --low=1 --high=2 -f GZIP=8 -l dset1:CHUNK=5x6 $POC ~/output/test411.h5

###### Debugging: 

```
Source: 

    270	     if(sdim->rank > 0) {
    271	         for(u = 0; u < sdim->rank; u++)
		// f=0x00007fffffffd158  →  [...]  →  "/home/ethan/output/test21.h5", p=0x00007fffffffd150  →  [...]  →  0x0000000000000000, sdim=0x00007fffffffd198  →  [...]  →  0x0000000000000000, u=0x0
 →  272	             H5F_ENCODE_LENGTH(f, p, sdim->size[u]);
    273	         if(flags & H5S_VALID_MAX) {
    274	             for(u = 0; u < sdim->rank; u++)
    275	                 H5F_ENCODE_LENGTH(f, p, sdim->max[u]);
    276	         } /* end if */

gef➤ p sdim->size[u]
Cannot access memory at address 0x0

gef➤  x/i $pc
=> 0x7ffff7952fcf <H5O_sdspace_encode+503>:	mov    rax,QWORD PTR [rax]

gef➤  i r
rax            0x0	0x0
rbx            0x0	0x0
rcx            0x0	0x0
rdx            0x0	0x0
rsi            0x5555558f0d44	0x5555558f0d44
rdi            0x555555865080	0x555555865080
rbp            0x7fffffffd1a0	0x7fffffffd1a0
rsp            0x7fffffffd140	0x7fffffffd140
r8             0x5555558f4c30	0x5555558f4c30
r9             0x5555558f4c30	0x5555558f4c30
r10            0x7aa	0x7aa
r11            0x7ffff794d884	0x7ffff794d884
r12            0x55555555a800	0x55555555a800
r13            0x7fffffffe0e0	0x7fffffffe0e0
r14            0x0	0x0
r15            0x0	0x0
rip            0x7ffff7952fcf	0x7ffff7952fcf <H5O_sdspace_encode+503>
eflags         0x10246	[ PF ZF IF RF ]
cs             0x33	0x33
ss             0x2b	0x2b
ds             0x0	0x0
es             0x0	0x0
fs             0x0	0x0
gs             0x0	0x0

```

###### ASAN Report

```
ASAN:DEADLYSIGNAL
=================================================================
==2517==ERROR: AddressSanitizer: SEGV on unknown address 0x000000000000 (pc 0x7f9350e16417 bp 0x7fff62115240 sp 0x7fff621151e0 T0)
==2517==The signal is caused by a READ memory access.
==2517==Hint: address points to the zero page.
    #0 0x7f9350e16416 in H5O_sdspace_encode /home/ethan/hdf5-develop/src/H5Osdspace.c:272
    #1 0x7f9350e11734 in H5O_sdspace_shared_encode /home/ethan/hdf5-develop/src/H5Oshared.h:137
    #2 0x7f9350dfe660 in H5O_msg_flush /home/ethan/hdf5-develop/src/H5Omessage.c:2187
    #3 0x7f9350d4bc98 in H5O__chunk_serialize /home/ethan/hdf5-develop/src/H5Ocache.c:1660
    #4 0x7f9350d3e0e2 in H5O__cache_serialize /home/ethan/hdf5-develop/src/H5Ocache.c:538
    #5 0x7f93506fe51f in H5C__generate_image /home/ethan/hdf5-develop/src/H5C.c:8654
    #6 0x7f93506dd647 in H5C__flush_single_entry /home/ethan/hdf5-develop/src/H5C.c:6081
    #7 0x7f93506db846 in H5C__flush_ring /home/ethan/hdf5-develop/src/H5C.c:5868
    #8 0x7f9350696b26 in H5C_flush_cache /home/ethan/hdf5-develop/src/H5C.c:1149
    #9 0x7f93505e5686 in H5AC_flush /home/ethan/hdf5-develop/src/H5AC.c:731
    #10 0x7f93509a1bd8 in H5F__flush_phase2 /home/ethan/hdf5-develop/src/H5Fint.c:1807
    #11 0x7f935099a734 in H5F__dest /home/ethan/hdf5-develop/src/H5Fint.c:1128
    #12 0x7f93509a53f2 in H5F_try_close /home/ethan/hdf5-develop/src/H5Fint.c:2156
    #13 0x7f93509a3e76 in H5F__close_cb /home/ethan/hdf5-develop/src/H5Fint.c:1985
    #14 0x7f9350c6c1a5 in H5I_dec_ref /home/ethan/hdf5-develop/src/H5I.c:1262
    #15 0x7f9350c6c581 in H5I_dec_app_ref /home/ethan/hdf5-develop/src/H5I.c:1307
    #16 0x7f93509a32e3 in H5F__close /home/ethan/hdf5-develop/src/H5Fint.c:1927
    #17 0x7f93509648bc in H5Fclose /home/ethan/hdf5-develop/src/H5F.c:683
    #18 0x560586a6706d in copy_objects /home/ethan/hdf5-develop/tools/src/h5repack/h5repack_copy.c:382
    #19 0x560586a5b8c5 in h5repack /home/ethan/hdf5-develop/tools/src/h5repack/h5repack.c:54
    #20 0x560586a5b801 in main /home/ethan/hdf5-develop/tools/src/h5repack/h5repack_main.c:769
    #21 0x7f934fe7c1c0 in __libc_start_main (/lib/x86_64-linux-gnu/libc.so.6+0x211c0)
    #22 0x560586a56129 in _start (/home/ethan/hdf5-develop/hdf5/bin/h5repack+0x1c129)

AddressSanitizer can not provide additional info.
SUMMARY: AddressSanitizer: SEGV /home/ethan/hdf5-develop/src/H5Osdspace.c:272 in H5O_sdspace_encode
==2517==ABORTING

```
=============================================================

