---
layout: post
title: Software Profilling
date: 2019-11-08
---

## gprof

Needs special compile:
Needs -pgcompiler option

CFLAGS = "-g -pg -O2" CXXFLAGS="-g -pg -O2"

Uses instrumentation for call graph + sampling for timing

Thanks to instrumentation it is able to see every single function call. Both slow down the program.

Run the program

gprof program | gprof2dot | dot -T x11

## perf

No need for compiler options, -g is recommended
No instrumentation, only sampling

Sampling uses software interrupts which may miss inline and small functions, because it only checks every ...snds of a second.

perf report ..| less

## dd

dd of=100.txt bs=1M count=100 iflags=fullblock

## time

/usr/bin/time -> external timing tool

## gprof-tools
