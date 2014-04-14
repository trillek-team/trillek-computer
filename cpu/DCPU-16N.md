---
layout : default
title : DCPU-16N CPU
cat : CPU
---

# DCPU-16N Specification
#### Copyrights 1985 Mojang, Meisaka of Trillek
#### Version 0.8 based on DCPU-16 (1.7)


SUMMARY
===========================================================================

* 16 bit CISC CPU design
* 8 bit memory words (1 octet per address)
* 16 bit memory data bus transfers two octets per clock
* 16 bit virtual addresses accessing 65536 octets of RAM
* 16 bit I/O address bus
* 24 bit physical addresses via integrated Extended Memory Unit (EMU)
  up to 16 million octets are addressable.
* 8 registers 16 bits wide (**A**, **B**, **C**, **X**, **Y**, **Z**, **I**, **J**)
* program counter (**PC**) - 16 bit
* stack pointer (**SP**) - 16 bit
* 32 bit high speed multiplication unit.
* 32 bit high speed division unit (only 8 cycles per operation!).
* 16 bit barrel shifter
* ALU high word - extra/excess (**EX**) - 16 bit
* interrupt address (**IA**) - 16 bit
* transparent interrupt queue with 16 bit messages

In this document, anything within [brackets] is shorthand for "the value of the
RAM at the location of the value inside the brackets". For example, **SP** means
stack pointer, but [**SP**] means the value of the RAM at the location the stack
pointer is pointing at.

Whenever the CPU needs to read a word, it reads two octets from [**PC**+1]:[**PC**], 
then increases **PC** by two octets. Shorthand for this is [**PC**++].
In some cases, the CPU will modify a value before reading it,
in this case the shorthand is [++**PC**].
When **PC** is an odd value, each instruction will take an extra cycle to read.

For stability and to reduce bugs, it's strongly suggested all multi-word
or multi-octet operations use little endian in all DCPU-16N programs,
wherever possible.

For stability and to prevent excess heat dissipation, it is recommended to
clock the DCPU-16N at no more than 1000 kHz. For best stability in deep space
or high radiation environments, a clock rate of 200kHz is recommended, as well
as core memory or full ECC memory.



INSTRUCTIONS
================================================================================

Instructions are 1-3 words long and length is fully defined by the first word.
In a basic instruction, the lower five bits of the first word of the instruction
are the opcode, and the remaining eleven bits are split into a five bit value b
and a six bit value a.
b is always handled by the processor after a, and is the lower five bits.
In bits (in LSB-0 format), a basic instruction has the format: `aaaaaabbbbbooooo`

Some instructions will skip or slightly alter the behavior of the
next instruction(s), but length is still fully defined by the first word.

In the tables below, C is the time required in cycles to look up the value, or
perform the opcode, VALUE is the numerical value, NAME is the mnemonic, and
DESCRIPTION is a short text that describes the opcode or value.


### Values: (5/6 bits)
 C | VALUE     | DESCRIPTION
:-:|----------:|----------------------------------------------------------------
 0 | 0x00-0x07 | register (**A**, **B**, **C**, **X**, **Y**, **Z**, **I**, or **J**, in that order)
 1 | 0x08-0x0f | [register]
 2 | 0x10-0x17 | [register + *next word*]
 1 |      0x18 | (PUSH / [--**SP**]) if in b, or (POP / [**SP**++]) if in a
 1 |      0x19 | [**SP**] / PEEK
 2 |      0x1a | [**SP** + next word] / PICK n
 0 |      0x1b | **SP**
 0 |      0x1c | **PC**
 0 |      0x1d | **EX**
 2 |      0x1e | [*next word*]
 1 |      0x1f | *next word* (literal)
 0 | 0x20-0x3f | literal value 0xffff-0x1e (-1..30) (literal) (only for a)


* "next word" means "[**PC**++]". Increases the word length of the
  instruction by 1 (two octets).

* By using 0x18, 0x19, 0x1a as PEEK, POP/PUSH, and PICK there's a reverse stack
  starting at memory location 0xffff. Example: `SET PUSH, 10`, `SET X, POP`

* Attempting to write to a literal value fails silently

