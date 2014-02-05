Trillek Virtual Computer Specs
=====================================
Version 0.4 

**ADVICE** : In this documents there some technical stuff that could looks hard
or complex to understand for not hardware guys.
Some of these stuff there is only to give natural limitations of what can do 
and can't do the computer. If your only interest is programing the computer, 
you should check the instruction set of a CPU and the specs of the devices to
understand how program the computer and use the devices at assembly or C programing
level.

SUMMARY
------

- 32 bit data bus, but allow transfers of 16 and 8 bit.
- 24 bit address space.
- 32KiB ROM chip at address 0x100000-0x107FFF
- Initial 128KiB RAM at 0x000000- 0x01FFFF
- RAM expandable with modules of 128KiB to a total of 1 MiB of RAM (0x000000-0x0FFFFF)
- CPUs are connected by a CPU board (actually TR3200 and DCPU-16E) to the mother board. Only one CPU can be connected to the computer at same time (not multi-processor setups)
- CPU Clock speed could be 1Mhz , 500 Khz, 200 Khz and 100Khz (actually we work with 100Khz, but we expect to allow higher speeds)
- Devices uses a fixed clock of 100Khz (thinking to change it to 50 KHz) for internal stuff.
- Devices are [memory mapped](http://en.wikipedia.org/wiki/Memory-mapped_I/O).So dcpu's **HWI** is replaced by writing/reading to addresses where the device is listening. **HWN** and **HWQ** is replaced by a special device that lists how many and what devices are. This integrated device is the **Hardware Enumerator** (**HE**).
- Devices could have a physical switch, called *Jumper*, that allow to change some operations values when the computer is power off. Jumper value is limited to a 4 values (0 to 3)
- Addresses used by devices are over 0x110000 to avoid address clashes with the RAM/ROM.
- The **Hardware Enumerator** is “intelligent” and assigns physical address blocks to the devices (except integrated devices that have always a fixed and concrete addresses). Some devices could have preferred address blocks, and that will get it if other devices is not using it yet. Other devices can don’t care much of what address block get.
- Devices could do **DMA** operations at will, but ONLY one device could do that at same time, and can only transfer 4 bytes every Device Clock (like if the DMA operates in the falling clock flank and the CPU operated in the rising clock flank.)
- The computer can be expanded to a total 32 devices, not counting integrated devices on motherboard. This can be archived by plugin the device boards in the expansion bus. Some devices will requiere a external module attached to the computer, like floppy drives, graphics cards, joysticks, weapons, etc...
- Integrated devices on motherboard:
    - Programmable Interval Timer (**PIT**) aka *Clock* device.
    - Real Time Clock (**RTC**), that gives the date and time in game world when is polled (not have alarm).
    - Random Number Generator (**RNG**), that generates a 32 bit random number every time that is polled (at implementation level, a simple call to rand_r)
    - Beeper or *buzzer* device (**Beeper**). Simply generates a squared wave sound at desired frequency.
    - **Keyboard controller**. The computer can be expanded with additional keyboard controllers, but this is the base.



HOW WORKS
---------
![Computer Architecture Diagram](./computer.png "Diagram")

As can you see, the computer uses a 24 bit Address Bus and 32 bit Data bus. RAM
and ROM are directly attached to these buses, as any device in the computer
that is controllable by software. Also there is the integrated devices and the **Hardware Enumerator**.

### Interrupts

To avoid clashes with interrupt petitions, we daisy chain the interrupt signals *INT* and *IACQ* . So when two devices try to generate a interrupt at same time, the device more near to the CPU (with lowest slot number), have preference. The **PIT** and **Keyboard controller** devices can generate interrupts, so we put it between the expandable devices and the CPU having more preference that any expansion device. Plus the **PIT** have more preference as is more near to the CPU that the Keyboard Controller.

**NOTE FOR USERS**: In other words, you only need to worry about the interrupt 
message in your **Interrupt Service Routine** (ISR). This stuff is to put some limitations to the computer and 
add some details at implementation of it.

**NOTE FOR IMPLEMENTATION**: This means that when you need to "executes" the hardware 
devices, you only need to loop the device array in order and check if device
**x** send a Interrupt. If it happens, allow it to send the message to the CPU, and 
just ignore the Interrupt petitions for the rest of the loop.


### Hardware Enumerator and Plug & Play

The hardware enumerator is a device that a boot time, halts the CPU before begin to execute instructions, and begin to poll the 32 possible devices. When polls a device, it gets the number of address blocks and his size that the device needs. The hardware enumerator assigns the address block to a not assigned address region and communicate to the device what address region to use. When it ends to poll, it allows the CPU to begin to execute instructions. 
Software can poll the Hardware Enumerator about a particular device slot to get information about it and what address blocks are using and where are mapped.Additionally contains a few read only registers with information about the computer itself, as the motherboard build id.

**NOTE FOR USERS**: You only need to worry that this device allows (reading/writing a particular addresses) to know how many devices they are, what device are and what address you must read/write to operate it.

**NOTE FOR VM IMPLEMENTATION**: Number of devices comes from the length of the 
array of devices, the rest of the data can come from getters/setters in the device class. The emplace algorithm not should be complex as you have around 14 MiB of address space to emplace devices that usually uses a few bytes of address space.

### PIT (PROGRAMMABLE INTERVAL TIMER)

The PIT consists in two 32 bit timers as can you find in any modern 
micro-controller. Allow to do time measurements and generate periodic 
interrupts for system clock and tasks switchers. Have the highest priority when needs to signal a interrupt.

**NOTE FOR USERS**: Could look a bit more hard that the Noth's DCPU-16 Timer 
device, but gives more freedom and control. Plus is more easy to 
understand and use that the IBM PC timer. Using the highest interrupt priority means that will be the
first Interrupt to be attended by the CPU when simultaneous interrupts happens.

**NOTE FOR VM IMPLEMENTATION**: Uses two vars per timer. One stores the Reload 
value and the other count downs every timer clock tick. The times generated are
in Virtual Computer time, so if you run the Virtual Computer at 200% speed, the measured times should be the half.

### RTC (Real Time Clock)

Is a basic device that gives the actual game time and date. Not have alarm, so is necessary doing a polling every 12 or 24 hours to keep a software clock in sync with game time.

### RNG (Random Number Generator)
Is a basic device that writing to it, sets the RNG seed, and reading from it, gets a 32 bit random number.

### Beeper
Simple basic Beeper with similar functionality to the IBM PC speaker or ZX Spectrum beeper. It have less power as can't allow do PWM to generate basic crude PCM sound
, but it makes a lot more simple to use and understand.

**NOTE FOR VM IMPLEMENTATION**: Try to use a Band-Limited Sound Synthesis lib 
to generate square wave sound, but a crude Fourier synthesis could do the trick.

### Devices with DMA (Direct Memory Access)
DMA operations by the hardware devices are allowed, but only one DMA operation could be happening at same time at a rate of 4 byte for each device clock. To avoid that two or more devices try to do a DMA operation, there is a BUSY BUS signal. DMA operations happens in the opposite flank that the CPU clock, so don't interfere and not need contention hardware.

**NOTE FOR VM IMPLEMENTATION**: By practical reasons, this will translated in a flag in the Virtual Computer to indicate if a device will being doing DMA, as can't be two devices doing a DMA at same time. 

DOCUMENTS
---------

- [TR3200 CPU](./TR3200.md)
- [DCPU-16E CPU](./DCPU-16E.txt)
- [Hardware Enumeration](./Hardware_Enumeration.md)
- [Programmable Interval Timer](./Timers.md) (aka Timer or Clock)
- [Beeper](./Beeper.md)
- [Generic Keyboard](./Keyboard.md)
- [Color Display Adapter](./CDA.md) (CDA)
- [Computer Architecture Diagram](./computer.dia) (DIA file)
- [Calling Conventions](./calling_convention.md)

REFERENCE IMPLEMENTATION
------------------------
See [TR3200-VM](https://github.com/trillek-team/trillek-tr3200-vm). Also, can run in you [browser](http://cpu.zardoz.es).
