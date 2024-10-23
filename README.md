# Debugging using GDB and objdump in userspace and kernel space
GDB


**How to debug a user-space binary using GDB**

1) g++ f.c -g
2) ulimit -c unlimited
3) ./a.out // Produces the core file
4) ls core*
5) gdb a.out <core_fname>
7) r
8) run the seg fault scenario
9) bt
10) collect bt command logs

**How to debug a kernel module using GDB**

1) Get the debug version of the kernel module. (compiled with -g -DDEBUG)
2) gdb
3) list *(function_name+offset) // It will show the line number.

Note: There is a "gdb-multiarch" which can be used for all the different processor architectures (Eg: ARM/X86/MIPS/SPARC/RISC-V)

**Typical Example of crash:**

Core 0 PC: [br_fdb_find_vid_by_mac+0x50/0xc8] <0xffffffc0807be3c4>\
Core 0 LR: [br_port_dev_get+0xe0/0x108] <0xffffffc0807a6a50>

[<0xffffffc0807be3c4>] br_fdb_find_vid_by_mac+0x50/0xc8\
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

**Example crash**

We get the information of PC, LR, SP, and all the registers value as highlighted.

**[346002.647934] PC is at neigh_get_next+0x2c/0xd0
[346002.652790] LR is at neigh_seq_next+0x30/0x74**
[346002.657303] pc : [<42611ac0>]    lr : [<42613754>]    psr: a0000013
**[346002.661732] sp : b4953e68  ip : 8f91d000  fp : 8e5d0a00
[346002.668241] r10: 9a2186a0  r9 : b4953f78  r8 : 42b3fc00
[346002.673536] r7 : 8e601b80  r6 : 00000000  r5 : a68b7740  r4 : 8a150a00
[346002.678832] r3 : 00000000  r2 : 8efd6400  r1 : 8e5d0a00  r0 : 9a218688**
[346002.685171] Flags: NzCv  IRQs on  FIQs on  Mode SVC_32  ISA ARM  Segment user
[346002.691767] Control: 10c0383d  Table: 9764406a  DAC: 00000055
[346002.699060] Process ip_daemon_process (pid: 16106, stack limit = 0x79bcdda6)
[346002.704874] Stack: (0xb4953e68 to 0xb4954000)
[346002.711038] 3e60:                   9a2186a0 9a218688 a68b7740 000003fd 36df07c8 42613754
[346002.715554] 3e80: 9a218688 00000400 00000000 000003fd 36df07c8 4225c1ec 0000000c 00000000
[346002.723801] 3ea0: aef83840 9a2186b8 00000001 ba112c00 4225bf40 00000000 00000000 b4953f78
[346002.732047] 3ec0: 00000400 00000003 36df0730 4228f58c aef83840 4228f534 42b06e48 422383cc
[346002.740294] 3ee0: 93ade708 00000101 00000000 00a33058 00000020 4227955c 00000002 b4953f00
[346002.748540] 3f00: b7c1cd90 b9418550 4228f0f4 ba81a080 86d9501a 00000002 00000001 a997d3b8
[346002.756787] 3f20: 00000000 42b06e48 00000000 00000000 00000000 6633c8c9 aef83840 00000400
[346002.765034] 3f40: 36df07c8 00000000 b4953f78 00000400 00000003 422385c0 aef83840 36df07c8
[346002.773279] 3f60: aef83840 b4953f78 36df07c8 42b06e48 aef83840 42238870 00000000 00000000
[346002.781526] 3f80: 00000001 6633c8c9 36ef1843 36df0730 00000001 3e98f8eb 00000003 42101204
[346002.789773] 3fa0: b4950000 42101000 36df0730 00000001 00000006 36df07c8 00000400 00000000
[346002.798019] 3fc0: 36df0730 00000001 3e98f8eb 00000003 36f051b4 00000000 3e98fb68 36df0730
[346002.806266] 3fe0: 00024084 3e98f8b8 36ecbaa8 36ecb6d4 60000010 00000006 00000000 00000000
[346002.814520] [<42611ac0>] (neigh_get_next) from [<42613754>] (neigh_seq_next+0x30/0x74)
[346002.822768] [<42613754>] (neigh_seq_next) from [<4225c1ec>] (seq_read+0x2ac/0x4d0)
[346002.830668] [<4225c1ec>] (seq_read) from [<4228f58c>] (proc_reg_read+0x58/0x90)
[346002.838310] [<4228f58c>] (proc_reg_read) from [<422383cc>] (__vfs_read+0x28/0x18c)
[346002.845941] [<422383cc>] (__vfs_read) from [<422385c0>] (vfs_read+0x90/0x108)
[346002.853314] [<422385c0>] (vfs_read) from [<42238870>] (ksys_read+0x60/0xc8)
[346002.860607] [<42238870>] (ksys_read) from [<42101000>] (ret_fast_syscall+0x0/0x50)
[346002.867810] Exception stack(0xb4953fa8 to 0xb4953ff0)
[346002.875187] 3fa0:                   36df0730 00000001 00000006 36df07c8 00000400 00000000
[346002.880397] 3fc0: 36df0730 00000001 3e98f8eb 00000003 36f051b4 00000000 3e98fb68 36df0730
[346002.888644] 3fe0: 00024084 3e98f8b8 36ecbaa8 36ecb6d4
[346002.896890] Code: e5957008 1a00000b ea000003 e5943124 (e5933320) 
[346002.902149] ---[ end trace eda0e678802f4c7a ]---

