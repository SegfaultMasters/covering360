##### Description

HDF5 is a data model, library, and file format for storing and managing data. It supports an unlimited variety of datatypes, and is designed for flexible and efficient I/O and for high volume and complex data. HDF5 is portable and is extensible, allowing applications to evolve in their use of HDF5. The HDF5 Technology suite includes tools and applications for managing, manipulating, viewing, and analyzing data in the HDF5 format. link: https://portal.hdfgroup.org/display/HDF5/HDF5

## Divided By Zero - H5D__create_chunk_file_map_hyper_div_zero (CVE-2018-17233)

A SIGFPE signal is raised in the function H5D__create_chunk_file_map_hyper() of H5Dchunk.c in the 'hdf5' package 1.10.3 and before during an attempted parse of a crafted HDF file, because of incorrect protection against division by zero. It could allow a remote denial of service attack.

#### Affected version - 1.10.3 and before (compiled from source)

##### Command: h5dump -r -d BAG_root/metadata $POC 

###### Debugging

```
{
DATASET "BAG_root/metadata" {
   DATATYPE  H5T_STRING {
      STRSIZE 1;
      STRPAD H5T_STR_NULLTERM;
      CSET H5T_CSET_ASCII;
      CTYPE H5T_C_S1;
}
   DATASPACE  SIMPLE { ( 4795 ) / ( H5S_UNLIMITED ) }

Program received signal SIGFPE, Arithmetic exception.
0x00007ffff6140acf in H5D__create_chunk_file_map_hyper (fm=0x61e000000c80, io_info=0x7fffffffb910) at H5Dchunk.c:1578
1578	        scaled[u] = start_scaled[u] = sel_start[u] / fm->layout->u.chunk.dim[u];

(gdb) x/i $pc
=> 0x7ffff6140acf <H5D__create_chunk_file_map_hyper+1018>:	div    rdi

(gdb) info registers 
rax            0x7ffff668b280	140737327444608
rbx            0x7fffffffb320	140737488335648
rcx            0x0	0
rdx            0x0	0
rsi            0x7ffff668b280	140737327444608
rdi            0x0	0
rbp            0x7fffffffb340	0x7fffffffb340
rsp            0x7fffffffaa30	0x7fffffffaa30
r8             0x7	7
r9             0x61e000000c80	107614700571776
r10            0x3d1	977
r11            0x7ffff66882e1	140737327432417
r12            0xffffffff550	17592186041680
r13            0x7fffffffaa80	140737488333440
r14            0x7fffffffaa80	140737488333440
r15            0x7fffffffb3e0	140737488335840
rip            0x7ffff6140acf	0x7ffff6140acf <H5D__create_chunk_file_map_hyper+1018>
eflags         0x10206	[ PF IF RF ]
cs             0x33	51
ss             0x2b	43
ds             0x0	0
es             0x0	0
fs             0x0	0
gs             0x0	0

```
###### Backtrace

```
ASAN:DEADLYSIGNAL
=================================================================
==37286==ERROR: AddressSanitizer: FPE on unknown address 0x7ffff6140acf (pc 0x7ffff6140acf bp 0x7fffffffb340 sp 0x7fffffffaa30 T0)
    #0 0x7ffff6140ace in H5D__create_chunk_file_map_hyper /home/ethan/hdf5-1_10_3_gcc/src/H5Dchunk.c:1578
    #1 0x7ffff613dfa0 in H5D__chunk_io_init /home/ethan/hdf5-1_10_3_gcc/src/H5Dchunk.c:1169
    #2 0x7ffff61b6702 in H5D__read /home/ethan/hdf5-1_10_3_gcc/src/H5Dio.c:589
    #3 0x7ffff61b2515 in H5Dread /home/ethan/hdf5-1_10_3_gcc/src/H5Dio.c:198
    #4 0x5555555bce14  (/home/ethan/hdf5-1_10_3_gcc/hdf5/bin/h5dump+0x68e14)
    #5 0x5555555be2b4  (/home/ethan/hdf5-1_10_3_gcc/hdf5/bin/h5dump+0x6a2b4)
    #6 0x5555555cc6de  (/home/ethan/hdf5-1_10_3_gcc/hdf5/bin/h5dump+0x786de)
    #7 0x555555582a85  (/home/ethan/hdf5-1_10_3_gcc/hdf5/bin/h5dump+0x2ea85)
    #8 0x5555555881c1  (/home/ethan/hdf5-1_10_3_gcc/hdf5/bin/h5dump+0x341c1)
    #9 0x555555579872  (/home/ethan/hdf5-1_10_3_gcc/hdf5/bin/h5dump+0x25872)
    #10 0x7ffff5aa41c0 in __libc_start_main (/lib/x86_64-linux-gnu/libc.so.6+0x211c0)
    #11 0x555555572129  (/home/ethan/hdf5-1_10_3_gcc/hdf5/bin/h5dump+0x1e129)
```
=============================================================

