---
layout: post
title: Vectorization
date: 2019-10-08
categories: Lab
---

Vectorization refers to converting an algorithm from performing one operation at a time to doing a set of multiple operations on each instruction cycle. Modern CPUs include a set of 128-bit vector registers that can be broken into multiple (most commonly 4) smaller blocks. If for example we break a 128-bit register into 4 32-bit blocks, then we can perform 4 operations at a time, in an ideal world this will result in our code being 4 times faster.

In this post we are going to take a look at other ways we can optimize the code we used in [Scaling Audio Through Software] by taking advantage of vectorization in an AARCH64 machine. It describes the work done for [Lab 5].

## Auto-vectorization

Most modern compilers are smart enough to identify operations that may be vectorized and produce the necessary machine instructions. We can ask `GCC` to gives us a look into how it decides whether to vectorize a loop or not by using the `-fopt-info-vec-all` option.

If a loop is vectorized we will see something like this:

```
vol1.c:12:5: note: vectorized 2 loops in function.
```

To see what a vectorized loop looks like in machine instructions we can use `objdump --source file`. The instructions that will let us know that vectorization is indeed happening will look something like this:

```
4005e4:       0f10a401        sxtl    v1.4s, v0.4h
4005e8:       4f10a400        sxtl2   v0.4s, v0.8h
4005ec:       4e21d821        scvtf   v1.4s, v1.4s
4005f0:       4e21d800        scvtf   v0.4s, v0.4s
4005f4:       6e22dc21        fmul    v1.4s, v1.4s, v2.4s
4005f8:       6e22dc00        fmul    v0.4s, v0.4s, v2.4s
4005fc:       4ea1b821        fcvtzs  v1.4s, v1.4s
400600:       4ea1b800        fcvtzs  v0.4s, v0.4s
400604:       0e612823        xtn     v3.4h, v1.4s
400608:       4e612803        xtn2    v3.8h, v0.4s
```

Notice the `v` before the register number, this means that the register is being used as a vector register. The decimal value indicates the number of blocks the register is broken into and the letter after it (h or s) indicates whether they're half words or single words.

For example `v1.4s` indicates that register `v1` is broken into 4 single word blocks, i.e. 4x 32-bit registers. On the other hand `v0.8h` means that `v0` is broken into 8x 16-bit registers.

If the compiler couldn't vectorize a loop it will give us the reason why.

```
vol1.c:38:2: note: === vect_analyze_data_refs ===
vol1.c:38:2: note: got vectype for stmt: _18 = *_17;
vector(8) short int
vol1.c:38:2: note: not vectorized: not enough data-refs in basic block.
vol1.c:38:2: note: ===vect_slp_analyze_bb===
```

This is in reference to the following loop:

```c
for (x = 0; x < SAMPLES; x++) {
        ttl = (ttl + data[x])%1000;
}
```

The loop that was auto-vectorized looked like this:

```c
for (x = 0; x < SAMPLES; x++) {
        data[x] = scale_sample(data[x], 0.75);
}
```

We are now going to take a look at a different implementation of the above loop by using manual vectorization.

## Inline assembly

One way to force vectorization is by doing it manually. Inline assembly allows us to do this relatively easy within our C code. An important thing to keep in mind is that inline assembly is architecture dependant and thus not very portable.

When we use inline assembly we have the option to reserve some registers for our variables, this will force the compiler to use those specific registers instead of assigning them at compile time. We can do it by declaring a variable like this:

```c
register int16_t*       cursor          asm("r20");
```

Now when we refer to cursor in our inline assembly it will mean r20 specifically.

If you remember from the previous post on [Scaling Audio Through Software] we need a multiplier that represents the scaling of the sample. We are going to use a fixed-point value since it's faster than floating-point.

To do so we multiply our scaling factor (0.75) by 32767. Since we are using 16-bit integers we have to make sure not to overflow so we use the upper bound of signed int16 which is 32767.

```c
vol_int = (int16_t) (0.75 * 32767.0);
```

Because we want to use vector registers we have to duplicate vol_int into each element of a vector register. We can accomplish this by doing the following:

```c
__asm__ ("dup v1.8h,%w0"::"r"(vol_int));
```

The loop that goes through each sample and scales it by our vol_int factor looks like this:

```c
while(cursor < limit){
    __asm__ (
        "ldr q0, [%[cursor]], #0        \n\t"

        "sqdmulh v0.8h, v0.8h, v1.8h    \n\t"

        "str q0, [%[cursor]],#16                \n\t"

        : [cursor]"+r"(cursor)
        : "r"(cursor)
        : "memory"
        );
}
```

Notice that when we load register q0, we set the offset to 0. Because cursor is 16-bit wide, this will then fill q0 with 8 copies of cursor. On the other hand when we store q0 to memory we must specify the offset `#16` otherwise the whole register would be stored as a single value and not 4x 16-bit values.

`+r` indicates the register may be used for reading and writing. `[cursor]` is used to indicate that the register may be called `%cursor`. The instruction `"r"(cursor)` specifies where we wish to output the result of the instruction. We add the `"memory"` clobber because our instructions may overwrite the values stored in memory, with this we tell the compiler not to trust previously memory loaded values and to reload them after our assembly code is executed.

## Intrinsics

We can also write the previous loop using intrinsics. Intrinsic functions are compiler specific functions that will usually be inserted inline and produce highly efficient machine instructions.

While intrinsics may be more efficient and easier to use than inline assembly, they still have a huge impact in portability.

Our loop using intrinsics looks like this:

```c
while ( cursor < limit ) {
    vst1q_s16(cursor, vqdmulhq_s16(vld1q_s16(cursor), vdupq_n_s16(vol_int)));

    cursor += 8;
}
```

There's a lot happening in that single line of code so lets go over it from the innermost function.

```c
vdupq_n_s16(vol_int)
```

This function duplicates the value of vol_int into each element of a vector register.

```c
vld1q_s16(cursor)
```

This loads a vector register with the values in memory starting at `cursor`.

```c
vqdmulhq_s16(vld1q_s16(cursor), vdupq_n_s16(vol_int))
```

This instruction multiplies the corresponding values in the "cursor" register with the values in the "vol_int" register. Doubles the results and then places the significant half of the results into a vector register.

```c
vst1q_s16(cursor, vqdmulhq_s16(vld1q_s16(cursor), vdupq_n_s16(vol_int)));
```

Finally, this instruction will store each of the elements returned by the previous `vqmulhq` instruction to where `cursor` is pointing.

After each operation we will need to move `cursor` by 8, the reason behind this number is because we are processing and storing the values of two vector registers each of which contains 4 values.

[Scaling Audio Through Software]: {{site.baseurl}}{% post_url 2019-10-02-Scaling-Audio-Through-Software %}
[Lab 5]: https://wiki.cdot.senecacollege.ca/wiki/SPO600_SIMD_Lab
