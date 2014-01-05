DCPU-16 Specification
=====================

Version 2.0a (WIP)

SUMMARY
=======

* 16 bit CPU
* 32 bit address space
* 8 generic purpose registers (A, B, C, X, Y, Z, I, J)
* 4 segment selector registers (PS, DS, SS, IS)
* program counter (PC)
* stack pointer (SP)
* extra/excess (EX)
* interrupt address (IA)

In this document, anything within [brackets] is shorthand for "the value of the
RAM at the location of the value inside the brackets". For example, A means the 
value of A register, but [A] means the value of the RAM at the location that 
points (DS << 16) | A. Also anything i nthe form [bar:foo] means that is the 
value of the RAM at the location that points (barr << 16) | foo . So the previus 
example could be write as [DS:A]. Also in the notation we will use BYTES for 8 
bit values, WORDS for 16 bit values and DWORDs for 32 bit values.

Whenever the CPU needs to read an instruction, it reads [PS:PC], then increases 
PC by two as every instrucction is 16 bit wide. Shorthand for this is [PS:PC+2].
In some cases, the CPU will modify a value before [U:++V] or [U:--V]were U is a Segment 
register and V is a register. 
When we say that the DCPU-16 Push a value to the Stack, writes these value at 
[SS:--SP]. When we say that TR3200 Pops a value from the stack, reads a value 
from [SS:SP++]. Remember that we work with 16 bit values in Little Endian, so 
to store a value in the Stack, we need to push each byte of a word value.

Code must be aligned to 2 byte boundary.

Registers :
 
  - A, B, C, X, Y, Z, I, J - General Purpose Registers (GPR) of 16 bit
  - PS - Program Segment register
  - PC - Program Counter. [PS:PC] points to the next instruction to be executed
  - SS - Stack Segment register
	- SP - Satck Pointer. [SS:SP] points to the top of the Stack
  - EX - Excess/Extra registers. Used to store the carry/burrow or excess in some ALU operations  
	- IS - Interrupt Segment
	- IA - Interrupt Address. [IS:IA] points to the Interrupt service routine.
	- DS - Data Segment register. Memory read/write uses DS as the 16 bit MSB part of the address.

BOOT/RESET STATUS
-----------------

All registers are set to 0 when the TR3200 boot-up or does a reset.

INSTRUCTIONS
------------

Instructions are 1-3 words long and are fully defined by the first word.
In a basic instruction, the lower five bits of the first word of the instruction
are the opcode, and the remaining eleven bits are split into a five bit value b
and a six bit value a.
b is always handled by the processor after a, and is the lower five bits.
In bits (in LSB-0 format), a basic instruction has the format: aaaaaabbbbbooooo

In the tables below, C is the time required in cycles to look up the value, or
perform the opcode, VALUE is the numerical value, NAME is the mnemonic, and
DESCRIPTION is a short text that describes the opcode or value.

    --- Values: (5/6 bits) ---------------------------------------------------------
     C | VALUE     | DESCRIPTION
    ---+-----------+----------------------------------------------------------------
     0 | 0x00-0x07 | register (A, B, C, X, Y, Z, I or J, in that order)
     1 | 0x08-0x0f | [DS:register]
     2 | 0x10-0x17 | [DS:(register + next word)]
     0 |      0x18 | (PUSH / [SS:--SP]) if in b, or (POP / [SS:SP++]) if in a
     0 |      0x19 | [SS:SP] / PEEK
     1 |      0x1a | [SS:(SP + next word)] / PICK n
     0 |      0x1b | SP
     0 |      0x1c | PC
     0 |      0x1d | EX
     1 |      0x1e | [DS:next word]
     1 |      0x1f | next word (literal)
     0 | 0x20-0x3f | literal value 0xffff-0x1e (-1..30) (literal) (only for a)
    ---+-----------+----------------------------------------------------------------
  
- "next word" means "[PC+2]". Increases the word length of the instruction by 1.
- Stack operations (PUSH/POP/PEEK) stores/reads a word in the Stack.
- By using 0x18, 0x19, 0x1a as PEEK, POP/PUSH, and PICK there's a reverse stack
  starting at memory location SS:FFFF. Example: "SET PUSH, 10", "SET X, POP"
