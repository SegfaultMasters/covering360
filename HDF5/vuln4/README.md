##### Description

HDF5 is a data model, library, and file format for storing and managing data. It supports an unlimited variety of datatypes, and is designed for flexible and efficient I/O and for high volume and complex data. HDF5 is portable and is extensible, allowing applications to evolve in their use of HDF5. The HDF5 Technology suite includes tools and applications for managing, manipulating, viewing, and analyzing data in the HDF5 format. link: https://portal.hdfgroup.org/display/HDF5/HDF5

## Divided By Zero - POC_apply_filters_h5repack_filters

A SIGFPE signal is raised in the function apply_filters() of h5repack_filters.c in the 'hdf5' package 1.10.x and 1.8.x versions during an attempted parse of a crafted HDF file, because of incorrect protection against division by zero. It could allow a remote denial of service attack.

We believe that vulnerability has been introduced in the commit "0b4b87e36ff3ee8a6b1c24733a5b24b7b9112a1f" 

#### Affected version - 1.10.x and 1.8.x and before (compiled from source)
#### Tested versions - 1.8.20 & 1.10.3


##### Command: ./h5repack -f GZIP=8 -l dset1:CHUNK=5x6 $POC ~/output/test11.h5

###### Debugging

```
gef➤ x/i $pc
=> 0x55555556393c <apply_filters+1129>:	div    QWORD PTR [rbp-0xbd0]

gef➤  i r
rax            0x2000000	0x2000000
rbx            0x0	0x0
rcx            0x11	0x11
rdx            0x0	0x0
rsi            0x55555579b900	0x55555579b900
rdi            0x0	0x0
rbp            0x7fffffffd5a0	0x7fffffffd5a0
rsp            0x7fffffffc970	0x7fffffffc970
r8             0xfcffffc0	0xfcffffc0
r9             0x7fffffffdc80	0x7fffffffdc80
r10            0x5555557a1840	0x5555557a1840
r11            0x7fffffffd1e8	0x7fffffffd1e8
r12            0x55555555a300	0x55555555a300
r13            0x7fffffffe130	0x7fffffffe130
r14            0x0	0x0
r15            0x0	0x0
rip            0x55555556393c	0x55555556393c <apply_filters+1129>
eflags         0x10206	[ PF IF RF ]
cs             0x33	0x33
ss             0x2b	0x2b
ds             0x0	0x0
es             0x0	0x0
fs             0x0	0x0
gs             0x0	0x0

gef➤ x/d $rbp-0xbd0
0x7fffffffc9d0:	0

```
###### Backtrace

```
#0  0x000055555556393c in apply_filters (name=0x5555558dde00 "/dset2", rank=0x12, dims=0x7fffffffd740, msize=0x8, dcpl_id=0xa000014, options=0x7fffffffdc80, has_filter=0x7fffffffd5ec) at h5repack_filters.c:336
#1  0x00005555555602f1 in do_copy_objects (fidin=0x1000000, fidout=0x1000001, travt=0x5555558dd8d0, options=0x7fffffffdc80) at h5repack_copy.c:807
#2  0x000055555555e52e in copy_objects (fnamein=0x7fffffffe4a9 "/home/ethan/hdf5-repack/POC_024", fnameout=0x7fffffffe4d1 "/home/ethan/output/test11.h5", options=0x7fffffffdc80) at h5repack_copy.c:299
#3  0x000055555555a46a in h5repack (infile=0x7fffffffe4a9 "/home/ethan/hdf5-repack/POC_024", outfile=0x7fffffffe4d1 "/home/ethan/output/test11.h5", options=0x7fffffffdc80) at h5repack.c:54
#4  0x000055555556f32d in main (argc=0x7, argv=0x7fffffffe138) at h5repack_main.c:660

```
=============================================================

