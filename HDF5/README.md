##### Description

HDF5 is a data model, library, and file format for storing and managing data. It supports an unlimited variety of datatypes, and is designed for flexible and efficient I/O and for high volume and complex data. HDF5 is portable and is extensible, allowing applications to evolve in their use of HDF5. The HDF5 Technology suite includes tools and applications for managing, manipulating, viewing, and analyzing data in the HDF5 format. link: https://portal.hdfgroup.org/display/HDF5/HDF5

##### Vulnerability related to DivByZero__H5D_chunk_POC

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
