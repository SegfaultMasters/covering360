
##### Description
WildMIDI is a simple software midi player which has a core softsynth library that can be used with other applications.             
Vendor: http://www.mindwerks.net/projects/wildmidi



## Denial of service due to recursive loop in main()

An inline function `wmidi_write()` called by `write_wav_output()` from `send_output()` at line [2063](https://github.com/Mindwerks/wildmidi/blob/master/src/wildmidi.c#L2063) in `wildmidi.c` is recursively being called inside a while loop as no break statement is issued, results in generating a WAV file of unrestricted size, causing the system to hang up.



###### **Affected version:**
0.4.3 (Master)




###### **Command**:  
wildmidi -o /home/aceteam/wildmidi/some.WAV $POC


 
### Debugging

```
(gdb) 
2063                if (send_output(output_buffer, res) < 0) { [ 0%] -  
(gdb) s
write_wav_output (output_data=0x629000005200 "", output_size=16384) at /home/woot/Desktop/wildmidi-master/src/wildmidi.c:495
495	    if (wmidi_write(audio_fd, output_data, output_size) < 0) {
(gdb) s
wmidi_write (fd=3, buf=0x629000005200, size=16384) at /home/woot/Desktop/wildmidi-master/src/wildmidi.c:380
380	    return write(fd, buf, size);
```

```
(gdb) n 100
Initializing Sound System
Initializing libWildMidi 0.4.3

 +  Volume up        e  Better resampling    n  Next Midi
 -  Volume down      l  Log volume           q  Quit
 ,  1sec Seek Back   r  Reverb               .  1sec Seek Forward
 m  save as midi     p  Pause On/Off

Playing $POC 
[Approx 1092m 48s Total]
2063                if (send_output(output_buffer, res) < 0) { [ 0%] /  
(gdb) n 100
2063                if (send_output(output_buffer, res) < 0) { [ 0%] /  
(gdb) n 1000
2063                if (send_output(output_buffer, res) < 0) { [ 0%] /  
(gdb) n 5000
2063                if (send_output(output_buffer, res) < 0) { [ 0%] /  
(gdb) n 50000
     =             [    ] [100] [ 3m  9s Processed] [ 0%] -  %] \   [ 0%] -  WAV file size keeps increasing
	 
2054                perc_play = (wm_info->current_sample * 100)[ 0%] \  
```




###### **Backtrace:**
 
 ```
(gdb) bt
#0  wmidi_write (fd=3, buf=0x629000005200, size=16384) at /home/woot/Desktop/wildmidi-master/src/wildmidi.c:380
#1  0x000000000040253a in write_wav_output (output_data=0x629000005200 "", output_size=16384) at /home/woot/Desktop/wildmidi-master/src/wildmidi.c:495
#2  0x00000000004054f3 in main (argc=4, argv=0x7fffffffde38) at /home/woot/Desktop/wildmidi-master/src/wildmidi.c:2063
```


```
(gdb) i r
rax            0x4000	16384
rbx            0x7fffffffdd30	140737488346416
rcx            0x7ffff69322c0	140737330225856
rdx            0x0	0
rsi            0x0	0
rdi            0xc527fff8a40	13548474305088
rbp            0x7fffffffd990	0x7fffffffd990
rsp            0x7fffffffd980	0x7fffffffd980
r8             0x10000000000	1099511627776
r9             0xc527fff9240	13548474307136
r10            0xffffffff	4294967295
r11            0x246	582
r12            0xffffffffb4a	17592186043210
r13            0x7fffffffda50	140737488345680
r14            0x7fffffffda50	140737488345680
r15            0x0	0
rip            0x4025ea	0x4025ea <write_wav_output+217>
eflags         0x206	[ PF IF ]
cs             0x33	51
ss             0x2b	43
ds             0x0	0
es             0x0	0
fs             0x0	0
gs             0x0	0
```


Please do confirm if you are able to reproduce the issue with this [Reproducer](https://github.com/SegfaultMasters/covering360/blob/master/wildmidi/hang_main_00)