- Attempting to write to a literal value fails silently
- Any write to PC register gets cleared his lower bit, enforcing PC register 
  value to be a multiple of 2.

    --- Basic opcodes (5 bits) ----------------------------------------------------
     C | VAL  | NAME     | DESCRIPTION
    ---+------+----------+---------------------------------------------------------
     - | 0x00 | n/a      | Type 2 or 3 instruction - see below
     2 | 0x01 | SET b, a | sets b to a
     2 | 0x02 | ADD b, a | sets b to b+a, sets EX to 0x0001 if there's an overflow, 
       |      |          | 0x0 otherwise
     2 | 0x03 | SUB b, a | sets b to b-a, sets EX to 0xffff if there's an underflow,
       |      |          | 0x0 otherwise
     20| 0x04 | MUL b, a | sets b to b*a, sets EX to ((b*a)>>16)&0xffff (treats b,
       |      |          | a as unsigned)
     30| 0x05 | MLI b, a | like MUL, but treat b, a as signed
     30| 0x06 | DIV b, a | sets b to b/a, sets EX to ((b<<16)/a)&0xffff. if a==0,
       |      |          | sets b and EX to 0 instead. (treats b, a as unsigned)
     40| 0x07 | DVI b, a | like DIV, but treat b, a as signed. Rounds towards 0
     30| 0x08 | MOD b, a | sets b to b%a. if a==0, sets b to 0 instead.
     40| 0x09 | MDI b, a | like MOD, but treat b, a as signed. (MDI -7, 16 == -7)
     2 | 0x0a | AND b, a | sets b to b&a
     2 | 0x0b | BOR b, a | sets b to b|a
     2 | 0x0c | XOR b, a | sets b to b^a
     2 | 0x0d | SHR b, a | sets b to b>>>a, sets EX to ((b<<16)>>a)&0xffff 
       |      |          | (logical shift)
     2 | 0x0e | ASR b, a | sets b to b>>a, sets EX to ((b<<16)>>>a)&0xffff 
       |      |          | (arithmetic shift) (treats b as signed)
     2 | 0x0f | SHL b, a | sets b to b<<a, sets EX to ((b<<a)>>16)&0xffff
     3+| 0x10 | IFB b, a | performs next instruction only if (b&a)!=0
     3+| 0x11 | IFC b, a | performs next instruction only if (b&a)==0
     3+| 0x12 | IFE b, a | performs next instruction only if b==a 
     3+| 0x13 | IFN b, a | performs next instruction only if b!=a 
     3+| 0x14 | IFG b, a | performs next instruction only if b>a 
     3+| 0x15 | IFA b, a | performs next instruction only if b>a (signed)
     3+| 0x16 | IFL b, a | performs next instruction only if b<a 
     3+| 0x17 | IFU b, a | performs next instruction only if b<a (signed)
     - | 0x18 | -        |
     2 | 0x19 | NOT b, a | sets b to !a
     3 | 0x1a | ADX b, a | sets b to b+a+EX, sets EX to 0x0001 if there is an over-
       |      |          | flow, 0x0 otherwise
     3 | 0x1b | SBX b, a | sets b to b-a+EX, sets EX to 0xFFFF if there is an under-
       |      |          | flow, 0x0001 if there's an overflow, 0x0 otherwise
     4 | 0x1c | LJMP b, a| sets PS to b and sets PC to a. 
     5 | 0x1d | LJSR b, a| pushes the PS register to the stack, then push address of
		   |      |          | the next instruction to the stack. Finally sets PS to b and PC to a.
     3 | 0x1e | STI b, a | sets b to a, then increases I and J by 1
     3 | 0x1f | STD b, a | sets b to a, then decreases I and J by 1
    ---+------+----------+----------------------------------------------------------

- The conditional opcodes take one cycle longer to perform if the test fails.
  When they skip a conditional instruction, they will skip an additional
  instruction at the cost of one extra cycle. This continues until a non-
  conditional instruction has been skipped. This lets you easily chain
  conditionals. Interrupts are not triggered while the DCPU-16 is skipping.
