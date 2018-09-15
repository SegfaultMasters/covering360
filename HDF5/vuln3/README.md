##### Description

HDF5 is a data model, library, and file format for storing and managing data. It supports an unlimited variety of datatypes, and is designed for flexible and efficient I/O and for high volume and complex data. HDF5 is portable and is extensible, allowing applications to evolve in their use of HDF5. The HDF5 Technology suite includes tools and applications for managing, manipulating, viewing, and analyzing data in the HDF5 format. link: https://portal.hdfgroup.org/display/HDF5/HDF5

## Memory leak - H5O__chunk_deserialize_memory_leak

Memory leak in the H5O__chunk_deserialize() function in H5Ocache.c in HDF HDF5 through 1.10.3 allows attackers to cause a denial of service (memory consumption) via a crafted HDF5 file. 

#### Affected version - 1.10.3 and before (compiled from source)

##### Command: ./h5dump -r -d BAG_root/metadata $POC 


###### Debugging: valgrind --leak-check=full ./h5dump -r -d BAG_root/metadata $POC

Running the above command in 1.8.20 gives the information about H5O__chunk_deserialize


###### Backtrace

```
h5dump error: internal error (file h5dump.c:line 1493)

=================================================================
==37399==ERROR: LeakSanitizer: detected memory leaks

Direct leak of 24 byte(s) in 1 object(s) allocated from:
    #0 0x7f89da102b50 in __interceptor_malloc (/usr/lib/x86_64-linux-gnu/libasan.so.4+0xdeb50)
    #1 0x55a606934e5d  (/home/ethan/hdf5-1_10_3_gcc/hdf5/bin/h5dump+0x82e5d)
    #2 0x55a606935835  (/home/ethan/hdf5-1_10_3_gcc/hdf5/bin/h5dump+0x83835)
    #3 0x55a6068d2d23  (/home/ethan/hdf5-1_10_3_gcc/hdf5/bin/h5dump+0x20d23)
    #4 0x55a6068d6b6d  (/home/ethan/hdf5-1_10_3_gcc/hdf5/bin/h5dump+0x24b6d)
    #5 0x7f89d8cae1c0 in __libc_start_main (/lib/x86_64-linux-gnu/libc.so.6+0x211c0)

Direct leak of 24 byte(s) in 1 object(s) allocated from:
    #0 0x7f89da102b50 in __interceptor_malloc (/usr/lib/x86_64-linux-gnu/libasan.so.4+0xdeb50)
    #1 0x55a606934e5d  (/home/ethan/hdf5-1_10_3_gcc/hdf5/bin/h5dump+0x82e5d)
    #2 0x55a60693584d  (/home/ethan/hdf5-1_10_3_gcc/hdf5/bin/h5dump+0x8384d)
    #3 0x55a6068d2d23  (/home/ethan/hdf5-1_10_3_gcc/hdf5/bin/h5dump+0x20d23)
    #4 0x55a6068d6b6d  (/home/ethan/hdf5-1_10_3_gcc/hdf5/bin/h5dump+0x24b6d)
    #5 0x7f89d8cae1c0 in __libc_start_main (/lib/x86_64-linux-gnu/libc.so.6+0x211c0)

Direct leak of 24 byte(s) in 1 object(s) allocated from:
    #0 0x7f89da102b50 in __interceptor_malloc (/usr/lib/x86_64-linux-gnu/libasan.so.4+0xdeb50)
    #1 0x55a606934e5d  (/home/ethan/hdf5-1_10_3_gcc/hdf5/bin/h5dump+0x82e5d)
    #2 0x55a606935841  (/home/ethan/hdf5-1_10_3_gcc/hdf5/bin/h5dump+0x83841)
    #3 0x55a6068d2d23  (/home/ethan/hdf5-1_10_3_gcc/hdf5/bin/h5dump+0x20d23)
    #4 0x55a6068d6b6d  (/home/ethan/hdf5-1_10_3_gcc/hdf5/bin/h5dump+0x24b6d)
    #5 0x7f89d8cae1c0 in __libc_start_main (/lib/x86_64-linux-gnu/libc.so.6+0x211c0)

Indirect leak of 480 byte(s) in 1 object(s) allocated from:
    #0 0x7f89da102b50 in __interceptor_malloc (/usr/lib/x86_64-linux-gnu/libasan.so.4+0xdeb50)
    #1 0x55a606934eda  (/home/ethan/hdf5-1_10_3_gcc/hdf5/bin/h5dump+0x82eda)
    #2 0x55a606935835  (/home/ethan/hdf5-1_10_3_gcc/hdf5/bin/h5dump+0x83835)
    #3 0x55a6068d2d23  (/home/ethan/hdf5-1_10_3_gcc/hdf5/bin/h5dump+0x20d23)
    #4 0x55a6068d6b6d  (/home/ethan/hdf5-1_10_3_gcc/hdf5/bin/h5dump+0x24b6d)
    #5 0x7f89d8cae1c0 in __libc_start_main (/lib/x86_64-linux-gnu/libc.so.6+0x211c0)

Indirect leak of 480 byte(s) in 1 object(s) allocated from:
    #0 0x7f89da102b50 in __interceptor_malloc (/usr/lib/x86_64-linux-gnu/libasan.so.4+0xdeb50)
    #1 0x55a606934eda  (/home/ethan/hdf5-1_10_3_gcc/hdf5/bin/h5dump+0x82eda)
    #2 0x55a606935841  (/home/ethan/hdf5-1_10_3_gcc/hdf5/bin/h5dump+0x83841)
    #3 0x55a6068d2d23  (/home/ethan/hdf5-1_10_3_gcc/hdf5/bin/h5dump+0x20d23)
    #4 0x55a6068d6b6d  (/home/ethan/hdf5-1_10_3_gcc/hdf5/bin/h5dump+0x24b6d)
    #5 0x7f89d8cae1c0 in __libc_start_main (/lib/x86_64-linux-gnu/libc.so.6+0x211c0)

Indirect leak of 480 byte(s) in 1 object(s) allocated from:
    #0 0x7f89da102b50 in __interceptor_malloc (/usr/lib/x86_64-linux-gnu/libasan.so.4+0xdeb50)
    #1 0x55a606934eda  (/home/ethan/hdf5-1_10_3_gcc/hdf5/bin/h5dump+0x82eda)
    #2 0x55a60693584d  (/home/ethan/hdf5-1_10_3_gcc/hdf5/bin/h5dump+0x8384d)
    #3 0x55a6068d2d23  (/home/ethan/hdf5-1_10_3_gcc/hdf5/bin/h5dump+0x20d23)
    #4 0x55a6068d6b6d  (/home/ethan/hdf5-1_10_3_gcc/hdf5/bin/h5dump+0x24b6d)
    #5 0x7f89d8cae1c0 in __libc_start_main (/lib/x86_64-linux-gnu/libc.so.6+0x211c0)

Indirect leak of 20 byte(s) in 1 object(s) allocated from:
    #0 0x7f89da09b538 in strdup (/usr/lib/x86_64-linux-gnu/libasan.so.4+0x77538)
    #1 0x55a606935b84  (/home/ethan/hdf5-1_10_3_gcc/hdf5/bin/h5dump+0x83b84)
    #2 0x55a60693539c  (/home/ethan/hdf5-1_10_3_gcc/hdf5/bin/h5dump+0x8339c)
    #3 0x55a6069372a5  (/home/ethan/hdf5-1_10_3_gcc/hdf5/bin/h5dump+0x852a5)
    #4 0x7f89d955943b in H5G_visit_cb /home/ethan/hdf5-1_10_3_gcc/src/H5Gint.c:988
    #5 0x7f89d957125f in H5G__node_iterate /home/ethan/hdf5-1_10_3_gcc/src/H5Gnode.c:1002
    #6 0x7f89d927907b in H5B__iterate_helper /home/ethan/hdf5-1_10_3_gcc/src/H5B.c:1165
    #7 0x7f89d9279417 in H5B_iterate /home/ethan/hdf5-1_10_3_gcc/src/H5B.c:1210
    #8 0x7f89d9587bdc in H5G__stab_iterate /home/ethan/hdf5-1_10_3_gcc/src/H5Gstab.c:556
    #9 0x7f89d957a58f in H5G__obj_iterate /home/ethan/hdf5-1_10_3_gcc/src/H5Gobj.c:697
    #10 0x7f89d9559e9f in H5G_visit_cb /home/ethan/hdf5-1_10_3_gcc/src/H5Gint.c:1072
    #11 0x7f89d957125f in H5G__node_iterate /home/ethan/hdf5-1_10_3_gcc/src/H5Gnode.c:1002
    #12 0x7f89d927907b in H5B__iterate_helper /home/ethan/hdf5-1_10_3_gcc/src/H5B.c:1165
    #13 0x7f89d9279417 in H5B_iterate /home/ethan/hdf5-1_10_3_gcc/src/H5B.c:1210
    #14 0x7f89d9587bdc in H5G__stab_iterate /home/ethan/hdf5-1_10_3_gcc/src/H5Gstab.c:556
    #15 0x7f89d957a58f in H5G__obj_iterate /home/ethan/hdf5-1_10_3_gcc/src/H5Gobj.c:697
    #16 0x7f89d955b38a in H5G_visit /home/ethan/hdf5-1_10_3_gcc/src/H5Gint.c:1220
    #17 0x7f89d96516e9 in H5L__visit /home/ethan/hdf5-1_10_3_gcc/src/H5L.c:3480
    #18 0x7f89d96446d6 in H5Lvisit_by_name /home/ethan/hdf5-1_10_3_gcc/src/H5L.c:1312
    #19 0x55a606937ab0  (/home/ethan/hdf5-1_10_3_gcc/hdf5/bin/h5dump+0x85ab0)
    #20 0x55a60693b6a1  (/home/ethan/hdf5-1_10_3_gcc/hdf5/bin/h5dump+0x896a1)
    #21 0x55a6069359a7  (/home/ethan/hdf5-1_10_3_gcc/hdf5/bin/h5dump+0x839a7)
    #22 0x55a6068d2d23  (/home/ethan/hdf5-1_10_3_gcc/hdf5/bin/h5dump+0x20d23)
    #23 0x55a6068d6b6d  (/home/ethan/hdf5-1_10_3_gcc/hdf5/bin/h5dump+0x24b6d)
    #24 0x7f89d8cae1c0 in __libc_start_main (/lib/x86_64-linux-gnu/libc.so.6+0x211c0)

Indirect leak of 10 byte(s) in 1 object(s) allocated from:
    #0 0x7f89da09b538 in strdup (/usr/lib/x86_64-linux-gnu/libasan.so.4+0x77538)
    #1 0x55a606935b84  (/home/ethan/hdf5-1_10_3_gcc/hdf5/bin/h5dump+0x83b84)
    #2 0x55a606935304  (/home/ethan/hdf5-1_10_3_gcc/hdf5/bin/h5dump+0x83304)
    #3 0x55a6069372a5  (/home/ethan/hdf5-1_10_3_gcc/hdf5/bin/h5dump+0x852a5)
    #4 0x7f89d955943b in H5G_visit_cb /home/ethan/hdf5-1_10_3_gcc/src/H5Gint.c:988
    #5 0x7f89d957125f in H5G__node_iterate /home/ethan/hdf5-1_10_3_gcc/src/H5Gnode.c:1002
    #6 0x7f89d927907b in H5B__iterate_helper /home/ethan/hdf5-1_10_3_gcc/src/H5B.c:1165
    #7 0x7f89d9279417 in H5B_iterate /home/ethan/hdf5-1_10_3_gcc/src/H5B.c:1210
    #8 0x7f89d9587bdc in H5G__stab_iterate /home/ethan/hdf5-1_10_3_gcc/src/H5Gstab.c:556
    #9 0x7f89d957a58f in H5G__obj_iterate /home/ethan/hdf5-1_10_3_gcc/src/H5Gobj.c:697
    #10 0x7f89d955b38a in H5G_visit /home/ethan/hdf5-1_10_3_gcc/src/H5Gint.c:1220
    #11 0x7f89d96516e9 in H5L__visit /home/ethan/hdf5-1_10_3_gcc/src/H5L.c:3480
    #12 0x7f89d96446d6 in H5Lvisit_by_name /home/ethan/hdf5-1_10_3_gcc/src/H5L.c:1312
    #13 0x55a606937ab0  (/home/ethan/hdf5-1_10_3_gcc/hdf5/bin/h5dump+0x85ab0)
    #14 0x55a60693b6a1  (/home/ethan/hdf5-1_10_3_gcc/hdf5/bin/h5dump+0x896a1)
    #15 0x55a6069359a7  (/home/ethan/hdf5-1_10_3_gcc/hdf5/bin/h5dump+0x839a7)
    #16 0x55a6068d2d23  (/home/ethan/hdf5-1_10_3_gcc/hdf5/bin/h5dump+0x20d23)
    #17 0x55a6068d6b6d  (/home/ethan/hdf5-1_10_3_gcc/hdf5/bin/h5dump+0x24b6d)
    #18 0x7f89d8cae1c0 in __libc_start_main (/lib/x86_64-linux-gnu/libc.so.6+0x211c0)

Indirect leak of 2 byte(s) in 1 object(s) allocated from:
    #0 0x7f89da09b538 in strdup (/usr/lib/x86_64-linux-gnu/libasan.so.4+0x77538)
    #1 0x55a606935b84  (/home/ethan/hdf5-1_10_3_gcc/hdf5/bin/h5dump+0x83b84)
    #2 0x55a606935304  (/home/ethan/hdf5-1_10_3_gcc/hdf5/bin/h5dump+0x83304)
    #3 0x55a60693779a  (/home/ethan/hdf5-1_10_3_gcc/hdf5/bin/h5dump+0x8579a)
    #4 0x55a60693b6a1  (/home/ethan/hdf5-1_10_3_gcc/hdf5/bin/h5dump+0x896a1)
    #5 0x55a6069359a7  (/home/ethan/hdf5-1_10_3_gcc/hdf5/bin/h5dump+0x839a7)
    #6 0x55a6068d2d23  (/home/ethan/hdf5-1_10_3_gcc/hdf5/bin/h5dump+0x20d23)
    #7 0x55a6068d6b6d  (/home/ethan/hdf5-1_10_3_gcc/hdf5/bin/h5dump+0x24b6d)
    #8 0x7f89d8cae1c0 in __libc_start_main (/lib/x86_64-linux-gnu/libc.so.6+0x211c0)

SUMMARY: AddressSanitizer: 1544 byte(s) leaked in 9 allocation(s).
```
=============================================================