* Make note of the PUSH operation, some instructions that modify the b value,
  in that case remember that the CPU will change the register before reading
  the value, and then write to that location the result (as --**SP** implies).
  


### Basic opcodes (5 bits)
 C | VAL  | NAME     | DESCRIPTION
:-:|------|----------|---------------------------------------------------------
 - | 0x00 | n/a      | special instruction - see below
 1 | 0x01 |`SET b, a`| sets b to a
 2 | 0x02 |`ADD b, a`| sets b to `b + a`, sets **EX** to 0x0001 if there's an overflow, 0x0 otherwise
 2 | 0x03 |`SUB b, a`| sets b to `b - a`, sets **EX** to 0xffff if there's an underflow, 0x0 otherwise
 3 | 0x04 |`MUL b, a`| sets b to `b * a`, sets **EX** to `((b * a) >> 16) & 0xffff` (treats b, a as unsigned)
 4 | 0x05 |`MLI b, a`| like MUL, but treat b, a as signed
 9 | 0x06 |`DIV b, a`| sets b to `b / a`, sets **EX** to `((b << 16) / a) & 0xffff`. if `a == 0`, sets b and **EX** to 0 instead. (treats b, a as unsigned)
 10| 0x07 |`DVI b, a`| like DIV, but treat b, a as signed. Rounds towards 0
 6 | 0x08 |`MOD b, a`| sets b to `b MODULUS a`. if `a == 0`, sets b to 0 instead.
 7 | 0x09 |`MDI b, a`| like MOD, but treat b, a as signed. (MDI -7, 16 == -7)
 1 | 0x0A |`AND b, a`| sets b to `b AND a`
 1 | 0x0B |`BOR b, a`| sets b to `b OR a`
 1 | 0x0C |`XOR b, a`| sets b to `b XOR a`
 1 | 0x0D |`SHR b, a`| sets b to `b >>> a`, sets **EX** to `((b<<16) >> a) & 0xffff` (logical shift)
 1 | 0x0E |`ASR b, a`| sets b to `b >> a`, sets **EX** to `((b<<16) >>> a) & 0xffff` (arithmetic shift) (treats b as signed)
 1 | 0x0F |`SHL b, a`| sets b to `b << a`, sets **EX** to `((b<<a) >> 16) & 0xffff`
 2+| 0x10 |`IFB b, a`| performs next instruction only if `(b & a) != 0`
 2+| 0x11 |`IFC b, a`| performs next instruction only if `(b & a) == 0`
 2+| 0x12 |`IFE b, a`| performs next instruction only if `b == a`
 2+| 0x13 |`IFN b, a`| performs next instruction only if `b != a`
 2+| 0x14 |`IFG b, a`| performs next instruction only if `b > a`
 2+| 0x15 |`IFA b, a`| performs next instruction only if `b > a` (signed)
 2+| 0x16 |`IFL b, a`| performs next instruction only if `b < a`
 2+| 0x17 |`IFU b, a`| performs next instruction only if `b < a` (signed)
 - | 0x18 | -        |
 - | 0x19 | -        |
 3 | 0x1A |`ADX b, a`| sets b to b+a+**EX**, sets **EX** to 0x0001 if there is an over-flow, 0x0 otherwise
 3 | 0x1B |`SBX b, a`| sets b to b-a+**EX**, sets **EX** to 0xFFFF if there is an under-flow, 0x0001 if there's an overflow, 0x0 otherwise
 3 | 0x1C |`HWW b, a`| writes a to I/O bus address b
 3 | 0x1D |`HWR b, a`| reads a from I/O bus address b
 2 | 0x1E |`STI b, a`| sets b to a, then increases **I** and **J** by 1
 2 | 0x1F |`STD b, a`| sets b to a, then decreases **I** and **J** by 1


* The conditional opcodes take one cycle longer to perform if the test fails.
  When they skip a conditional instruction, they will skip an additional
  instruction at the cost of one extra cycle. This continues until a non-
  conditional instruction has been skipped. This lets you easily chain
  conditionals. Interrupts are not triggered while the DCPU-16N is skipping.

* Signed numbers are represented using two's complement.