- Signed numbers are represented using two's complement.
- Any write to PC register gets cleared his lower bit, enforcing PC register 
  value to be a multiple of 2.
    
Type 2 or 3 instruction always have their lower five bits unset, have one value and a
five bit opcode. In binary, they have the format: aaaaaaooooo00000
The value (a) is in the same six bit format as defined earlier.

    --- Special opcodes: (5 bits) --------------------------------------------------
     C | VAL  | NAME  | DESCRIPTION
    ---+------+-------+-------------------------------------------------------------
     0 | 0x00 | n/a   | Type 3 instructions - see below
     4 | 0x01 | JSR a | pushes the address of the next instruction to the stack,
       |      |       | then sets PC to a. a most lowest bit is ignored
     - | 0x02 | -     |
     - | 0x03 | -     |
     - | 0x04 | -     |
     - | 0x05 | -     |
     2 | 0x06 | SIG a | extend the sign of the LSB byte
     2 | 0x07 | XCH a | excahnge LSB and MSB bytes of a 
     6 | 0x08 | INT a | triggers a software interrupt with message a
     2 | 0x09 | IAG a | sets a to IA 
     2 | 0x0a | IAS a | sets IA to a
     5 | 0x0b | RFI a | disables interrupt queueing, pops A from the stack, then 
       |      |       | pops PC from the stack and finally pops PS from the stack
     3 | 0x0c | IAQ a | if a is nonzero, interrupts will be added to the queue
       |      |       | instead of triggered. if a is zero, interrupts will be
       |      |       | triggered as normal again
     - | 0x0d | PSG   | Sets a to PS
     - | 0x0e | PSS   | Sets PS to a
     - | 0x0f | DSG   | Sets a to DS
     - | 0x10 | DSS   | Sets DS to a
     - | 0x11 | SSG   | Sets a to SS
     - | 0x12 | SSS   | Sets SS to a
     - | 0x13 | ISG   | Sets a to IS 
     - | 0x14 | ISS   | Sets IS to a
     - | 0x15 | -     |
     - | 0x16 | -     |
     - | 0x17 | -     |
     - | 0x18 | -     |
     - | 0x19 | -     |
     - | 0x1a | -     |
     - | 0x1b | -     |
     - | 0x1c | -     |
     - | 0x1d | -     |
     - | 0x1e | -     |
     - | 0x1f | -     | 
    ---+------+-------+-------------------------------------------------------------


- Any write to PC register gets cleared his lower bit, enforcing PC register 
  value to be a multiple of 2.
- Any write to IA register gets cleared his lower bit, enforcing IA register 
  value to be a multiple of 2.

Type 3 instructions alweays have their lower ten bits unset, have no operands and a is a 
six bit opcode. In binary, they have the format: oooooo0000000000
 
    --- SuperSpecial opcodes: (6 bits) --------------------------------------------------
     C | VAL | NAME | DESCRIPTION
    ---+------+-------+-------------------------------------------------------------
     - | 0x00 | SLEEP | Sleeps the CPU and waits that a uinterrupt happens
     - | 0x01 | -     |
     - | 0x02 | -     |
     - | 0x03 | -     |
     - | 0x04 | -     |
     - | 0x05 | -     |
     - | 0x06 | -     |
     - | 0x07 | -     |
     - | 0x08 | -     |
     - | 0x08 | -     |
     - | 0x09 | -     |
     - | 0x0a | -     |
     - | 0x0b | -     |
     - | 0x0c | -     |
     - | 0x0d | -     |
     - | 0x0e | -     |
     - | 0x0f | -     |
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
     - | 0x1a | -     |
     - | 0x1b | -     |
     - | 0x1c | -     |
     - | 0x1d | -     |
     - | 0x1e | -     |
     - | 0x1f | -     |
     - | 0x20 | -     |
     - | 0x21 | -     |
     - | 0x22 | -     |
     - | 0x23 | -     |
     - | 0x24 | -     |
     - | 0x25 | -     |
     - | 0x26 | -     |
     - | 0x27 | -     |
     - | 0x28 | -     |
     - | 0x28 | -     |
     - | 0x29 | -     |
     - | 0x2a | -     |
     - | 0x2b | -     |
     - | 0x2c | -     |
     - | 0x2d | -     |
     - | 0x2e | -     |
     - | 0x2f | -     |
     - | 0x30 | -     |
     - | 0x31 | -     |
     - | 0x32 | -     |
     - | 0x33 | -     |
     - | 0x34 | -     |
     - | 0x35 | -     |
     - | 0x36 | -     |
     - | 0x37 | -     |
     - | 0x38 | -     |
     - | 0x39 | -     |
     - | 0x3a | -     |
     - | 0x3b | -     |
     - | 0x3c | -     |
     - | 0x3d | -     |
     - | 0x3e | -     |
     - | 0x3f | -     |
    ---+------+-------+-------------------------------------------------------------

