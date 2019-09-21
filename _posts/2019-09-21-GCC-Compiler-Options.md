---
layout: post
title: GCC Compiler Options
date: 2019-09-21T18:00:00Z
categories: Lab
---

For [Lab 2](https://wiki.cdot.senecacollege.ca/wiki/SPO600_Compiled_C_Lab) we were asked to compile a simple Hello World program in C using a variety of GCC compiler options, we were then to take a look at what the resulting assembly instructions looked like and how the different compiler options may affect performance.

<!-- more -->

The different variations of the compiled program we worked on were:
- [Using -g -O0 -fno-builtin](#using--g--o0--fno-builtin)
- [Removing -g](#removing--g)
- [Replacing -O0 with -O3](#replacing--o0-with--o3)
- [Removing -fno-builtin](#removing--fno-builtin)
- [Adding -static](#adding--static)
- [Adding additional arguments to printf()](#adding-additional-arguments-to-printf)
- [Moving printf() outside of main()](#moving-printf-outside-of-main)

The C code used looked like this:

```c
#include <stdio.h>

int main() {
    printf("Hello World!\n");
}
```

### Using -g -O0 -fno-builtin

Doing a default compile without any options we get the resulting assembly code:

```
0000000000401126 <main>:
  401126:       55                      push   %rbp
  401127:       48 89 e5                mov    %rsp,%rbp
  40112a:       bf 10 20 40 00          mov    $0x402010,%edi
  40112f:       e8 fc fe ff ff          callq  401030 <puts@plt>
  401134:       b8 00 00 00 00          mov    $0x0,%eax
  401139:       5d                      pop    %rbp
  40113a:       c3                      retq
  40113b:       0f 1f 44 00 00          nopl   0x0(%rax,%rax,1)

```

Using the `-g -O0 -fno-builtin` options we get the following assembly code:

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

As we'll explore further down the [`-g-`](#removing--g) option enables debugging information, which has an impact in file size and not in the assembly code we see here. [`-O0`](#replacing--o0-with--o3) tells the compiler to not optimize the code which can accelerate the compilation process. And lastly, [`-fno-builtin`](#removing--fno-builtin) turns off the builtin function optimizations.

### Removing -g

The `-g` option allows us to get extra debugging information that can be useful if we're using GDB as our debugger, the man page for GCC does warn us that other debuggers may even refuse to run our program if this option is turned on.

The resulting assembly instructions with and without the `-g` option were exactly the same. However, turning it on did result in a slightly bigger executable. The resulting binary file with `-g` on was 25K while without it resulted in a 22K file. This might not seem like much, but that is a 12% increase in file size.

### Replacing -O0 with -O3

I'm using GCC v7.4 and according to the man page the `-O0` option is turned on by default. The `-O0` option turns off all optimization flags for a faster compilation time.

We replaced the `-O0` option with `-O3` for maximum optimization by the compiler. This resulted in the following assembly instructions:

<pre>
<code>
0000000000401040 &lt;main&gt;:
  <b>401040:       48 83 ec 08             sub    $0x8,%rsp</b>
  401044:       bf 10 20 40 00          mov    $0x402010,%edi
  <b>401049:       31 c0                   xor    %eax,%eax</b>
  40104b:       e8 e0 ff ff ff          callq  401030 <printf@plt>
  <b>401050:       31 c0                   xor    %eax,%eax</b>
  <b>401052:       48 83 c4 08             add    $0x8,%rsp</b>
  401056:       c3                      retq   
</code>
</pre>
The differences that the `-O3` option makes are the `sub`, `xor` and `add` instructions that replace `push`, `mov` and `pop` from the non optimized binary. 


### Removing -fno-builtin

The `-fno-builtin` option tells the compiler to not use the the built-in function optimizations. This means that some code that could be optimized, like the call to `printf()` in our program, won't be changed to a more efficient function like `puts()`. Removing the `-fno-builtin` option results in the following instructions:

<pre>
<code>
0000000000401126 &lt;main&gt;:
  401126:       55                      push   %rbp
  401127:       48 89 e5                mov    %rsp,%rbp
  40112a:       bf 10 20 40 00          mov    $0x402010,%edi
  <b>40112f:       e8 fc fe ff ff          callq  401030 <puts@plt></b>
  401134:       b8 00 00 00 00          mov    $0x0,%eax
  401139:       5d                      pop    %rbp
  40113a:       c3                      retq   
  40113b:       0f 1f 44 00 00          nopl   0x0(%rax,%rax,1)
</code>
</pre>


### Adding -static

The `-static` option prevents dynamic linking of the shared libraries.

By default, if supported by the system, `gcc` uses dynamic linking of shared libraries. However, we can force static linking by using the `-static` option. Using dynamic linking results in a much smaller compiled program since function calls to the shared libraries are resolved at run time. On the other hand with statically linked libraries the entire library must be compiled alongside our code, resulting in an necessarily big program.

The size difference between a statically linked program and a dynamic linked one will vary depending on the size of the libraries used, but for our example we are only using the stdio.h library, yet statically linking a program results in a 1.7MB program compared to the 25KB dynamically linked version.

When we added the `-static` option the resulting instructions looked like this:

<pre>
<code>
0000000000401bb5 &lt;main&gt;:
  401bb5:       55                      push   %rbp
  401bb6:       48 89 e5                mov    %rsp,%rbp
  401bb9:       bf 10 00 48 00          mov    $0x480010,%edi
  401bbe:       b8 00 00 00 00          mov    $0x0,%eax
  <b>401bc3:       e8 f8 72 00 00          callq  408ec0 &lt;_IO_printf&gt;</b>
  401bc8:       b8 00 00 00 00          mov    $0x0,%eax
  401bcd:       5d                      pop    %rbp
  401bce:       c3                      retq
  401bcf:       90                      nop
</code>
</pre>

The main difference we can notice is in the line were the function call is done, in the dynamically linked program the function `printf()` gets called from the Procedure Linkage Table (PLT), while on the statically linked program the function called is `_IO_printf` and as we can see it doesn't reference the PLT since it has already been resolved at compile time.

### Adding additional arguments to printf()

As expected adding more parameters results in a greater instruction count, here's what the compiler spits out when we add a single extra parameter to `printf()`:

<pre>
<code>
0000000000401126 &lt;main&gt;:
  401126:       55                      push   %rbp
  401127:       48 89 e5                mov    %rsp,%rbp
  <b>40112a:       be d2 04 00 00          mov    $0x4d2,%esi</b>
  40112f:       bf 10 20 40 00          mov    $0x402010,%edi
  401134:       b8 00 00 00 00          mov    $0x0,%eax
  401139:       e8 f2 fe ff ff          callq  401030 <printf@plt>
  40113e:       b8 00 00 00 00          mov    $0x0,%eax
  401143:       5d                      pop    %rbp
  401144:       c3                      retq   
  401145:       66 2e 0f 1f 84 00 00    nopw   %cs:0x0(%rax,%rax,1)
  40114c:       00 00 00 
  40114f:       90                      nop
</code>
</pre>

As you can see it simply adds one more `mov` instruction. And adding a third parameter results in just another `mov` instruction:

<pre>
<code>
0000000000401126 &lt;main&gt;:
  401126:       55                      push   %rbp
  401127:       48 89 e5                mov    %rsp,%rbp
  <b>40112a:       ba d8 d3 00 00          mov    $0xd3d8,%edx</b>
  <b>40112f:       be d2 04 00 00          mov    $0x4d2,%esi</b>
  401134:       bf 10 20 40 00          mov    $0x402010,%edi
  401139:       b8 00 00 00 00          mov    $0x0,%eax
  40113e:       e8 ed fe ff ff          callq  401030 <printf@plt>
  401143:       b8 00 00 00 00          mov    $0x0,%eax
  401148:       5d                      pop    %rbp
  401149:       c3                      retq   
  40114a:       66 0f 1f 44 00 00       nopw   0x0(%rax,%rax,1)
</code>
</pre>

### Moving printf() outside of main()

Lastly, we tried moving `printf()` to a user function `output()`. This resulted in the `callq` instruction inside `main()` invoking our own `output()` function.

<pre>
<code>
000000000040113c &lt;main&gt;:
  40113c:       55                      push   %rbp
  40113d:       48 89 e5                mov    %rsp,%rbp
  401140:       b8 00 00 00 00          mov    $0x0,%eax
  <b>401145:       e8 dc ff ff ff          callq  401126 &lt;output&gt;</b>
  40114a:       b8 00 00 00 00          mov    $0x0,%eax
  40114f:       5d                      pop    %rbp
  401150:       c3                      retq   
  401151:       66 2e 0f 1f 84 00 00    nopw   %cs:0x0(%rax,%rax,1)
  401158:       00 00 00 
  40115b:       0f 1f 44 00 00          nopl   0x0(%rax,%rax,1)
</code>
</pre>

Here's what the `output()` function looks like. Which is basically a repeat of `main()` when we were calling `printf()` from it.

```
0000000000401126 <output>:
  401126:       55                      push   %rbp
  401127:       48 89 e5                mov    %rsp,%rbp
  40112a:       bf 10 20 40 00          mov    $0x402010,%edi
  40112f:       b8 00 00 00 00          mov    $0x0,%eax
  401134:       e8 f7 fe ff ff          callq  401030 <printf@plt>
  401139:       90                      nop
  40113a:       5d                      pop    %rbp
  40113b:       c3                      retq   
```
Moving the `printf()` call to another function resulted in nearly double the number of instructions, with today's processing times this may not seem as much but it's good to keep in mind to avoid useless wrappers.