* Instructions with extra output in **EX**, put the extra value in **EX** before
  writing to b. Using **EX** as the b operand on these instructions is not
  is recommended for stability and to reduce bugs.

* Division instructions (DIV DVI MOD MDI) tend to dissipate significantly more
  power than other instructions in practice, ensure proper thermal management
  before using them excessively to ensure stability and to reduce bugs.


## Special opcodes

Special opcodes always have their lower five bits unset, have one value and a
five bit opcode. In binary, they have the format: `aaaaaaooooo00000`
The value (a) is in the same six bit format as defined earlier.

### Special opcodes: (5 bits)

 C | VAL  | NAME  | DESCRIPTION
:-:|------|-------|-------------------------------------------------------------
 - | 0x00 | n/a   | compact instruction - see below
 3 | 0x01 |`JSR a`| pushes the address of the next instruction to the stack, then sets **PC** to a
 4 | 0x02 |`BSR a`| pushes the address of the next instruction to the stack, then adds a to **PC**
 - | 0x03 | -     |
 - | 0x04 | -     |
 1 | 0x05 |`NEG a`| sets a to its two's complement negation.
 - | 0x06 | -     |
421| 0x07 |`HCF a`| Reinitialize the oscillator and store power in a, used for internal testing of the CPU, should not be used. (typical result is CPU catching fire)
 4 | 0x08 |`INT a`| triggers a software interrupt with message a
 1 | 0x09 |`IAG a`| sets a to **IA**
 1 | 0x0A |`IAS a`| sets **IA** to a
 3 | 0x0B |`RFI a`| disables interrupt queueing, pops **A** from the stack, then pops **PC** from the stack
 2 | 0x0C |`IAQ a`| if a is nonzero, interrupts will be added to the queue instead of triggered. if a is zero, interrupts will be triggered as normal again
 - | 0x0D | -     |
 4 | 0x0E |`MMW a`| treats a as two values, in binary: `ppppppppppppssss` writes those values to the EMU, changing one memory page mapping. sets block s to page p (see "Extended Memory" below)
 4 | 0x0F |`MMR a`| treats a as two values, in binary: `ppppppppppppssss` reads block s from the EMU, and sets p to active page (see "Extended Memory" below)
 - | 0x10 | -     |
 - | 0x11 | -     |
 - | 0x12 | -     |
 - | 0x13 | -     |
 1 | 0x14 |`SXB a`| sign extend byte, sets all bits in high octet of a to the MSB of the low octet.
 2 | 0x15 |`SWP a`| swap the high and low octets in a
 - | 0x16 | -     |
 - | 0x17 | -     |
 - | 0x18 | -     |
 - | 0x19 | -     |
 - | 0x1A | -     |
 - | 0x1B | -     |
 - | 0x1C | -     |
 - | 0x1D | -     |
 - | 0x1E | -     |
 - | 0x1F | -     |



## Implied opcodes

Implied opcodes always have their lower ten bits unset, have a bit value and a
five bit opcode. In binary, they have the format: `vooooo0000000000`
The value(v) is a bit flag used by some opcodes, and is ignored otherwise.


### Implied opcodes: (5 bits)

 C | VAL  | NAME  | DESCRIPTION
:-:|------|-------|-------------------------------------------------------------
 4 | 0x00 |`HLT`  | if interrupts are enabled, generates an interrupt with message 0, otherwise halts CPU operation.
 3 | 0x01 |`SLP`  | halts CPU operation, and puts the CPU in a low power state if interrupts are enabled, then the DCPU-16N will resume operation when the next interrupt is triggered.
 - | 0x02 | -     |
 - | 0x03 | -     |
 1 | 0x04 |`BYT v`| Prevent writes to a byte in the output word of the next instruction, prevents write to the high byte when v is 1, prevents write to the low byte when v is 0
 - | 0x05 | -     |
 - | 0x06 | -     |
 - | 0x07 | -     |
 - | 0x08 | -     |
 - | 0x09 | -     |
 - | 0x0A | -     |
 - | 0x0B | -     |
 - | 0x0C | -     |
 - | 0x0D | -     |
 - | 0x0E | -     |
 - | 0x0F | -     |
 2 | 0x10 |`SKP`  | Unconditionally skip next instruction
 - | 0x11 | -     |
 - | 0x12 | -     |
 - | 0x13 | -     |
 - | 0x14 | -     |
 - | 0x15 | -     |
 - | 0x16 | -     |
 - | 0x17 | -     |
 - | 0x18 | -     |
 - | 0x19 | -     |
 - | 0x1A | -     |
 - | 0x1B | -     |
 - | 0x1C | -     |
 - | 0x1D | -     |
 - | 0x1E | -     |
 - | 0x1F | -     |


