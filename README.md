Proposition of Trillek Computer Specs
=====================================

**ADVICE** : In this documents there some technical stuff that could looks hard to understand from the newbies. 
Some of these stuff there is only to give natural limitations of what can do and can't do the computer. If your only interest is programing the computer, 
your should check the instrucction set of the CPU and the specs of the devices to understand how use the devices

SUMARY
------

- 32 bit RISC like CPU.
- I/O devices are memmory mapped.
- 512 KiB + 64Kib of ROM as base configuration. RAM expandable in chunks of 256KiB.
- Motherboard includes a Speaker and a *Timer* devices.
- To 32 devices via expansion bus.
- Hardware Enumeration, but not hot plugin of devices.
- Can use to 15 Interrupt Requests (IRQs).
- Devices can be configured via physical *jumper*. These jumpers can be read from 
  the software or change the value via software to allow some basic Plug & Play.


HOW WORKS
---------
![Computer Arquitecture Diagram](./computer.png "Diagram")

As can you see, the computer uses a 32 bit Address Bus and 32 bit Data bus. RAM and ROM are 
directly attached, to these busses, as any device in the computer that is controllabe by software.
Tehe is two special devices, the Hardware Enumerator and the Interrupt Controller, plus two 
integrated devices, the PIT and the Speaker

### Interrupt Controller and Interrupts

As can you read in the RC3200 CPU specs, only can accept a single interrupt petition. 
To handle simulatenus interrupts, we intercalate between the devices and the INT and IACQ lines, 
a Interrupt Controller. This Controller gives priority to the lowest IRQ when two or more interrupts
hapens at same time. The IRQx lines are use to send a Interrupt ReQuest from the devices, 
setting to Low level. The AIRQx lines are used by the Interrupt Controller to signal to the device 
that the interrupt request can be procesed now and that the device must put in the Data Bus the 
message to the CPU, when the IACQx line does the transition from High to Low.

**NOTE FOR USERS**: In other words, interrupt handlers are atomic, can't be interruped by other 
interrupt, and you only need to worry about the interrupt message in the ISR. This stuff is to put 
some limitations to the computer and add some details at implementation of it.

**NOTE FOR VM IMPLEMNTATION**: This means that when you executes the hardware devices, you only 
need to loop the device array in order and check if device x send a IRQ. If it hapens, allow it 
to send the mesasge to the CPU, and just igonre the IRQs for the rest of the loop.

### Hardware Enumerator and Plug & Play

The Hardware enumerator is a device that a boot time, halts the CPU using the WAIT line before 
begin to execute instrucctions, and begin to poll the 32 posible devices. When it ends to poll, 
it allows the CPU to begin to execute instrucctions. In this way, determines how many devices 
they are in the expansion bus.
In any time, responding to the software commands send to it, can togle ENUM_REQ and put the 
device number in the enumeration bus, and in the next clock cycle, puts the enumaration or 
jumper command. The device now must put the response in the Data Bus to be readed by the software.

**NOTE FOR USERS**: You only need to worry that this device allows to know how many devices they are
what are they, what does and allow to read or change some configuration parmeters as what 
memory address uses or what interrupt message can use (see device X specs for what does jumpers).

**NOTE FOR VM IMPLEMNTATION**: Number of devices comes from the lenght of the array of devices, 
the rest of the data can getters/setters in the device class.

### PIT (PROGRAMMABLE INTERVAL TIMER)

The PIT consists in two 32 bit timmers as can you find in any moderm micro-controller. Allow to 
do precise time measuremnts and generate periodic interrupts for system clock and tasks switchers.
Uses the IRQ0, so have the highest interrupt priority.

**NOTE FOR USERS**: Could look a bit more hard that the Noth's DCPU-16 Timer device, but 
allow to correct the derive of time by the latency of attending the interrupt and procesing 
it and give you more control. Plus is more easy to understand and use that the IBM PC timer.

**NOTE FOR VM IMPLEMENTATION**: Uses two vars per timer. One stores the Reload value and the 
other count downs every timer clock tick. The times generated are in VM time, so if you run 
the VM at 200% speed, the times should be the half.

### Speaker
Simple basic Speaker with similar functionality to the IBM PC speaker of ZX Spectrum beeper. It 
have less power as can't allow do PWM to generate PCM sound, but it makes a lot more simple 
to use and understand.

**NOTE FOR VM IMPLEMENTATION**: Try to use a Band-Limited Sound Synthesis lib to generate the sound. 

DOCUMENTS
---------

- [RC3200 CPU](./RC3200.md)
- [Hardware Enumeration](./Hardware_Enumeration.md)
- [Programable Interval Timer](./Timers.md) (aka Timer or Clock)
- [Speaker](./Speaker.md)
- [Generic Keyboard](./Keyboard.md)
- [Color Display Adapter](./CDA.md) (CDA)
- [Computer Arquitecture Diagram](./computer.dia) (DIA file)
