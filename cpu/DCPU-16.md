---
layout : default
title : DCPU-16 CPU
cat : CPU
---
# DCPU-16 Specification  

#### Version 1.8a (Base on Mojang's v 1.7)  



SUMMARY
================================================================================

* 16 bit CISC CPU
* 16 bit address space
* 16 bit I/O address
* [word addressing](http://en.wikipedia.org/wiki/Word-addressable) (minimal address unit is a word)
* little endian architecture
* 8 registers (A, B, C, X, Y, Z, I, J)
* program counter (PC)
* stack pointer (SP)
* extra/excess (EX)
* interrupt address (IA)

In this document, anything within [brackets] is shorthand for "the value of the
RAM at the location of the value inside the brackets". For example, SP means
stack pointer, but [SP] means the value of the RAM at the location the stack
pointer is pointing at.
Also in the notation we will use **BYTE** or octect for 8 bit values, **WORD** 
for 16 bit values and **DWORD** for 32 bit values.

This CPU does word addressing, this means that the minimal address unit is a 
WORD.

Whenever the CPU needs to read a word, it reads [PC], then increases PC by one.
Shorthand for this is [PC++]. In some cases, the CPU will modify a value before
reading it, in this case the shorthand is [++PC].




INSTRUCTIONS
================================================================================

Instructions are 1-3 words long and are fully defined by the first word.
In a basic instruction, the lower five bits of the first word of the instruction
are the **opcode**, and the remaining eleven bits are split into a five bit value b
and a six bit value a.
b is always handled by the processor after a, and is the lower five bits.
In bits (in LSB-0 format), a basic instruction has the format: `aaaaaabbbbbooooo`

In the tables below, C is the time required in cycles to look up the value, or
perform the opcode, VALUE is the numerical value, NAME is the mnemonic, and
DESCRIPTION is a short text that describes the opcode or value.



### Values: (5/6 bits)

 C | VALUE     | DESCRIPTION
:-:|----------:|----------------------------------------------------------------
 0 | 0x00-0x07 | register (**A**, **B**, **C**, **X**, **Y**, **Z**, **I** or **J**, in that order)
 1 | 0x08-0x0f | [register]
 2 | 0x10-0x17 | [register + *next word*]
 1 |      0x18 | (PUSH / [--**SP**]) if in b, or (POP / [**SP**++]) if in a
 1 |      0x19 | [**SP**] / PEEK
 2 |      0x1a | [**SP** + *next word*] / PICK n
 0 |      0x1b | **SP**
 0 |      0x1c | **PC**
 0 |      0x1d | **EX**
 1 |      0x1e | [next word]
 1 |      0x1f | next word (literal)
 0 | 0x20-0x3f | literal value 0xffff-0x1e (-1 to 30) (literal) (only for a)

  
* "next word" means "[PC++]". Increases the word length of the instruction by 1.
* By using 0x18, 0x19, 0x1a as PEEK, POP/PUSH, and PICK there's a reverse stack
  starting at memory location 0xffff. Example: "SET PUSH, 10", "SET X, POP"
* Attempting to write to a literal value fails silently



### Basic opcodes (5 bits)

 C | VAL  | NAME     | DESCRIPTION
:-:|------|----------|---------------------------------------------------------
 - | 0x00 | n/a      | special instruction - see below
 1 | 0x01 |`SET b, a`| sets b to a
 2 | 0x02 |`ADD b, a`| sets b to b+a, sets EX to 0x0001 if there's an overflow, 0x0 otherwise
 2 | 0x03 |`SUB b, a`| sets b to b-a, sets EX to 0xffff if there's an underflow, 0x0 otherwise
 10| 0x04 |`MUL b, a`| sets b to b*a, sets EX to ((b*a)>>16)&0xffff (treats b, a as unsigned)
 15| 0x05 |`MLI b, a`| like MUL, but treat b, a as signed
 20| 0x06 |`DIV b, a`| sets b to b/a, sets EX to ((b<<16)/a)&0xffff. if a==0, sets b and EX to 0 instead. (treats b, a as unsigned)
 25| 0x07 |`DVI b, a`| like DIV, but treat b, a as signed. Rounds towards 0
 20| 0x08 |`MOD b, a`| sets b to b%a. if a==0, sets b to 0 instead.
 25| 0x09 |`MDI b, a`| like MOD, but treat b, a as signed. (MDI -7, 16 == -7)
 1 | 0x0a |`AND b, a`| sets b to b&a
 1 | 0x0b |`BOR b, a`| sets b to b|a
 1 | 0x0c |`XOR b, a`| sets b to b^a
 1 | 0x0d |`SHR b, a`| sets b to b>>>a, sets EX to ((b<<16)>>a)&0xffff (logical shift)
 1 | 0x0e |`ASR b, a`| sets b to b>>a, sets EX to ((b<<16)>>>a)&0xffff (arithmetic shift) (treats b as signed)
 1 | 0x0f |`SHL b, a`| sets b to b<<a, sets EX to ((b<<a)>>16)&0xffff
 2+| 0x10 |`IFB b, a`| performs next instruction only if (b&a)!=0
 2+| 0x11 |`IFC b, a`| performs next instruction only if (b&a)==0
 2+| 0x12 |`IFE b, a`| performs next instruction only if b==a 
 2+| 0x13 |`IFN b, a`| performs next instruction only if b!=a 
 2+| 0x14 |`IFG b, a`| performs next instruction only if b>a 
 2+| 0x15 |`IFA b, a`| performs next instruction only if b>a (signed)
 2+| 0x16 |`IFL b, a`| performs next instruction only if b<a 
 2+| 0x17 |`IFU b, a`| performs next instruction only if b<a (signed)
 - | 0x18 | -        |
 - | 0x19 | -        |
 3 | 0x1a |`ADX b, a`| sets b to b+a+EX, sets **EX** to 0x0001 if there is an overflow, 0x0 otherwise
 3 | 0x1b |`SBX b, a`| sets b to b-a+EX, sets **EX** to 0xFFFF if there is an underflow, 0x0 otherwise
 - | 0x1c | -        |
 - | 0x1d | -        |
 2 | 0x1e |`STI b, a`| sets b to a, then increases I and J by 1
 2 | 0x1f |`STD b, a`| sets b to a, then decreases I and J by 1


* The branching opcodes take one cycle longer to perform if the test fails
  When they skip an if instruction, they will skip an additional instruction
  at the cost of one extra cycle. This lets you easily chain conditionals.  
* Signed numbers are represented using two's complement arithmetic.

## Special instructions
    
Special opcodes always have their lower five bits unset, have one value and a
five bit opcode. In binary, they have the format: `aaaaaaooooo00000`
The value (a) is in the same six bit format as defined earlier.

### Special opcodes: (5 bits)

 C  | VAL  | NAME  | DESCRIPTION
:--:|------|-------|-------------------------------------------------------------
 -  | 0x00 | n/a   | implied instruction - see below
 3  | 0x01 |`JSR a`| pushes the address of the next instruction to the stack,
    |      |       | then sets **PC** to **a**
 -  | 0x02 | -     |
 -  | 0x03 | -     |
 -  | 0x04 | -     |
 -  | 0x05 | -     |
 -  | 0x06 | -     |
 -  | 0x07 | -     | 
 4  | 0x08 |`INT a`| triggers a software interrupt with message **a**
 1  | 0x09 |`IAG a`| sets **a** to **IA** 
 1  | 0x0a |`IAS a`| sets **IA** to **a**
 3  | 0x0b |`RFI a`| disables interrupt queueing, pops A from the stack, then 
    |      |       | pops PC from the stack
 2  | 0x0c |`IAQ a`| if **a** is nonzero, interrupts will be added to the queue
    |      |       | instead of triggered. if a is zero, interrupts will be
    |      |       | triggered as normal again
 -  | 0x0d | -     |
 -  | 0x0e | -     |
 -  | 0x0f | -     |
 350| 0x10 |`HWN a`| sets **a** to number of connected hardware devices
 8  | 0x11 |`HWQ a`| sets A, B, X, Y registers to information about hardware **a**
 14 | 0x12 |`HWI a`| sends an "interrupt" to hardware **a**
 -  | 0x13 | -     |
 -  | 0x14 | -     |
 -  | 0x15 | -     |
 -  | 0x16 | -     |
 -  | 0x17 | -     |
 -  | 0x18 | -     |
 -  | 0x19 | -     |
 -  | 0x1a | -     |
 -  | 0x1b | -     |
 -  | 0x1c | -     |
 -  | 0x1c | -     |
 2  | 0x1e |`BNK a`| sets bank to a
 2  | 0x1f |`CHG`  | switch Least Significant and Most Significant bytes of a

* RBK changes the ROM bank being used. At reboot is bank 0.
* BNK changes the RAM bank being used. At reboot is bank 0.


## Implied instructions

Implied opcodes always have their lower ten bits unset, have a bit value and a
five bit opcode. In binary, they have the format: `oooooo0000000000`


### Implied opcodes: (5 bits)

 C | VAL  | NAME  | DESCRIPTION
:-:|------|-------|-------------------------------------------------------------
 1 | 0x00 |`SLP`  | simple halts CPU operation. If interrupts are enabled, an incoming interrupt will wake up the CPU.
 - | 0x01 | -     |
 - | 0x02 | -     |
 - | 0x03 | -     |
 - | 0x04 | -     |
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
 - | 0x10 | -     |
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



INTERRUPTS
================================================================================

The DCPU-16 will perform at most one interrupt between each instruction. If
multiple interrupts are triggered at the same time, they are added to a queue.
If the queue grows longer than 256 interrupts, the DCPU-16 will catch fire. 

When **IA** is set to something other than 0, interrupts triggered on the DCPU-16
will turn on interrupt queueing, push PC to the stack, followed by pushing A to
the stack, then set the PC to IA, and A to the interrupt message.
 
If **IA** is set to 0, a triggered interrupt does nothing. Software interrupts still
take up four clock cycles, but immediately return, incoming hardware interrupts
are ignored. Note that a queued interrupt is considered triggered when it leaves
the queue, not when it enters it.

Interrupt handlers should end with RFI, which will disable interrupt queueing
and pop A and PC from the stack as a single atomic instruction.
IAQ is normally not needed within an interrupt handler, but is useful for time
critical code.



BANKS
================================================================================
To allow to operate with more that 64KiW (128KiB) of RAM/ROM, the DCPU-16 
implements a simple memory bank scheme.

The first 16 KiW (= 32 KiB), this is addresses 0x0000 to 0x3FFFF, are mapped to a 
bank. `BNK a` changes the bank used. Setting it to a not existent bank makes that 
reads in these address range, return inconsistent junk data. If the bank is not 
existent or correspond to ROM, writes to it fails silently. When the DCPU-16 
does a reboot or boots is set to bank 0.

The address range from 0x4000 to 0xFFFF is always mapped to RAM, this is 48KiW 
(= 96KiB).


HARDWARE
================================================================================

The DCPU-16 supports up to 32 connected hardware devices. These devices can
be anything from additional storage, sensors, monitors or speakers.
How to control the hardware is specified per hardware device, but the DCPU-16
supports a standard enumeration method for detecting connected hardware via
the **HWN**, **HWQ** and **HWI** instructions.

**HWQ** sets the CPU registers with this values :

- A register is **Device Type** (Byte)
- B register is **Device SubType** (Byte) + **Device ID** (Byte) << 8
- X+(Y<<16) is a 32 bit word identifying the **Device Vendor/Builder**

**HWI** (aka "Interrupt to Hardware") sent a "message" to a hardware device,
using register **Z** as command value, and **A**, **B**, **C** and **X** as 
command values. Command 0xFFFF, simply does a read operation, setting CPU 
registers **A**, **B**, **C** and **X** to value set by the device.


ANNEX : Bridge between DCPU-16 and Trillek Computer Architecture
================================================================================

The bank scheme used by the DCPU-16 allows to use the whole ROM and RAM of the 
computer architecture. The address range from 0x4000 to 0xFFFF is correspond to
computer addresses 0x8000 to 0x1FFFF, as the DCPU-16 address words and not bytes.

The banks lists is :

 BANK |  COMPUTER ADDRESS   | DESCRIPTION
:----:|:-------------------:|---------------------------------------------------
  0   | 0x100000 - 0x107FFF | ROM page
  1   | 0x000000 - 0x007FFF | RAM page 0
  2   | 0x020000 - 0x027FFF | RAM page 1
  3   | 0x028000 - 0x02FFFF | RAM page 2
  4   | 0x030000 - 0x037FFF | RAM page 3
  5   | 0x038000 - 0x03FFFF | RAM page 4
  6   | 0x040000 - 0x047FFF | RAM page 5
  7   | 0x048000 - 0x04FFFF | RAM page 6
  8   | 0x050000 - 0x057FFF | RAM page 7
  9   | 0x058000 - 0x05FFFF | RAM page 8
  X   | ...      - ...      | RAM page X
  31  | 0x0F0000 - 0x0F7FFF | RAM page 30
  32  | 0x0F8000 - 0x0FFFFF | RAM page 31
  33  | 0x110000 - 0x117FFF | Device Enumeration and Control page 0
  34  | 0x118000 - 0x11FFFF | DEC page 1

Bank 0 is used by the ROM and is the bank used when the computer boot up, 
allowing the ROM code to bootstrap the computer. RAM banks 0 to 31, allows to 
use the extra RAM of the computer, and the DEC (Devices Enumeration and 
Control) banks, allows low level access to the Device Enumeration and 
Control hardware registers of each device.


**HWN**, **HWQ** and **HWI** are hard code subroutines that access to the 
Device Enumeration & Control registers doing. In particular, **HWI** does write/read 
operations on Enumeration & Control register of a particular device.

Any command to a device that expects a memory address, MUST be in computer 
addresses (i.e. byte addresses). So not forget to do a x2 multiplication (left 
shift by one bit)

 
