

##### Description
fig2dev is a library used by Xfig package to translate fig code to other graphical languages (tikz, shape, jpeg, png etc.)               
Wiki: https://en.wikipedia.org/wiki/Xfig


## SIGSEGV due to Memory corruption in free_comments()

Function read_splineobject() in read.c, at line [1078](https://sourceforge.net/p/mcj/fig2dev/ci/3.2.7/tree/fig2dev/read.c#l1078) `s` struct is being passed to Spline_malloc() (internally calls malloc() ), which returns an invalid address '0xbebebebe'.


```
gef➤  p s->fill_style
$22 = 0xbebebebe
gef➤  p s->next
$23 = (struct f_spline *) 0xbebebebebebebebe
gef➤  p s->back_arrow
$24 = (struct f_arrow *) 0xbebebebebebebebe
gef➤  p s->controls
$25 = (struct f_control *) 0xbebebebebebebebe
gef➤  p s->comments
$26 = (struct f_comment *) 0xbebebebebebebebe
gef➤  p s->thickness
$29 = 0xbebebebe
gef➤  p s->depth
$30 = 0xbebebebe
```


Anyhow few of the members are being nulled out later (not having junk value but NULL), but not s->comments, which later is passed to free_comments(). 
```
gef➤  p s->controls
$38 = (struct f_control *) 0x0
gef➤  p s->comments
$39 = (struct f_comment *) 0xbebebebebebebebe`
```

In free_comments(), at line [172](https://sourceforge.net/p/mcj/fig2dev/ci/3.2.7/tree/fig2dev/free.c#l172) when trying to access d->next, Segmentation fault is being triggered as it's pointing to an invalid memory address. 

```
gef➤  p d->next
Cannot access memory at address 0xbebebebebebebec6`
```



###### **Control flow :**
```
Main() - fig2dev.c -> 
readfp_fig() - read.c -> 
read_objects() - read.c -> 
read_splineobject() - read.c ->
free_splinestorage() - free.c -> 
free_comments() - free.c -> 
```

###### **Affected version:**
3.2.7a




###### **Command**:  
fig2dev -L tikz $POC



###### **Backtrace:**

```
#0  0x0000000000424cf6 in free_comments (c=<optimized out>) at free.c:172
#1  free_splinestorage (s=s@entry=0x60800000bf20) at free.c:136
#2  0x000000000043b625 in read_splineobject (fp=fp@entry=0x61600000fc80) at read.c:1140
#3  0x000000000043e0a0 in read_objects (obj=0x7fffffffdc70, fp=0x61600000fc80) at read.c:383
#4  readfp_fig (fp=0x61600000fc80, obj=obj@entry=0x7fffffffdc70) at read.c:172
#5  0x0000000000440237 in read_fig (file_name=<optimized out>, obj=obj@entry=0x7fffffffdc70) at read.c:142
#6  0x0000000000404104 in main (argc=0x4, argv=0x7fffffffddf8) at fig2dev.c:424
```




###### **Debugging:**

```
gef➤  i r
rax            0xc10000017ee	0xc10000017ee
rbx            0x61600000fc80	0x61600000fc80
rcx            0xbebebebebebebec6	0xbebebebebebebec6
rdx            0x17d7d7d7d7d7d7d8	0x17d7d7d7d7d7d7d8
rsi            0xc10000017ed	0xc10000017ed
rdi            0xbebebebebebebebe	0xbebebebebebebebe
rbp            0x60800000bf20	0x60800000bf20
rsp            0x7fffffffd5d0	0x7fffffffd5d0
r8             0x7ffff6b5d770	0x7ffff6b5d770
r9             0x7ffff7fd9780	0x7ffff7fd9780
r10            0xc10000017e9	0xc10000017e9
r11            0xc10000017ea	0xc10000017ea
r12            0x7fffffffd8a0	0x7fffffffd8a0
r13            0x60800000bf20	0x60800000bf20
r14            0x0	0x0
r15            0x7fffffffd9e0	0x7fffffffd9e0
rip            0x424cf6	0x424cf6 <free_splinestorage+950>
eflags         0x10a07	[ CF PF IF OF RF ]
cs             0x33	0x33
ss             0x2b	0x2b
ds             0x0	0x0
es             0x0	0x0
fs             0x0	0x0
gs             0x0	0x0
```



```
gef➤  ptype s->comments
type = struct f_comment {
    char *comment;
    struct f_comment *next;
} *
```


#### ASAN output

```
==115139==ERROR: AddressSanitizer: SEGV on unknown address 0x000000000000 (pc 0x000000424cf6 bp 0x60800000bf20 sp 0x7fffffffd5d0 T0)
    #0 0x424cf5 in free_comments /home/woot/Desktop/fig2dev-3.2.7a/fig2dev/free.c:172
    #1 0x424cf5 in free_splinestorage /home/woot/Desktop/fig2dev-3.2.7a/fig2dev/free.c:136
    #2 0x43b624 in read_splineobject /home/woot/Desktop/fig2dev-3.2.7a/fig2dev/read.c:1140
    #3 0x43e09f in read_objects /home/woot/Desktop/fig2dev-3.2.7a/fig2dev/read.c:383
    #4 0x43e09f in readfp_fig /home/woot/Desktop/fig2dev-3.2.7a/fig2dev/read.c:172
    #5 0x404103 in main /home/woot/Desktop/fig2dev-3.2.7a/fig2dev/fig2dev.c:424
    #6 0x7ffff67b782f in __libc_start_main (/lib/x86_64-linux-gnu/libc.so.6+0x2082f)
    #7 0x406038 in _start (/home/woot/Desktop/fig2dev-3.2.7a/fig2dev/fig2dev+0x406038)

AddressSanitizer can not provide additional info.
SUMMARY: AddressSanitizer: SEGV /home/woot/Desktop/fig2dev-3.2.7a/fig2dev/free.c:172 free_comments
```


Reproducer file -  [reproducer](https://github.com/SegfaultMasters/covering360/blob/master/fig2dev/free_comments_00)





## SIGSEGV - NULL pointer dereference in compute_open_spline()

A macro COPY_CONTROL_POINT at line [192](https://sourceforge.net/p/mcj/fig2dev/ci/master/tree/fig2dev/trans_spline.c#l192) in function compute_open_spline(), swaps the value of passed variables, leaving the value of p2 as 0x0. 


~~~
 #define COPY_CONTROL_POINT(P0, S0, P1, S1) \
      P0 = P1; \
      S0 = S1
~~~
	  
 COPY_CONTROL_POINT(p2, s2, p1->next, s1->next);
 
 
~~~
gef➤  p p1->next
$65 = (struct f_point *) 0x0
~~~
 
 
Later when accessing  `next`, the member of structure pointer `p2`, leads to a segmentation fault as it try to access an invalid memory address.


`if (p2->next == NULL)`





###### **Debugging:**
~~~
$rax   : 0x0               
$rbx   : 0x7fffffffd600      →  0x00007fffffffda40  →  0x00007fffffffda70  →  0x00007fffffffdbd0  →  0x00007fffffffdc00  →  0x00007fffffffdc30  →  0x00007fffffffdd10  →  0x0000000000516090
$rcx   : 0x0               
$rdx   : 0x0               
$rsp   : 0x7fffffffd4e0      →  0x000060800000bf28  →  0x0000000000000000
$rbp   : 0x7fffffffd550      →  0x00007fffffffd630  →  0x00007fffffffda70  →  0x00007fffffffdbd0  →  0x00007fffffffdc00  →  0x00007fffffffdc30  →  0x00007fffffffdd10  →  0x0000000000516090
$rsi   : 0x0               
$rdi   : 0x7ffff715fed0      →  0x0000000000000001
$rip   : 0x423650            →  <compute_open_spline+779> mov rax, QWORD PTR [rax+0x8]
~~~

──────────────────────────────────────────────────────────────

~~~
 0x423644 <compute_open_spline+767> mov    rdi, rax
 0x423647 <compute_open_spline+770> call   0x4022b0 <__asan_report_load8@plt>
 0x42364c <compute_open_spline+775> mov    rax, QWORD PTR [rbp-0x30]
→   0x423650 <compute_open_spline+779> mov    rax, QWORD PTR [rax+0x8]
 0x423654 <compute_open_spline+783> test   rax, rax
 0x423657 <compute_open_spline+786> jne    0x42366b <compute_open_spline+806>
 0x423659 <compute_open_spline+788> mov    rax, QWORD PTR [rbp-0x30]
 0x42365d <compute_open_spline+792> mov    QWORD PTR [rbp-0x28], rax
 0x423661 <compute_open_spline+796> mov    rax, QWORD PTR [rbp-0x18]
~~~

 
 ──────────────────────────────────────────────────────────────
 
~~~
 gef➤  bt 
#0  0x0000000000423650 in compute_open_spline (spline=0x60800000bf20, precision=0.5) at trans_spline.c:193
#1  0x0000000000426667 in create_line_with_spline (s=0x60800000bf20) at trans_spline.c:494
#2  0x0000000000420c76 in read_splineobject (fp=0x61600000fc80) at read.c:1207
#3  0x0000000000419669 in read_objects (fp=0x61600000fc80, obj=0x7fffffffdc80) at read.c:383
#4  0x0000000000418841 in readfp_fig (fp=0x61600000fc80, obj=0x7fffffffdc80) at read.c:172
#5  0x000000000041872a in read_fig (file_name=0x7fffffffe1e9 "/home/woot/Desktop/xfig/ou/crashes/id:000001,sig:11,src:000027,op:flip1,pos:76", obj=0x7fffffffdc80) at read.c:142
#6  0x0000000000410ea4 in main (argc=0x4, argv=0x7fffffffddf8) at fig2dev.c:424
~~~



 
 
###### **ASAN Output:**
 
~~~
 ASAN:SIGSEGV
=================================================================
==75521==ERROR: AddressSanitizer: SEGV on unknown address 0x000000000008 (pc 0x00000044562d bp 0x000000000000 sp 0x7fffffffd4d0 T0)
    #0 0x44562c  (/usr/local/bin/fig2dev+0x44562c)
    #1 0x446c68  (/usr/local/bin/fig2dev+0x446c68)
    #2 0x43b3db  (/usr/local/bin/fig2dev+0x43b3db)
    #3 0x43e09f  (/usr/local/bin/fig2dev+0x43e09f)
    #4 0x404103  (/usr/local/bin/fig2dev+0x404103)
    #5 0x7ffff67b782f in __libc_start_main (/lib/x86_64-linux-gnu/libc.so.6+0x2082f)
    #6 0x406038  (/usr/local/bin/fig2dev+0x406038)
~~~


Reproducer file - [Reproducer](https://github.com/SegfaultMasters/covering360/blob/master/fig2dev/compute_open_spline_01)
 
