---
layout: post
title: Working Title
date: 2019-09-17
---

For [Lab 2](https://wiki.cdot.senecacollege.ca/wiki/SPO600_Compiled_C_Lab) we were asked to compile a simple Hello World program in C using different compiler options, we were then to take a look at what the assembly code looks like and how the different compiler options affected it. 

The different variations of the compiled program we worked on were:
- Added the [`-static`](#-static) compiler option
- Removed the [`-fno-builtin`](#-fno-builtin)compiler option
- Removed the [`-g`](#-g) compiler option
- Added additional arguments to `printf()`
- Moved `printf()` to a separate function outside `main()`
- Replaced the [`-O0`](#-O0) option with `-O3`

<!-- more -->

The C code used looks like this:

```c
#include <stdio.h>

int main() {
    printf("Hello World!\n");
}
```

### -static

The `-static` option prevents dynamic linking of the shared libraries.

By default, if supported by the system, `gcc` uses dynamic linking of shared libraries. However, we can force static linking by using the `-static` option. Using dynamic linking results in a much smaller compiled program since function calls to the shared libraries are resolved at run time. On the other hand with statically linked libraries the entire library must be compiled alongside our code, resulting in an unecessarily big program.

The size difference between a statically linked program and a dynamic linked one will vary depending on the size of the libraries used, but for our example we are only using the stdio.h library, yet statically linking a program results in a 1.7MB program compared to the 25KB dynamically linked version.

A dynamically linked program compiled with the options `-g -O0 -fno-builtin` results in the following assembly code:

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

Adding the `-static` option results in this:

```
0000000000401bb5 <main>:
  401bb5:       55                      push   %rbp
  401bb6:       48 89 e5                mov    %rsp,%rbp
  401bb9:       bf 10 00 48 00          mov    $0x480010,%edi
  401bbe:       b8 00 00 00 00          mov    $0x0,%eax
  401bc3:       e8 f8 72 00 00          callq  408ec0 <_IO_printf>
  401bc8:       b8 00 00 00 00          mov    $0x0,%eax
  401bcd:       5d                      pop    %rbp
  401bce:       c3                      retq   
  401bcf:       90                      nop
```

The main difference we can notice is in the line were the function call is done, in the dynamically linked program the function `printf()` gets called from the Procedure Linkage Table (PLT), while on the statically linked program the function called is `_IO_printf` and as we can see it doesn't reference the PLT since it has already been resolved at compile time.