* The BYT opcode prevents writing a byte of the next instruction, conditional
  instructions are ignored, applying to the next non-conditional instruction, 
  allowing conditionally masking byte writes. BYT applies to only one
  instruction, similar to how conditionals work, that instruction may be skipped

* Interrupts are queued when BYT is in effect, as there is no way to maintain
  its state between interrupts.

* Executing two BYT instructions in a row, turns on the byte masking, then
  turns it off again, effectively a two word no op.

* BYT only prevents writes to the direct output of operations, reads are still
  handled as normal 16 bit. A BYT operation used on a SWP operation will
  duplicate bytes. Indirect writes (to **EX** for example) are not affected.

* Internally the DCPU-16N keeps track whether it is skipping, queuing interrupts
  , masking byte writes (of which byte), and how many interrupts are queued.
  All of these flags and values are transparent from the programming model, so
  user of the DCPU-16N need not concern themselves with the implementation.


INTERRUPTS
=================================================================    

The DCPU-16N will perform at most one interrupt between each instruction. If
multiple interrupts are triggered at the same time, they are added to a queue.
If the queue grows longer than 256 interrupts, the DCPU-16N will catch fire. 

When **IA** is set to something other than 0, interrupts triggered on the DCPU-16N
will turn on interrupt queueing, push **PC** to the stack, followed by pushing **A** to
the stack, then set the **PC** to **IA**, and **A** to the interrupt message.
 
If **IA** is set to 0, a triggered interrupt does nothing. Software interrupts still
take up four clock cycles, but immediately return, incoming hardware interrupts
are ignored. Note that a queued interrupt is considered triggered when it leaves
the queue, not when it enters it.

Interrupt handlers should end with RFI, which will disable interrupt queuing
and pop **A** and **PC** from the stack as a single atomic instruction.
IAQ is normally not needed within an interrupt handler, but is useful for time
critical code.


HARDWARE
===================================================================    

The DCPU-16N supports both memory or I/O bus mapped hardware devices. These
devices can be anything from additional storage, sensors, monitors or speakers.
How to control the hardware is specified per hardware device.

Both the I/O bus and Memory bus are 16 bits wide and can both contain hardware
devices. Either bus need not contain devices, or have hardware connected at all.

Some hardware devices may have only an 8 bit data bus connection, in these cases
reading or writing 16 bit values will take an additional cycle each read/write.

The DPCU-16N does not support hot swapping hardware. The behavior of connecting
or disconnecting hardware while the DCPU-16N computer is running is undefined
and not recommended, as this may damage the hardware and/or DCPU-16N.


EXTENDED MEMORY
============================================================

The DCPU-16N features a simple integrated memory control device called the 
Extended Memory Unit (EMU), that allows the DCPU-16N to access a 24 bit address
space. The EMU maps memory from the 16 bit virtual address space to the 24 bit
physical address space. The virtual memory space of the DCPU-16N is split
into 16 0x1000 byte blocks. The EMU maps each block to one of 4096 pages in
the 24 bit physical address space, where each page is also 0x1000 bytes.

Pages can be selected to blocks using the MMW instruction which takes a packed
12 bit page number and a 4 bit block number to map the page to. The pages may
be mapped to multiple blocks at once, meaning both blocks will refer to the 
same memory or hardware at that physical address.

The initial layout of the pages at boot time is defined by the computer system
that the DCPU-16N is used in. If left unspecified, blocks are set sequentially
starting at page 0. Computer systems should define at minimum, a setting for
block 0, or layout of page 0, to point a ROM/EPROM or other non-volitile
storage to ensure the DCPU-16N will boot to user machine code.