INTERRUPTS
==========

The DCPU-16 will perform at most one interrupt between each instruction. If
multiple interrupts are triggered at the same time, they are added to a queue.
If the queue grows longer than 256 interrupts, the DCPU-16 will not signal the 
hardware to accept the incoming new interrupts bia the IACQ signal. 

When IA is set to something other than 0, interrupts triggered on the DCPU-16
will turn on interrupt queueing, push PS to the stack, followed by pushing PC to
the stack, followed by pushing A to the stack, then set the [PS:PC] to [IS:IA], 
and A to the interrupt message.
 
If IA is set to 0, a triggered interrupt does nothing. Software interrupts still
take up six clock cycles, but immediately return, incoming hardware interrupts
are ignored. Note that a queued interrupt is considered triggered when it leaves
the queue, not when it enters it.

Interrupt handlers should end with RFI, which will disable interrupt queueing
and pop A, PC and PS from the stack as a single atomic instruction.
IAQ is normally not needed within an interrupt handler, but is useful for time
critical code.

HARDWARE
========

The DCPU-16 handles hardware devices as memory mapped. The usual addresses uses by
the devices begins at FF00:0000, that is the last 16 MiB of the address space. 


EXAMPLE MEMORY MAP
------------------


The memory map is defined by the OS and computer architecture, but here we give
a example memory map for a 128KiB RAM+ 64KiB ROM and a graphics device at 0xFF0A:0000 :


    [FFFF:FFFF] |---------|
                |         |
                :         :
                : Unused  :
                :         :
                |         |
    [FF0A:2580] |---------|
                |         |
                |  VIDEO  |
                |   RAM   |
                |         |
    [FF0A:0000] |---------|
                |         |
                :         :
                : Unused  :
                :         :
                |         |
    [0002:0000] |---------|
                |         |
                |  STACK  |
                |         |
                vvvvvvvvvvv
                |         |
                :         :
                :Available:
                :         :
                |         |
    [0001:0400] |---------|
                |Interrupt|
                | Service |
                | Rutines |
    [0001:0000] |---------|
                |         |
                |   BIOS  |
                |   ROM   |
                |         |
    [0000:0000] |---------|

It uses a mapped video ram to represent a text only video mode, and uses a 64Kib
ROM that works as Basic Input Output System (BIOS) that sets the hardware, gives some 
basic ISRs, some utile functions, and launch user code from a floppy.

ELECTRICAL SIGNALS
==================

- Data Bus (Input/Output) of 16 bits
- Address Bus (Input/Output) of 32 bits
- R/W (Output): Low to signal that the CPU ask to read an address, High to
  signal that the CPU writes an address.
- _INT (Input): Used to signal the CPU that a hardware interrupt is happening
- IACQ (Output): Indicates that the CPU is processing an interrupt and will not
  accept new interrupts. Only can accept new interrupts if is low.
- _RESET (Input): Resets the CPU state if a edge from High to Low hapens.
- _Wait (Input): Makes to the CPU wait at actual stage. Used to arbitrate in
  multi-cpu systems and DMA use of the bus.
- AD_ACT (Output): Indicates that the CPU is using the Address and Data bus


PROTOCOLS
---------

### Sending an interrupt

1.   Pull down _INT line only if IACQ is low
2.   Wait to IACQ gets High
3.   When IACQ gets High, put at data bus the interrupt message.
4.   Release _INT line
5.   Wait to IACQ gets Low -> End Of Interrupt (EOI)

