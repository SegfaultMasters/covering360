##### Description

HDF5 is a data model, library, and file format for storing and managing data. It supports an unlimited variety of datatypes, and is designed for flexible and efficient I/O and for high volume and complex data. HDF5 is portable and is extensible, allowing applications to evolve in their use of HDF5. The HDF5 Technology suite includes tools and applications for managing, manipulating, viewing, and analyzing data in the HDF5 format. link: https://portal.hdfgroup.org/display/HDF5/HDF5

## Divided By Zero - DivByZero__H5D_chunk_POC

A SIGFPE signal is raised in the function H5D__chunk_init of H5Dchunk.c in the 'hdf5' package 1.10.2 during an attempted parse of a crafted HDF file, because of incorrect protection against division by zero. The value of 'dset->shared->layout.u.chunk.dim[u]' becomes zero on the iteration when u=3. This may leads to Denial of Service (application crash).

#### Affected version - 1.10.2 (compiled from source)

##### Command ./h5stat -A -T -G -D -S $POC (DivByZero__H5D_chunk_POC)

###### Debugging

```
break H5Dchunk.c:1022 if u = 3

Breakpoint 2, H5D__chunk_init (f=0x60700000de60, dxpl_id=0xa00000000000008, dset=0x606000000c20, dapl_id=0xa00000000000007) at H5Dchunk.c:1022
1022 - rdcc->scaled_dims[u] = dset->shared->curr_dims[u] / dset->shared->layout.u.chunk.dim[u];
1: dset->shared->layout.u.chunk.dim[u] = 0x0
2: dset->shared->curr_dims[u] = 0x101
3: u = 0x3
```
###### Backtrace

```
#0  H5D__chunk_init (f=0x60700000de60, dxpl_id=0xa00000000000008, dset=0x606000000c20, dapl_id=0xa00000000000007) at H5Dchunk.c:1022
#1  0x00007ffff6285bef in H5D__layout_oh_read (dataset=0x606000000c20, dxpl_id=0xa00000000000008, dapl_id=0xa00000000000007, plist=0x60400000cfd0) at H5Dlayout.c:653
#2  0x00007ffff62668fa in H5D__open_oid (dataset=0x606000000c20, dapl_id=0xa00000000000007, dxpl_id=0xa00000000000008) at H5Dint.c:1598
#3  0x00007ffff62644e5 in H5D_open (loc=0x7fffffffc3f0, dapl_id=0xa00000000000007, dxpl_id=0xa00000000000008) at H5Dint.c:1390
#4  0x00007ffff62638e5 in H5D__open_name (loc=0x7fffffffc580, name=0x602000009470 "/Dataset1", dapl_id=0xa00000000000007, dxpl_id=0xa00000000000008) at H5Dint.c:1324
#5  0x00007ffff61ea0b0 in H5Dopen2 (loc_id=0x100000000000000, name=0x602000009470 "/Dataset1", dapl_id=0xa00000000000007) at H5D.c:293
#6  0x00000000004063ac in ?? ()
#7  0x0000000000407563 in ?? ()
#8  0x0000000000438833 in ?? ()
#9  0x00007ffff64068e9 in H5G_visit_cb (lnk=0x7fffffffcd20, _udata=0x7fffffffd400) at H5Gint.c:925
#10 0x00007ffff641cc50 in H5G__node_iterate (f=0x60700000de60, dxpl_id=0xa00000000000008, _lt_key=0x61200001e048, addr=0x4e0, _rt_key=0x61200001e050, _udata=0x7fffffffd070) at H5Gnode.c:1004
#11 0x00007ffff61461fa in H5B__iterate_helper (f=0x60700000de60, dxpl_id=0xa00000000000008, type=0x7ffff6deba80 <H5B_SNODE>, addr=0x180, op=0x7ffff641c5eb <H5G__node_iterate>, udata=0x7fffffffd070) at H5B.c:1179
#12 0x00007ffff61465c7 in H5B_iterate (f=0x60700000de60, dxpl_id=0xa00000000000008, type=0x7ffff6deba80 <H5B_SNODE>, addr=0x180, op=0x7ffff641c5eb <H5G__node_iterate>, udata=0x7fffffffd070) at H5B.c:1224
#13 0x00007ffff64336ab in H5G__stab_iterate (oloc=0x606000000e68, dxpl_id=0xa00000000000008, order=H5_ITER_INC, skip=0x0, last_lnk=0x0, op=0x7ffff64061cc <H5G_visit_cb>, op_data=0x7fffffffd400) at H5Gstab.c:563
#14 0x00007ffff6425ac9 in H5G__obj_iterate (grp_oloc=0x606000000e68, idx_type=H5_INDEX_NAME, order=H5_ITER_INC, skip=0x0, last_lnk=0x0, op=0x7ffff64061cc <H5G_visit_cb>, op_data=0x7fffffffd400, dxpl_id=0xa00000000000008) at H5Gobj.c:706
#15 0x00007ffff6408508 in H5G_visit (loc_id=0x100000000000000, group_name=0x442e80 "/", idx_type=H5_INDEX_NAME, order=H5_ITER_INC, op=0x438308, op_data=0x7fffffffd630, lapl_id=0xa00000000000000, dxpl_id=0xa00000000000008) at H5Gint.c:1160
#16 0x00007ffff64e7b04 in H5Lvisit_by_name (loc_id=0x100000000000000, group_name=0x442e80 "/", idx_type=H5_INDEX_NAME, order=H5_ITER_INC, op=0x438308, op_data=0x7fffffffd630, lapl_id=0xa00000000000000) at H5L.c:1381
#17 0x0000000000438dca in ?? ()
#18 0x000000000043c6e4 in ?? ()
#19 0x000000000040c21e in ?? ()
#20 0x00007ffff5ba7830 in __libc_start_main (main=0x40bb52, argc=0x7, argv=0x7fffffffddf8, init=<optimized out>, fini=<optimized out>, rtld_fini=<optimized out>, stack_end=0x7fffffffdde8) at ../csu/libc-start.c:291
#21 0x0000000000404e59 in ?? ()
```
=============================================================

