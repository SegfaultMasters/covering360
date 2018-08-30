

##### Description
vcftools is a suite of functions for use on genetic variation data in the form of VCF and BCF files. The tools provided will be used mainly to summarize data, run calculations on data, filter out data, and convert data into other useful file formats.              
Vendor: https://vcftools.github.io/man_latest.html


## SIGABRT due to received negative value in bcf_entry::set_ALT()

There is an SIGABRT signal due to a received zero value (*in n_allele*), which is being passed to `ALT.resize(n_allele-1)` in function `bcf_entry::set_ALT()` at bcf_entry_setters.cpp.
The zero value is being introduced in `bcf_entry::parse_basic_entry()` function (bcf_entry.ccp),  where `N_allele` value is being assigned by right shifting `n_allele_info` by 16, which again is zero.  

`N_allele  = n_allele_info >> 16`

When the `n_allele` value is zero, the computation(0-1) will result in -1 negative value, giving an invalid length. 
Whenever a invalid size is passed to vector::resize, it throws an std::length_error exception, internally calling abort(), raising an SIGABRT.  




###### **Debugging:**

```
gef➤  p n_allele
$1 = 0x0
gef➤  ptype ALT
type = std::vector<std::string>
```


###### **Backtrace:**

```
gef➤  bt
#0  0x00007ffff6f7b428 in __GI_raise (sig=sig@entry=0x6) at ../sysdeps/unix/sysv/linux/raise.c:54
#1  0x00007ffff6f7d02a in __GI_abort () at abort.c:89
#2  0x00007ffff78bd8f7 in ?? () from /usr/lib/x86_64-linux-gnu/libstdc++.so.6
#3  0x00007ffff78c3a46 in ?? () from /usr/lib/x86_64-linux-gnu/libstdc++.so.6
#4  0x00007ffff78c3a81 in std::terminate() () from /usr/lib/x86_64-linux-gnu/libstdc++.so.6
#5  0x00007ffff78c3cb4 in __cxa_throw () from /usr/lib/x86_64-linux-gnu/libstdc++.so.6
#6  0x00007ffff78bf7a9 in ?? () from /usr/lib/x86_64-linux-gnu/libstdc++.so.6
#7  0x000000000041d08b in std::vector<std::__cxx11::basic_string<char, std::char_traits<char>, std::allocator<char> >, std::allocator<std::__cxx11::basic_string<char, std::char_traits<char>, std::allocator<char> > > >::_M_check_len (__s=0x4b6531 "vector::_M_fill_insert", __n=0xffffffffffffffff, this=0x70c348) at /usr/include/c++/5/bits/stl_vector.h:1425
#8  std::vector<std::__cxx11::basic_string<char, std::char_traits<char>, std::allocator<char> >, std::allocator<std::__cxx11::basic_string<char, std::char_traits<char>, std::allocator<char> > > >::_M_fill_insert (this=this@entry=0x70c348, __position=__position@entry=non-dereferenceable iterator for std::vector, __n=__n@entry=0xffffffffffffffff, __x="") at /usr/include/c++/5/bits/vector.tcc:489
#9  0x00000000004156e2 in std::vector<std::__cxx11::basic_string<char, std::char_traits<char>, std::allocator<char> >, std::allocator<std::__cxx11::basic_string<char, std::char_traits<char>, std::allocator<char> > > >::insert (__x="", __n=0xffffffffffffffff, __position=..., this=0x70c348) at /usr/include/c++/5/bits/stl_vector.h:1073
#10 std::vector<std::__cxx11::basic_string<char, std::char_traits<char>, std::allocator<char> >, std::allocator<std::__cxx11::basic_string<char, std::char_traits<char>, std::allocator<char> > > >::resize (__x=<error reading variable: access outside bounds of object referenced via synthetic pointer>, __new_size=0xffffffffffffffff, this=0x70c348) at /usr/include/c++/5/bits/stl_vector.h:716
#11 bcf_entry::set_ALT (this=this@entry=0x70bf00, n_allele=0x0) at bcf_entry_setters.cpp:13
#12 0x000000000040d7f6 in bcf_entry::parse_basic_entry (this=0x70bf00, parse_ALT=<optimized out>, parse_FILTER=<optimized out>, parse_INFO=<optimized out>) at bcf_entry.cpp:115
#13 0x0000000000475a6a in variant_file::write_stats (this=this@entry=0x709580, params=...) at variant_file_output.cpp:5416
#14 0x0000000000406436 in main (argc=<optimized out>, argv=<optimized out>) at vcftools.cpp:58
```



```

gef➤  i r
rax            0x0	0x0
rbx            0x7071a8	0x7071a8
rcx            0x7ffff6f7b428	0x7ffff6f7b428
rdx            0x6	0x6
rsi            0xdcc	0xdcc
rdi            0xdcc	0xdcc
rbp            0x6d4b00	0x6d4b00 <stderr@@GLIBC_2.2.5>
rsp            0x7fffffffcdb8	0x7fffffffcdb8
r8             0x7ffff730c770	0x7ffff730c770
r9             0x7ffff7fd9740	0x7ffff7fd9740
r10            0x8	0x8
r11            0x246	0x246
r12            0x70bce0	0x70bce0
r13            0xffffffffffffffff	0xffffffffffffffff
r14            0x7fffffffd050	0x7fffffffd050
r15            0x70c348	0x70c348
rip            0x7ffff6f7b428	0x7ffff6f7b428 <__GI_raise+56>
eflags         0x246	[ PF ZF IF ]
cs             0x33	0x33
ss             0x2b	0x2b
ds             0x0	0x0
es             0x0	0x0
fs             0x0	0x0
gs             0x0	0x0
```
