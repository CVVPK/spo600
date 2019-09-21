
>Keep in mind:
```
-g               # enable debugging information
-O0              # do not optimize (that's a capital letter and then the digit zero)
-fno-builtin     # do not use builtin function optimizations
```

Default compile w/o any options:

```as
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


First compile with all compiler options:
```as
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

Second compile `-static`:

```as
0000000000401bb5 <main>:
  401bb5:       55                      push   %rbp
  401bb6:       48 89 e5                mov    %rsp,%rbp
  401bb9:       bf 10 00 48 00          mov    $0x480010,%edi
  401bbe:       b8 00 00 00 00          mov    $0x0,%ea
  401bc3:       e8 f8 72 00 00          callq  408ec0 <_IO_printf>
  401bc8:       b8 00 00 00 00          mov    $0x0,%eax
  401bcd:       5d                      pop    %rbp
  401bce:       c3                      retq   
  401bcf:       90                      nop

```
Size difference of using `-static` option:
```
-rwxrwxr-x. 1 cval cval  25K Sep 17 10:23 all
-rwxrwxr-x. 1 cval cval 1.7M Sep 17 10:23 static
```

Compile result without -fno-builtin:

```as
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

Compile result of removing the `-g` option:

```as
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
Difference in size from removing `-g`:
```
-rwxrwxr-x. 1 cval cval 25K Sep 17 10:23 all
-rwxrwxr-x. 1 cval cval 22K Sep 17 10:24 nog

```

Compile result of adding one extra param to printf:
```as
0000000000401126 <main>:
  401126:       55                      push   %rbp
  401127:       48 89 e5                mov    %rsp,%rbp
  40112a:       be d2 04 00 00          mov    $0x4d2,%esi
  40112f:       bf 10 20 40 00          mov    $0x402010,%edi
  401134:       b8 00 00 00 00          mov    $0x0,%eax
  401139:       e8 f2 fe ff ff          callq  401030 <printf@plt>
  40113e:       b8 00 00 00 00          mov    $0x0,%eax
  401143:       5d                      pop    %rbp
  401144:       c3                      retq   
  401145:       66 2e 0f 1f 84 00 00    nopw   %cs:0x0(%rax,%rax,1)
  40114c:       00 00 00 
  40114f:       90                      nop

```

And now adding 2 params:
```as
0000000000401126 <main>:
  401126:       55                      push   %rbp
  401127:       48 89 e5                mov    %rsp,%rbp
  40112a:       ba d8 d3 00 00          mov    $0xd3d8,%edx
  40112f:       be d2 04 00 00          mov    $0x4d2,%esi
  401134:       bf 10 20 40 00          mov    $0x402010,%edi
  401139:       b8 00 00 00 00          mov    $0x0,%eax
  40113e:       e8 ed fe ff ff          callq  401030 <printf@plt>
  401143:       b8 00 00 00 00          mov    $0x0,%eax
  401148:       5d                      pop    %rbp
  401149:       c3                      retq   
  40114a:       66 0f 1f 44 00 00       nopw   0x0(%rax,%rax,1)

```

Compile result of moving `printf()` to a an `output()` function and then calling `output()` from within `main()`:
```as
000000000040113c <main>:
  40113c:       55                      push   %rbp
  40113d:       48 89 e5                mov    %rsp,%rbp
  401140:       b8 00 00 00 00          mov    $0x0,%eax
  401145:       e8 dc ff ff ff          callq  401126 <output>
  40114a:       b8 00 00 00 00          mov    $0x0,%eax
  40114f:       5d                      pop    %rbp
  401150:       c3                      retq   
  401151:       66 2e 0f 1f 84 00 00    nopw   %cs:0x0(%rax,%rax,1)
  401158:       00 00 00 
  40115b:       0f 1f 44 00 00          nopl   0x0(%rax,%rax,1)

```

Compile result of removing the `-O0` option and replacing it with `-O3`:

```as
0000000000401040 <main>:
  401040:       48 83 ec 08             sub    $0x8,%rsp
  401044:       bf 10 20 40 00          mov    $0x402010,%edi
  401049:       31 c0                   xor    %eax,%eax
  40104b:       e8 e0 ff ff ff          callq  401030 <printf@plt>
  401050:       31 c0                   xor    %eax,%eax
  401052:       48 83 c4 08             add    $0x8,%rsp
  401056:       c3                      retq   

```