# GDB
GDB


**How to debug a user-space binary using GDB**

1) g++ f.c -g\
2) ulimit -c unlimited \
3) ./a.out // Produces the core file \
4) ls core* \
5) gdb a.out <core_fname> \
7) r \
8) run the seg fault scenario \
9) bt \
10) collect bt command logs

**How to debug a kernel module using GDB**

1) Get the debug version of the kernel module. (compiled with -g -DDEBUG)
2) gdb
3) list *(function_name+offset) // It will show the line number.

Note: There is a "gdb-multiarch" which can be used for all the different processor architectures (Eg: ARM/X86/MIPS/SPARC/RISC-V)

**Typical Example of crash:**

Core 0 PC: [br_fdb_find_vid_by_mac+0x50/0xc8] <0xffffffc0807be3c4>
Core 0 LR: [br_port_dev_get+0xe0/0x108] <0xffffffc0807a6a50>

[<0xffffffc0807be3c4>] br_fdb_find_vid_by_mac+0x50/0xc8
[<0xffffffc0807a6a50>] br_port_dev_get+0xe0/0x108

**For the above example use the gdb as below**

gdb example.ko \
gdb> list *(br_fdb_find_vid_by_mac+0x50) // It will give the line number.

**Other GDB commands**
1) To get the offset of a member in a structure. \
    p &((struct sk_buff *)0)->data_len
   
2) In crashscope how to check the value at the location (here, structure contents) \
    v.v (struct structure_name *)0xEAOED040

3) After a crash the values will be printed of all the registers. \
   It can be used along with the objdump to decode all the information.

4) objdump -S -D filename.ko/vmlinux -o output.dump \
   // -S is used to say "disassemble with source" \
   // -D says to disassemble all sections \
   // -o is to redirect the output of the objdump \
   // vmlinux is a statically linked executable file that contains the Linux kernel in one of the object file formats supported by Linux, which includes Executable and Linkable Format (ELF) and Common Object File Format (COFF). The vmlinux file might be required for kernel debugging, symbol table generation 