## Stack Overflow - stackoverflow_H5P__get_cb

##### Description

A stack-overflow has been detected in the function H5P__get_cb() in H5Pint.c of HDF5 package (1.10.2) during an attempted parse of a crafted HDF file, because of handling an exceptional case when the buffer is too large. This cause the program to crash with segmenation fault SIGSEGV. 

#### Affected version - 1.10.2 (compiled from source)

##### Command: ./h5stat -A -T -G -D -S $POC

##### registers 
```
$rax   : 0x7ffff5cd49b0      →  <__memmove_avx_unaligned+0> mov rax, rdi
$rbx   : 0x8               
$rcx   : 0x1               
$rdx   : 0x8               
$rsp   : 0x7fffff7fee40    
$rbp   : 0x7fffff7ff6b0      →  0x00007fffff7ff700  →  0x00007fffff7ff760  →  0x00007fffff7ff820  →  0x00007fffff7ff8e0  →  0x00007fffff7ffb10  →  0x00007fffff7ffbe0  →  0x00007fffff7ffcb0
$rsi   : 0x602000009eb0      →  0x0000000000000060 ("`"?)
$rdi   : 0x7fffff7ff880      →  0x00000c04000b1e33  →  0x0000000000000000
$rip   : 0x7ffff6ef662f      →  <__asan_memcpy+111> call rax
$r8    : 0x7fffff7ff7c0      →  0x00007fffff7ff880  →  0x00000c04000b1e33  →  0x0000000000000000
$r9    : 0x65c             
$r10   : 0x62d001b44400      →  0x000000000000864c
$r11   : 0x7fffff7ff5c0      →  0x00007fffff7ff7a0  →  0x0000000041b58ab3
$r12   : 0x7fffff7ff880      →  0x00000c04000b1e33  →  0x0000000000000000
$r13   : 0x7ffff7381744      →  0x0000000000000001
$r14   : 0x7fffff7ff7a0      →  0x0000000041b58ab3
$r15   : 0x602000009eb0      →  0x0000000000000060 ("`"?)
```

#### ASAN Output
```
Filename: /home/woot/Hdf5_crashes/hdf5_hls_results/stack-overflow_POC
ASAN:SIGSEGV
=================================================================
==4432==ERROR: AddressSanitizer: stack-overflow on address 0x7ffd76478fa8 (pc 0x7f79626c662f bp 0x7ffd76479820 sp 0x7ffd76478fb0 T0)
    #0 0x7f79626c662e in __asan_memcpy (/usr/lib/x86_64-linux-gnu/libasan.so.2+0x8c62e)
    #1 0x7f7961e5a971 in H5P__get_cb /home/woot/hdf5-hdf5-1_10_2/src/H5Pint.c:4329
    #2 0x7f7961e52a3f in H5P__do_prop /home/woot/hdf5-hdf5-1_10_2/src/H5Pint.c:2641
    #3 0x7f7961e5acce in H5P_get /home/woot/hdf5-hdf5-1_10_2/src/H5Pint.c:4383
    #4 0x7f796190046d in H5AC_tag /home/woot/hdf5-hdf5-1_10_2/src/H5AC.c:2784
    #5 0x7f7961bf2099 in H5G__obj_get_linfo /home/woot/hdf5-hdf5-1_10_2/src/H5Gobj.c:323
    #6 0x7f7961bf9df9 in H5G__obj_lookup /home/woot/hdf5-hdf5-1_10_2/src/H5Gobj.c:1137
    #7 0x7f7961c11c8f in H5G_traverse_real /home/woot/hdf5-hdf5-1_10_2/src/H5Gtraverse.c:593
    #8 0x7f7961c1404e in H5G_traverse /home/woot/hdf5-hdf5-1_10_2/src/H5Gtraverse.c:866
    #9 0x7f7961be08d5 in H5G_loc_info /home/woot/hdf5-hdf5-1_10_2/src/H5Gloc.c:744
    #10 0x7f7961cf5ba7 in H5Oget_info_by_name /home/woot/hdf5-hdf5-1_10_2/src/H5O.c:535
    #11 0x43868b  (/home/woot/hdf5-hdf5-1_10_2/hdf5/bin/h5stat+0x43868b)
    #12 0x7f7961bd68e8 in H5G_visit_cb /home/woot/hdf5-hdf5-1_10_2/src/H5Gint.c:925
    #13 0x7f7961becc4f in H5G__node_iterate /home/woot/hdf5-hdf5-1_10_2/src/H5Gnode.c:1004
    #14 0x7f79619161f9 in H5B__iterate_helper /home/woot/hdf5-hdf5-1_10_2/src/H5B.c:1179
    #15 0x7f79619165c6 in H5B_iterate /home/woot/hdf5-hdf5-1_10_2/src/H5B.c:1224
    #16 0x7f7961c036aa in H5G__stab_iterate /home/woot/hdf5-hdf5-1_10_2/src/H5Gstab.c:563
    #17 0x7f7961bf5ac8 in H5G__obj_iterate /home/woot/hdf5-hdf5-1_10_2/src/H5Gobj.c:706
    #18 0x7f7961bd72cf in H5G_visit_cb /home/woot/hdf5-hdf5-1_10_2/src/H5Gint.c:1009
    #19 0x7f7961becc4f in H5G__node_iterate /home/woot/hdf5-hdf5-1_10_2/src/H5Gnode.c:1004
    #20 0x7f79619161f9 in H5B__iterate_helper /home/woot/hdf5-hdf5-1_10_2/src/H5B.c:1179
    #21 0x7f79619165c6 in H5B_iterate /home/woot/hdf5-hdf5-1_10_2/src/H5B.c:1224
    #22 0x7f7961c036aa in H5G__stab_iterate /home/woot/hdf5-hdf5-1_10_2/src/H5Gstab.c:563
    #23 0x7f7961bf5ac8 in H5G__obj_iterate /home/woot/hdf5-hdf5-1_10_2/src/H5Gobj.c:706
    #24 0x7f7961bd72cf in H5G_visit_cb /home/woot/hdf5-hdf5-1_10_2/src/H5Gint.c:1009
    #25 0x7f7961becc4f in H5G__node_iterate /home/woot/hdf5-hdf5-1_10_2/src/H5Gnode.c:1004
    #26 0x7f79619161f9 in H5B__iterate_helper /home/woot/hdf5-hdf5-1_10_2/src/H5B.c:1179
    #27 0x7f79619165c6 in H5B_iterate /home/woot/hdf5-hdf5-1_10_2/src/H5B.c:1224
    #28 0x7f7961c036aa in H5G__stab_iterate /home/woot/hdf5-hdf5-1_10_2/src/H5Gstab.c:563
    #29 0x7f7961bf5ac8 in H5G__obj_iterate /home/woot/hdf5-hdf5-1_10_2/src/H5Gobj.c:706
    #30 0x7f7961bd72cf in H5G_visit_cb /home/woot/hdf5-hdf5-1_10_2/src/H5Gint.c:1009
    #31 0x7f7961becc4f in H5G__node_iterate /home/woot/hdf5-hdf5-1_10_2/src/H5Gnode.c:1004
    #32 0x7f79619161f9 in H5B__iterate_helper /home/woot/hdf5-hdf5-1_10_2/src/H5B.c:1179
    #33 0x7f79619165c6 in H5B_iterate /home/woot/hdf5-hdf5-1_10_2/src/H5B.c:1224
    #34 0x7f7961c036aa in H5G__stab_iterate /home/woot/hdf5-hdf5-1_10_2/src/H5Gstab.c:563
    #35 0x7f7961bf5ac8 in H5G__obj_iterate /home/woot/hdf5-hdf5-1_10_2/src/H5Gobj.c:706
    .....
    .....
    .....
SUMMARY: AddressSanitizer: stack-overflow ??:0 __asan_memcpy
==4432==ABORTING
```

