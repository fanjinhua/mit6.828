# Lab1

## Exercise 3
- What is the last instruction of the boot loader executed, 
and what is the first instruction of the kernel it just loaded?

The last instruction executed by the bootloader is a call 
into the kernel: **0x7d61: call *0x10018.**  
The first instruction executed by the kernel is:
> 0x10000c: movw $0x1234,0x472. 
 
- Where is the first instruction of the kernel?
It's in *kern/entry.S* below the **entry** symbol, which is  
    movw $0x1234, 0x472
    
## Exercise 7
The *0x100025: mov %eax,%cr0* instruction sets **CR0_PG** flag,
which enable paging and use the `CR3` register.  
`CR3` enables the processor to translate linear addresses into physical addresses
 by locating the page directory and page tables for the current task.
  Typically, the upper 20 bits of `CR3` become the page directory base 
  register (PDBR), which stores the physical address of the first page directory entry.


>(gdb) x/4x 0x00100000  
0x100000:	0x1badb002	0x00000000	0xe4524ffe	0x7205c766  
(gdb) x/4i 0xf0100000  
   0xf0100000 <_start+4026531828>:	add    %al,(%eax)  
   0xf0100002 <_start+4026531830>:	add    %al,(%eax)  
   0xf0100004 <_start+4026531832>:	add    %al,(%eax)  
   0xf0100006 <_start+4026531834>:	add    %al,(%eax)  
(gdb) x/4x 0xf0100000  
0xf0100000 <_start+4026531828>:	0x00000000	0x00000000	0x00000000	0x00000000


>(gdb) x/10x 0x00100000  
0x100000:	0x1badb002	0x00000000	0xe4524ffe	0x7205c766  
0x100010:	0x34000004	0x0000b812	0x220f0011	0xc0200fd8  
0x100020:	0x0100010d	0xc0220f80  
(gdb) x/10x 0xf0100000  
0xf0100000 <_start+4026531828>:	0x1badb002	0x00000000	0xe4524ffe	0x7205c766  
0xf0100010 <entry+4>:	0x34000004	0x0000b812	0x220f0011	0xc0200fd8  
0xf0100020 <entry+20>:	0x0100010d	0xc0220f80  

- What is the first instruction after the new mapping is established that would fail 
to work properly if the mapping weren't in place? Comment out the movl %eax, %cr0 in 
kern/entry.S, trace into it, and see if you were right.

>=> 0x100020:	or     $0x80010001,%eax  
0x00100020 in ?? ()  
(gdb) si  
=> 0x100025:	mov    $0xf010002c,%eax
0x0010002a in ?? ()
(gdb) si  
=> 0xf010002c <relocated>:	add    %al,(%eax)  
relocated () at kern/entry.S:74  
74		movl	$0x0,%ebp			# nuke frame pointer  
(gdb) si  
Remote connection closed    # failed  


## Exercise 9
The kernel stack is setup in the `entry.S` file.

>movl $0x0, %ebp  
movl $(bootstacktop), %esp

File `memlayout.h` defines values for `PGSHIFT` and `KSTKSIZE`. 

>.data  
　　.p2align	PGSHIFT		# force page alignment  
　　.globl		bootstack  
bootstack:  
　　.space		KSTKSIZE 　　  *(amount of space allocated = KSTKSIZE)*  
　　.globl		bootstacktop     
bootstacktop:

*bootstacktop - bootstack = KSTKSIZE*  
Value of KSTKSIZE is 8*4096, PGSHIFT is 12.  
In `kernel.asm`    

>mov    $0xf0110000,%esp 

So the stack starts at `0xf0110000` and ends at `0xf0110000` - 
8*4096 = '0xf0108000'.
