---
layout: post
title: Why learn assembly?
date: 2019-09-25
categories: Thoughts
---

Most programmers know that the source code we write must be translated into something the machine actually understands. What we often never do is actually look at what the end result looks like. For most of our code what it compiles into is of little importance, with today's hardware there is just no reason to look at the resulting machine instructions most of the time. Except for some unlucky ones, who have long lost their sanity, most programmers will never have to bother with trying to understand assembly language.

If you have ever done an `objdump` on a binary, you have probably seen something that looks kind of like this:

```
0000000000401126 <main>:
  401126:       55                      push   %rbp
  401127:       48 89 e5                mov    %rsp,%rbp
  40112a:       bf 10 20 40 00          mov    $0x402010,%edi
  40112f:       b8 00 00 00 00          mov    $0x0,%eax
  401134:       e8 f7 fe ff ff          callq  401030 <printf@plt>
  401139:       b8 00 00 00 00          mov    $0x0,%eax
  40113e:       5d                      pop    %rbp
  40113f:       c3                      retq   `
```

If you had never seen a block similar to the one above, then I'm sorry for what I have done to you. Here are some tips on [finding a mental health provider][mental health].

On a more serious note, you might be trying to understand what is happening in the above code. Well, each line is an instruction, lets ignore the hexadecimal numbers for now since what we are interested in are the assembly instructions. Those look like `push %rbp`, `mov %rsp, %rbp`. 

These instructions are working with [cpu registers][registers]. For example `mov %rsp, %rbp` means move the bits currently in register `rsp` to register `rbp`. Here's a link to the different register names in [x86_64][x86-registers] and for [AArch64][aarch64-registers].

The `push` and `pop` instructions, as you may imagine if you're familiar with [stacks][stack-ds], are referring to pushing and popping data into and from the stack. `callq` is a function call. Here's a link to some of the basic [x86_64][x86-instructions] and [AArch64][aarch64-instructions] instructions.

You may have noticed by now, that although I'm showing you the results of a compiled C program in x86_64 architecture, I keep placing links to AArch64 references. This is because assembly language is architecture dependant. So if you wish to learn assembly more in depth you'll be most likely working with a single architecture at a time.

Assembly language can be quite intimidating to learn. The above instructions are the result of doing a simple `printf("Hello World!\n")`. So it is simply setting up the parameters passed to the `printf()`, yet it is 7 different instructions.

If you mostly write in a [high level language][high level language], like most of us do, you are used to writing very human friendly code, or at least I would hope so. This human friendly code may sometimes have consequences in the number of instructions resulting from the compilations. If anything learning assembly can help you see some of the differences that your code makes, and be able to wager the impact that certain operations will have.

As the tech market seems to  continuously move towards mobile platforms, the performance of our applications may need to be taken into account. Not because of memory constraints, or clock speeds. But because of energy consumption.

Mobile architectures like [AArch64][aarch64] are designed to save the battery life of the device. It is because of energy constraints that we may need to consider the impact our code has. Maybe a wrapper function that doesn't add anything to the program, other than a different way to read the source code, is unnecessary.

Another reason you may want to learn the basics of assembly is because you are trying to port software from one architecture to another. In order to better utilize the features of separate architectures there may be a need to know what the compiler is producing. Or if you're willing, even write assembly directly, but I don't think many people are this crazy.

There are some other reasons than the ones I list here. For me, I was first interested in it because I like knowing how things work at a lower level. Having a low level understanding of anything does two things for me. First, it helps me appreciate the ease of higher level explanations. And second, I believe it strengthens my knowledge base which in turn improves my reasoning.

[registers]: https://en.wikipedia.org/wiki/Processor_register
[mental health]: https://www.mayoclinic.org/diseases-conditions/mental-illness/in-depth/mental-health-providers/art-20045530
[x86-registers]:https://wiki.cdot.senecacollege.ca/wiki/X86_64_Register_and_Instruction_Quick_Start#Registers
[aarch64-registers]: https://wiki.cdot.senecacollege.ca/wiki/Aarch64_Register_and_Instruction_Quick_Start#Registers
[high level language]: https://en.wikipedia.org/wiki/High-level_programming_language
[aarch64]: https://en.wikichip.org/wiki/arm/aarch64