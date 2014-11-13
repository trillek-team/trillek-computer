```
Trillek DRAFT                                        Luis P. G., Author
draft-rfc-tr1.md                    
                                             
Category: Standards Track                               5 November 2014
```

Calling conventions proposal for TR3200 CPU
===========================================

# Status of This Memo

This document specifies calling conventions for the TR3200 CPU assembly 
, and requests discussion and suggestions for improvements. 
Distribution of this document is unlimited.

# Abstract

A calling convention is an implementation level (low level) scheme for 
how subroutines receive parameters from their caller and how they 
return a result. Also defines, what registers are preserved or not, and
where is done.


# Table of Contents

1. Introduction
2. Notation
3. CDECL convention
4. FastCall convention


# 1 Introduction

In computer science, a calling convention is an implementation level 
(low level) scheme for how subroutines receive parameters from their 
caller and how they return a result. This is somewhat related to a 
particular programming language's evaluation strategy but most often 
not considered part of it (or vice versa) as the latter is considered  
part of the language rather than the compiler and largely defined on a 
higher abstraction level.

Calling conventions can differ in:

 * where parameters and return values are placed (in registers, on the 
   call stack, a mix of both, or in other memory structures)
 * The order in which actual arguments for formal parameters are passed
   (or the parts of a large or complex argument)
 * How a (possibly long or complex) return value is delivered from the 
   callee back to the caller (on the stack, in a register, or within 
   the heap)
 * How the task of setting up for and cleaning up after a function call
   is divided between the caller and the callee

In some cases, differences also include the following:

* Conventions on which registers may be directly used by the callee, 
  without being preserved (otherwise regarded as an ABI detail)
* Which registers are considered to be volatile and, if volatile, need
  not be restored by the callee (often regarded as an ABI detail)

Although some languages actually specify (parts of) this in the 
programming language specification (or in some pivotal implementation),
different implementations of such languages (i.e. different compilers) 
may typically still use various calling conventions, often selectable. 
One reason for this is performance, another is a frequent adaption to 
other popular languages conventions (with or without technical reason), 
and a third is that "platforms" (CPU architecture + operating system 
combinations) also may impose restrictions or conventions.

This must be considered when combining modules written in multiple 
languages, or when calling operating system or library APIs from a 
language other than the one in which they are written; in these cases, 
special care must be taken to coordinate the calling conventions used 
by caller and callee. Even a program using a single programming 
language may use multiple calling conventions, either chosen by the 
compiler, for code optimization, or specified by the programmer.

CPU architectures always have more than one possible calling 
convention. With many general-purpose registers and other features, 
the potential number of calling conventions is large, although some 
architectures are formally specified to use only one calling 
convention, supplied by the architect.

(Source Wikipedia)

# 2 Notation

The key words **MUST**, **MUST NOT**, **REQUIRED**, **SHALL**, 
**SHALL NOT**, **SHOULD**, **SHOULD NOT**, **RECOMMENDED*, **MAY**, 
and **OPTIONAL** in this document are to be interpreted as described
in [RFC2119].

The key word **CALLER** is code that is making the call to the 
subroutine, the key word **CALLEE** is the subroutine being called.

# 3 CDECL convention

In **CDECL**, subroutine arguments are passed using the stack. 
Integer values and memory addresses are returned in the %r0 register. 
Register %r0 may be saved by **caller** as is used to return values. 
Any other register used by the **calle** must be saved by the calle.

Function arguments are passed on the stack, in right-to-left order. For
example, in ```void foo(1, 2, 3)```, results in this sequence : 
```
    PUSH 3
    PUSH 2
    PUSH 1
```
Also, the arguments are popped from the stack by the **caller** itself.
As the stack of the TR3200 can only push/pop 32-bit values, 8-bit and 
16-bit integer arguments are promoted to 32-bit arguments. 64-bit or 
more wider data, must be pushed to the stack on little-endian.

Result is stored on %r0 at end of the **calle** code.
(*TODO* Return 32-bit more wide data, and structs)

The **calle** must exit using ```RET``` instruction.

Function/subroutine name should be pre-appended with an underscore. 
For example ```foo()``` -> ```__foo```

On the **calle** code, the first thing that must do is push %bp 
register to the stack, and then, must copy the value of %sp to %bp.
With this, to access any argument, the **calle** code must do : 
```
 argument[N] = load [%bp + 4 + (N*4)]
```
Where **N** is the argument number counting from the left to the right 
and the first argument  is N = 1.

Next, the **calle** code, must allocate local variables on stack space.
To allocate, the %sp register must be decremented by 4 for every local 
variable of 8-bit, 16-bit or 32-bit wide. 64-bit wide variable, must 
decrement %sp by 8. For example, a function that have 3 local 
variables, one of 8-bit wide and two of 16-bit wide, must decrement 
%sp by 12.

The third step, is push on the stack, any register used by the 
**calle** code, except %r0.

The result stack of this steps would be : 
```
    |                |
    |                |
    |----------------|
    |  Input param N |  <-- %bp + 4 +4\*N
    |----------------|
    |   ..........   |
    |----------------|
    |  Input param 3 |  <-- %bp + 4 +4\*3
    |----------------|
    |  Input param 2 |  <-- %bp + 4 +4\*2
    |----------------|
    |  Input param 1 |  <-- %bp + 4 +4\*1
    |----------------|
    | Return address |
    |----------------|
    |  %bp old value |  <-- %bp points here
    |----------------|
    |                |  <-- %bp - 4*X to access it
    |     Local      |
    |   Variables    |
    |                |
    |----------------|
    |                |
    |     Pushed     |
    |   Registers    |
    |                |  <-- %sp points here
    |----------------|
```

When the **calle** code ends, the **calle** must : 

1. Pop registers from the stack, to restore his original value.
2. Set %sp to %bp value. This frees the allocate stack space used for 
   local vars.
3. Execute ```RET``` to return to the **caller**

When the **caller** continues the execution, must restore the previous
stack pile status. This is could be done POPing the arguments from the 
stack, but a simple ```ADD %sp, %sp, N\*4```, were **N**is the number of
arguments would do the job more faster and with less code.

# 4 FastCall convention
*TODO*

It's like CDECL convention, but uses %r0 to %r4 to store the first five
arguments, counting from left to the right, being %r0 to %r4 not 
preserved by the **calle**. In this way, uses less stack for arguments, 
and makes calls faster, as not need to push to the stack.

