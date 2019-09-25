---
layout: post
title: working title
date: 2019-09-24
---

In this post we are going to look at what a simple hello world program written in C compiles into. We will use different variations to print to the screen and compare them. We will then take a look at what a program that accomplishes the same function looks like in assembly code for x86_64 and AArch64. This is Part 1 of 2 of what we worked on for [Lab 3](https://wiki.cdot.senecacollege.ca/wiki/SPO600_Assembler_Lab). 

## Hello World! in C

We are going to take a look at a few different options we have available to print something on the CL with C and what difference it makes each method. To do this we're going to compare the machine instructions we get from the binaries and the file sizes of the following:

- [printf()](#printf)
- [write()](#write)
- [syscall()](#syscall)

### printf()

```c
printf("Hello World!\n");
```
The most common way we would print to the screen in C is by using `printf()`. So, this is what we did for the first iteration of our Hello World program. While it is very simple to use, we ought to remember about the overhead this function has which allows it to be so accessible. Behind the scenes `printf()` needs to determine things like formatting, size of the string, output stream, etc. As a result of this overhead, this option gives us the largest file size at 25KB in x86_64 and 72KB in AArch64.

In x86_64 GCC outputs the following assembler instructions:

```
0000000000401126 <main>:
  401126:       55                      push   %rbp
  401127:       48 89 e5                mov    %rsp,%rbp
  40112a:       bf 10 20 40 00          mov    $0x402010,%edi
  40112f:       b8 00 00 00 00          mov    $0x0,%eax
  401134:       e8 f7 fe ff ff          callq  401030 <printf@plt>
  401139:       b8 00 00 00 00          mov    $0x0,%eax
  40113e:       5d                      pop    %rbp
  40113f:       c3                      retq   
```

The resulting binary on AArch64 looks quite similar:

```
0000000000400594 <main>:
  400594:       a9bf7bfd        stp     x29, x30, [sp, #-16]!
  400598:       910003fd        mov     x29, sp
  40059c:       90000000        adrp    x0, 400000 <_init-0x418>
  4005a0:       9119c000        add     x0, x0, #0x670
  4005a4:       97ffffb7        bl      400480 <printf@plt>
  4005a8:       52800000        mov     w0, #0x0                        // #0
  4005ac:       a8c17bfd        ldp     x29, x30, [sp], #16
  4005b0:       d65f03c0        ret
  4005b4:       00000000        .inst   0x00000000 ; undefined
```

As we can see all our `main()` function needs to do is setup the parameters into the corresponding registers and then do a call to `printf()` to handover the operation. However, what we are not seeing here are all the instructions that the `printf()` procedure generates.

### write()

```c
write(1,"Hello World!\n",13);
```

The next version of the hello world program invokes the `write()` directly instead of going through `printf()`. `write()` is a system call that is ultimately called by `printf()`. We can use it to optimize our code by removing the overhead of the `printf)_` function.

Using `write()` to print out our Hello World message we get the following machine instructions on x86_64:

```
0000000000401126 <main>:
  401126:       55                      push   %rbp
  401127:       48 89 e5                mov    %rsp,%rbp
  40112a:       ba 0d 00 00 00          mov    $0xd,%edx
  40112f:       be 10 20 40 00          mov    $0x402010,%esi
  401134:       bf 01 00 00 00          mov    $0x1,%edi
  401139:       e8 f2 fe ff ff          callq  401030 <write@plt>
  40113e:       b8 00 00 00 00          mov    $0x0,%eax
  401143:       5d                      pop    %rbp
  401144:       c3                      retq   
  401145:       66 2e 0f 1f 84 00 00    nopw   %cs:0x0(%rax,%rax,1)
  40114c:       00 00 00 
  40114f:       90                      nop
```

And on AArch64:

```
0000000000400594 <main>:
  400594:       a9bf7bfd        stp     x29, x30, [sp, #-16]!
  400598:       910003fd        mov     x29, sp
  40059c:       d28001a2        mov     x2, #0xd                        // #13
  4005a0:       90000000        adrp    x0, 400000 <_init-0x418>
  4005a4:       9119e001        add     x1, x0, #0x678
  4005a8:       52800020        mov     w0, #0x1                        // #1
  4005ac:       97ffffb1        bl      400470syscall <write@plt>
  4005b0:       52800000        mov     w0, #0x0                        // #0
  4005b4:       a8c17bfd        ldp     x29, x30, [sp], #16
  4005b8:       d65f03c0        ret
  4005bc:       00000000        .inst   0x00000000 ; undefined

```

Although it seems to have more instructions since the system needs to handle the extra parameters that `write()` takes, we have to remember that `printf()` ultimately makes a call to `write()` and it also has to figure out the formatting of the string. The easiest way to compare this difference is through the size of the binaries, in x86_64 the program using `printf()` results in a 25KB file, while the one calling `write()` is only 23KB, a whole 10% smaller file size. Looking at AArch64 we get 72KB with `printf()` and 71KB using `write()`

### syscall()

```c
syscall(__NR_write,1,"Hello World!\n",13);
```

The third, and final, version of the program we tested tries to even further reduce the number of instructions that the compiler will spit out. We'd normally use the `syscall()` function to call a system call which has no wrapper in C. For the sake of an example we are simply using it to invoke the `__NR_write` system call which allows us to output to stdout (1). 

This is what the machine instructions of `main()` look like in x86_64: 

```
0000000000401126 <main>:
  401126:       55                      push   %rbp
  401127:       48 89 e5                mov    %rsp,%rbp
  40112a:       b9 0d 00 00 00          mov    $0xd,%ecx
  40112f:       ba 10 20 40 00          mov    $0x402010,%edx
  401134:       be 01 00 00 00          mov    $0x1,%esi
  401139:       bf 01 00 00 00          mov    $0x1,%edi
  40113e:       b8 00 00 00 00          mov    $0x0,%eax
  401143:       e8 e8 fe ff ff          callq  401030 <syscall@plt>
  401148:       b8 00 00 00 00          mov    $0x0,%eax
  40114d:       5d                      pop    %rbp
  40114e:       c3                      retq   
  40114f:       90                      nop
```

And below is AArch64:

```
0000000000400594 <main>:
  400594:       a9bf7bfd        stp     x29, x30, [sp, #-16]!
  400598:       910003fd        mov     x29, sp
  40059c:       528001a3        mov     w3, #0xd                        // #13
  4005a0:       90000000        adrp    x0, 400000 <_init-0x418>
  4005a4:       9119e002        add     x2, x0, #0x678
  4005a8:       52800021        mov     w1, #0x1                        // #1
  4005ac:       d2800800        mov     x0, #0x40                       // #64
  4005b0:       97ffffb4        bl      400480 <syscall@plt>
  4005b4:       52800000        mov     w0, #0x0                        // #0
  4005b8:       a8c17bfd        ldp     x29, x30, [sp], #16
  4005bc:       d65f03c0        ret
```

As we can see the machine instructions look similar to what we get when using `printf()` except for the fact that we are making a call to `syscall` in the PLT and we use more registers to hold the parameters since `syscall()` requires several more parameters. 

Surprisingly this method produced a slightly bigger file in x86_64 than the previous version using `write()` did. It is still smaller than using `printf()`, but it was about 10 bytes bigger than a call to `write()` produced. On AArch64 the file is the exact same size.

## Hello World in x86_64 Assembly

Now we are going to take a look at what it takes to accomplish the same task using the GNU and NASM Assemblers on x86_64. The code for the two appears below.

Source code for GNU assembler:

```
.text
.globl  _start

_start:
        movq    $len,%rdx                       /* message length */
        movq    $msg,%rsi                       /* message location */
        movq    $1,%rdi                         /* file descriptor stdout */
        movq    $1,%rax                         /* syscall sys_write */
        syscall

        movq    $0,%rdi                         /* exit status */
        movq    $60,%rax                        /* syscall sys_exit */
        syscall

.section .rodata

msg:    .ascii      "Hello, world!\n"
        len = . - msg
```

Source code for NASM Assembler:

```
section .text
global  _start

_start:
        mov     rdx,len                 ; message length
        mov     rcx,msg                 ; message location
        mov     rbx,1                   ; file descriptor stdout
        mov     rax,4                   ; syscall sys_write
        int     0x80

        mov     rax,1                   ; syscall sys_exit
        int     0x80

section .rodata

msg     db      'Hello, world!',0xa
len     equ     $ - msg
```

The instructions we get from GNU Assembler are these:

```
0000000000401000 <_start>:
  401000:       48 c7 c2 0e 00 00 00    mov    $0xe,%rdx
  401007:       48 c7 c6 00 20 40 00    mov    $0x402000,%rsi
  40100e:       48 c7 c7 01 00 00 00    mov    $0x1,%rdi
  401015:       48 c7 c0 01 00 00 00    mov    $0x1,%rax
  40101c:       0f 05                   syscall 
  40101e:       48 c7 c7 00 00 00 00    mov    $0x0,%rdi
  401025:       48 c7 c0 3c 00 00 00    mov    $0x3c,%rax
  40102c:       0f 05                   syscall 
```

And, the instructions from the NASM Assembler:

```
0000000000401000 <_start>:
  401000:       ba 0e 00 00 00          mov    $0xe,%edx
  401005:       48 b9 00 20 40 00 00    movabs $0x402000,%rcx
  40100c:       00 00 00 
  40100f:       bb 01 00 00 00          mov    $0x1,%ebx
  401014:       b8 04 00 00 00          mov    $0x4,%eax
  401019:       cd 80                   int    $0x80
  40101b:       b8 01 00 00 00          mov    $0x1,%eax
  401020:       cd 80                   int    $0x80
```

The source code for the two versions is fairly similar, and as it would be expected they produce the same number of instructions.

A thing to note is that while on the compiled C we were looking at just the instructions for the `main()` function there were hundreds more that were not shown. On the other hand, the above code blocks are all the instructions needed for the programs written in assembly.

Because of the difference in the number of instructions we see a huge improvement in the size of the program. As listed before the compiled C programs were about 20KB in size, but the assembly code programs produced binaries that are about 9KB in size. 

## Hello World in AArch64

We will now take a quick look at how we would write our Hello World program on AArch64. 

```
.text
.globl _start
_start:

        mov     x0, 1           /* file descriptor: 1 is stdout */
        adr     x1, msg         /* message location (memory address) */
        mov     x2, len         /* message length (bytes) */

        mov     x8, 64          /* write is syscall #64 */
        svc     0               /* invoke syscall */

        mov     x0, 0           /* status -> 0 */
        mov     x8, 93          /* exit is syscall #93 */
        svc     0               /* invoke syscall */

.data
msg:    .ascii      "Hello, world!\n"
len=    . - msg
```

And the following instructions is all we get:

```
00000000004000b0 <_start>:
  4000b0:       d2800020        mov     x0, #0x1                        // #1
  4000b4:       100800e1        adr     x1, 4100d0 <msg>
  4000b8:       d28001c2        mov     x2, #0xe                        // #14
  4000bc:       d2800808        mov     x8, #0x40                       // #64
  4000c0:       d4000001        svc     #0x0
  4000c4:       d2800000        mov     x0, #0x0                        // #0
  4000c8:       d2800ba8        mov     x8, #0x5d                       // #93
  4000cc:       d4000001        svc     #0x0
```

Again there's a drastic difference in the number of instructions compared to compiled C. And the file size difference is even more significant than the halving we saw in x86_64. Our assembly code produced a 1.9KB  file, much more smaller than the 72KB the compiled C took.

## Conclusions

It was interesting to see the difference it makes to write a procedure at a lower level. Although with a small program and current hardware these efficiencies are somewhat insignificant, it was enlightening to learn how much overhead a simple function like `printf()` adds to a program. There's definitively a reason we have created abstractions in the form of higher level languages, the lower level we attempt to write the more bug prone our software becomes and the more difficult it becomes to write more complex applications.

While the programs written in assembly are much more efficient due to their smaller number of instructions, the level of difficulty to write them is significantly greater than writing a single line of a C function. I can't even begin to imagine how difficult it would be to write a program that does more than simply write "Hello World!" to the screen. But as we will see on the next part of this series it only gets more fun as we attempt to do loops and conditionals in assembly.