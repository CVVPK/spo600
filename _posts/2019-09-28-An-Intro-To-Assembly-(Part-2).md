---
layout: post
title: An Intro To Assembly (Part 2)
date: 2019-09-28
categories: Lab
---

After learning how to print something to the CL in assembly we are now ready to learn how to write a for loop. As we saw in [Part 1][part-1] writing simple programs in assembly is a lot more complex than writing them in C. However, the resulting program is much smaller and faster to execute than compiled C.

This is Part 2 of the series Intro to Assembly. It continues from where we left off in [Part 1][part-1]. It is based on the work done during [Lab 3][lab-3].

- [Writing A Simple Loop](#writing-a-simple-loop)
- [Adding a loop number to the message](#adding-a-loop-number-to-the-message)
- [Expanding the loop to double digit iterations](#expanding-the-loop-to-double-digit-iterations)
- [Suppress the leading 0](#suppress-the-leading-0)
- [Conclusions](#conclusions)


<!-- more -->
## Writing A Simple Loop

First we are going to see what it takes to create a loop. From [Part 1][part-1] we already know how to print a message to the screen. Now we would like to print a message multiple times. If you have written a for loop on almost any other program, you know we need a condition so that we know when to stop the loop.

We begin by defining two macros, `start` will serve as the initial value of the iterator and `max` will be the maximum number of times we want to loop:

```as
start = 0
max = 10
```

We need a way to keep track of the iteration we are in, we are going to use `r15` to do so. So to initiate our iterator we move `start` to `r15`:

```as
mov $start, %r15
```
We will have to then increment the value stored in `r15` after each loop, just before doing checking our condition. To increment the value in the register we simply write:

```as
inc %r15
```


Now that we have an iterator that can increment on each loop, a macro that defines the maximum number of loops we want; all we're missing is a way to check that the iterator does not exceed the maximum value. So, to compare it we write:

```as
cmp $max, %r15
```

This will set a flag in the processor status register depending on whether they are equal or not. We can then use this flag to jump to a label, in our case we want to jump back the start of the `loop` label if the condition is false, we can do this with the following line:

```as
jne loop
```

All we're missing now is a message to print to the screen. We do the same thing we did in our Hello World! program to print something to the CL:

```as
mov	$len,%rdx
mov	$msg,%rsi
mov	$stdout,%rdi
```

Where `msg` and len are declared below:

```as
.data
msg:	.ascii	"Loop\n"
.set	len , . - msg
```

The finished code should look something like this in x86_64: 

```as
.text
.globl	_start

stdout = 1

start = 0
max = 10

_start:
	mov	$start,%r15

loop:
	mov	$len,%rdx
	mov	$msg,%rsi
	mov	$stdout,%rdi
	
	mov	$1,%rax
	syscall

	inc	%r15
	cmp	$max,%r15
	jne	loop

	mov	$0,%rdi
	mov	$60,%rax
	syscall

.data
msg:	.ascii	"Loop\n"
.set	len , . - msg
```

The code in AArch64 looks very similar, but there are a couple of important difference to note.

We will use the same macros as before to initialize our iterator and define the limit. Something to keep in mind when working with AArch64 is the order of the operations are different to x86_64. The order in which we declare an operation in x86_64 goes: `instruction value,register` while on AArch it is `instruction register,value`. A move instruction in x86_64 looks something like this:

```as
mov $1,%r15
```

In AArch64 it would be:

```as
mov w19,1
```

Notice the lack of `$` and `%`. Another peculiarity is the `w` before the register number. In AArch64 apart from having more registers available to use than x86_64 we can also determine whether we wish to use a 32-bit register by writing `w` before the register number or a 64-bit register by using `x`.

The logic of the program is the same as what we used for x86_64, we need an iterator to keep track of our loops, a condition and of course a message to print out.

The finished code in AArch 64 is below:

```as
.text
.globl	_start

stdout = 1

start = 0
max = 10

_start:
	mov	w19,start	/* Iterator */

loop:
	/* set up message */
	mov	x0, stdout
	adr	x1,msg
	mov	x2,len

	mov	x8,64		/* write message */
	svc	0
	
	add	w19,w19,1	/* Increment Iterator */
	cmp	w19,max		/* Check Iterator < max */
	b.ne	loop		/* Repeat if cmp ==false */

	mov	x0,0
	mov	x8,93
	svc	0

.data
msg:	.ascii	"Loop\n"
len = . - msg	
```

## Adding a loop number to the message

Now that we have got the basics of how to write a simple loop, we are going to make it a bit more fun. We will add the current iterator number to the message, so that it looks something like this `Loop: n` where 'n' is the number of the iteration.

In order to print something to the CL we use a string, `msg`. We have to remember that this string is a series of 8 bit values, and each value represents an [ASCII](https://en.wikipedia.org/wiki/ASCII) character. With this in mind, next thing to note is that 0 is not the same as "0". The first 0 represents a value, but "0" is an ASCII character. When trying to print a number to the CL we have to keep in mind the ASCII codes for numbers typically begin at 48. So, 48="0", 49="1"...57="9". So we are going to need to add 48 to whatever our iterator value is to get the correct ASCII value.

In this version of the program we are simply going to use a second register to keep track of the ASCII value of the iterator. However, this is only possible because the maximum value we want to display is "9". In a later version, when we display more than a single digit, we will see that this is probably not the best approach.

Inserting the digit inside our pre-existing string is not very complicated. Since we know where this string begins and ends in memory, we can just add an offset to our digit so that it is placed in the position we wish to display it.

Following the previous ideas, the code will look something like this: 

```as
.text
.globl	_start

stdout = 1

start = 0
max = 10

_start:
	mov	$start,%r15
	mov	$48, %r14	/* ASCII iterator */

loop:
	mov	$len,%rdx
	mov	$msg,%rsi
	movb %r14b,offset	/* Digit inserted with an offset */
	mov	$stdout,%rdi
	
	mov	$1,%rax
	syscall

	inc	%r15	/* Increase iterator by 1 */
	inc	%r14	/* Increase ASCII digit by 1 */

	cmp	$max,%r15	/* Loop while iterator != max */
	jne	loop

	mov	$0,%rdi
	mov	$60,%rax
	syscall

.data
msg:	.ascii	"Loop:  \n"
.set	len , . - msg
.set	offset, msg + 6		/* Digit's position */
```

Applying the same logic in AArch64 gives us the following:

```as
.text
.globl	_start

stdout = 1

start = 0
max = 10

_start:
	mov	w19,start	/* Iterator */
	mov	w20, 48
loop:

	/* set up message */
	mov	x0, 1
	adr	x1,msg
	mov	x2,len
	strb	w20,[x1,offset]	/* Place ASCII digit into offset */

	mov	x8,64		/* write message */
	svc	0
	
	add	w19,w19,1	/* Increment Iterator */
	add	w20, w19,48	/* Convert iterator to ASCII value and add 48 */
	cmp	w19,max		/* Check Iterator < max */
	b.ne	loop		/* Repeat if cmp ==false */

	mov	x0,0
	mov	x8,93
	svc	0

.data
msg:	.ascii	"Loop:  \n"
len = . - msg	
offset = msg + 6

```

## Expanding the loop to double digit iterations

What about making our loop go higher than 9? you may ask. Well, that presents us with a couple of extra challenges. As mentioned before our approach of using a register to represent the ASCII value of the iterator is not very good. The reason for that is that it can keep track of "0" -"9" but then we would have to keep on resetting it back in addition to needing a register for each digit and logic to keep them in sync. 

A simpler way of doing it is by using simple division. If you have programmed before you'll probably be familiar with the modulus operator (`%`). Whether you hate it or love it, getting the remainder of a number is the easiest way of breaking it up into its individual digits.

The way we do division in x86_64 and AArch64 is pretty different. x86_64 gives us the option to immediately access the remainder of the operation, while on AArch64 it is up to us to calculate it.

In order to perform the division in x86_64 we first need to place the value we wish to divide into `rax`. After that is done we can simply use the `div` instruction and specify the number we want `rax` to be divided by. The remainder will be put into `rdx`. An important thing to keep in mind is that `rdx` needs to be zero'd out before the division happens. Here's what this looks like:

```as
mov	%r15, %rax
mov	$0,%rdx
div	%r12
```

We can from there move `rdx` into a safer register and then add 48 to it to represent the second digit (from left to right) of a decimal number. The result of our division, which represents the first digit on a two digit decimal, gets put into `rax` so we can easily move it to a safer register and add 48.

```as
mov %rdx,%r14	/* second digit binary */
mov	%rax,%r13	/* first digit binary */
add	$48,%r13	/* convert 1st digit to ascii */
add	$48,%r14	/* convert 2nd digit to ascii */
```

After this is done is just a matter of once again placing the ASCII digits where they should be by using an offset and then printing to the screen.

The whole program will look something like this:

```as
.text
.globl	_start

stdout = 1

start = 0
max = 30

_start:
	mov	$start,%r15	/* binary iterator start */
	mov	$10,%r12	/* divisor */
loop:
	mov	%r15, %rax
	mov	$0,%rdx
	div	%r12

	mov 	%rdx,%r14	/* second digit binary */
	mov	%rax,%r13	/* first digit binary */
	add	$48,%r13	/* convert 1st digit to ascii */
	add	$48,%r14	/* convert 2nd digit to ascii */
	
	mov	$len,%rdx
	mov	$msg,%rsi	/* message */
	movb	%r13b,offset1	/* first digit */
	movb	%r14b,offset2	/* second digit */
	mov	$stdout,%rdi
	
	mov	$1,%rax
	syscall

	inc	%r15		/* Increment Iterator */
	cmp	$max,%r15	/* Check Iterator < max */
	jne	loop			
	
	mov	$0,%rdi
	mov	$60,%rax
	syscall

.data
msg:	.ascii	"Loop:   \n"
.set	len , . - msg
.set	offset1, msg + 6
.set	offset2, msg + 7

```

As I said before in AArch64 the division operation is a little different. So to accomplish the same we need to do two different instructions, the second of which provides us the remainder. A division instruction goes like this:

```as
udiv	w20, w19, w28
```

This will put into `w20` the result of dividing `w19` by `w28`. We must have previously moved the corresponding values into `w19` and `w28`. 

To calculate the remainder we need to do the following:

```as
msub	w21,w20,w28,w19
```

This operation is going to place the remainder into `w21` by performing this: `w19-(w20*w28)`. Note that we do this with the same values in `w19` and `w28` that we used for our division and it must be done after `w20` has been loaded with the result of the division.

The final solution in AArch64 is below:

```as
.text
.globl	_start

stdout = 1

start = 0
max = 30

_start:
	mov	w19,start	/* Iterator */
	mov	w28,10

loop:
	udiv	w20, w19, w28	/* calculate first digit 0#*/
	msub	w21,w20,w28,w19	/* calculate second digit 0# */

	add	w20,w20,48	/* convert first digit to ascii */
	add	w21,w21,48	/* convert second digit to ascii */

	/* set up message */
	mov	x0, 1
	adr	x1,msg
	mov	x2,len
	cmp	w20,48
	strb	w20,[x1,offset1]
	strb	w21,[x1,offset2]

	mov	x8,64		/* write message */
	svc	0
	
	add	w19,w19,1	/* Increment Iterator */
	cmp	w19,max		/* Check Iterator < max */
	b.ne	loop		/* Repeat if cmp ==false */

	mov	x0,0
	mov	x8,93
	svc	0

.data
msg:	.ascii	"Loop:   \n"
len = . - msg	
offset1 = msg + 6
offset2 = msg + 7

```

## Suppress the leading 0

If you tried running the previous program you'll notice that loops 0-9 are printed with two digits like `00` and `01`. Getting rid of this 0 is not as complicated as it might sound.

We know about labels, conditions and how to jump to a label based on the result of a condition. So to suppress the leading 0 we just have to add a condition so that our program jumps to a label before placing the first digit inside our string. We can write it like this:

```as
cmp	$48,%r13
je	nozero	/* Jump to label if r13 is "0" */

movb	%r13b,offset1	/* first digit */

nozero:
/* Rest of instructions */
```

In x86_64 the final program will look somewhat like the following:

```as
.text
.globl	_start

stdout = 1

start = 0
max = 30

_start:
	mov	$start,%r15	/* binary iterator start */
	mov	$10,%r12	/* divisor */
loop:
	mov	%r15, %rax
	mov	$0,%rdx
	div	%r12

	mov 	%rdx,%r14	/* second digit binary */
	mov	%rax,%r13	/* first digit binary */
	add	$48,%r13	/* convert 1st digit to ascii */
	add	$48,%r14	/* convert 2nd digit to ascii */
	
	mov	$len,%rdx
	mov	$msg,%rsi	/* message */
	cmp	$48,%r13
	je	nozero
	movb	%r13b,offset1	/* first digit */
nozero:
	movb	%r14b,offset2	/* second digit */
	mov	$stdout,%rdi
	
	mov	$1,%rax
	syscall

	inc	%r15		/* Increment Iterator */
	cmp	$max,%r15	/* Check Iterator < max */
	jne	loop			
	
	mov	$0,%rdi
	mov	$60,%rax
	syscall	
.data
msg:	.ascii	"Loop:   \n"
.set	len , . - msg
.set	offset1, msg + 6
.set	offset2, msg + 7

```

Doing the same thing in AArch64 looks like this:

```as
.text
.globl	_start

stdout = 1

start = 0
max = 30

_start:
	mov	w19,start	/* Iterator */
	mov	w28,10

loop:
	udiv	w20, w19, w28	/* calculate first digit -0*/
	msub	w21,w20,w28,w19	/* calculate second digit 0- */

	add	w20,w20,48	/* convert first digit to ascii */
	add	w21,w21,48	/* convert second digit to ascii */

	/* set up message */
	mov	x0, 1
	adr	x1,msg
	mov	x2,len
	cmp	w20,48
	b.eq	noleadzero
	strb	w20,[x1,offset1]

noleadzero:

	strb	w21,[x1,offset2]
	mov	x8,64		/* write message */
	svc	0
	
	add	w19,w19,1	/* Increment Iterator */
	cmp	w19,max		/* Check Iterator < max */
	b.ne	loop		/* Repeat if cmp ==false */

	mov	x0,0
	mov	x8,93
	svc	0

.data
msg:	.ascii	"Loop:   \n"
len = . - msg	
offset1 = msg + 6
offset2 = msg + 7
```

## Conclusions

Writing a program in assembly that accomplishes the same as a for loop is a lot more complex than I imagined. As programing languages become more abstract and accessible we start taking a lot of things for granted. It is great to see how much goes into performing a task that would usually look like `for(i=0; i<n; i++){};`.

The major difficulty when writing in assembly, at least that I have encountered, was finding readable documentation. I suppose since it is such a low level language and not really intended to be written by anyone but a very small number of programers, deciphering what an operation does from the official manuals proved to be very difficult. 

Although writing a simple program in assembly is such a tedious task, I had fun doing it, and hope you had as well. I'll probably try to make some other things in assembly and who knows I might post them here later or just burn my computer if things don't work as planned. 

Assembly might not be most people's cup of tea, but I think that knowing the basics of it is a great way to get an idea of what goes under the hood of most operations we write in higher level languages.

[part-1]: {{site.baseurl}}{% post_url 2019-09-27-An-Intro-To-Assembly-(Part-1) %}
[lab-3]: https://wiki.cdot.senecacollege.ca/wiki/SPO600_Assembler_Lab
