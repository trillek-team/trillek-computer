---
layout : default
title : Programmable Interval Timers
---
Programmable Interval Timers (PIT)
================================
Version 0.1d (WIP) 

The Programmable Interval Timer includes two 32 bit timers capable of generating
a interrupt to the CPU. Allows precise timings and periodic interrupts.
Uses CPU clock as source.

RESOURCES
---------

- Interrupt Message = 0x0001 if TMR0 does underflow
- Interrupt Message = 0x1001 if TMR1 does underflow
- Address 0x11E000 (Read dword) : TMR0_VAL
- Address 0x11E004 (Write dword): TMR0_RELOAD
- Address 0x11E008 (Read dword) : TMR1_VAL
- Address 0x11E00C (Write dword): TMR1_RELOAD
- Address 0x11E010 (Write/Read byte)     : TMR_CFG


OPERATION
---------

Reading at TMRx_VAL, gets the actual Timer x value.

Writing at TMRx_RELOAD, changes Timer x reload value when overflows.

Writing at TMR_CFG, changes the operation mode of both timers. Accepts this
format :

- Bit 0 : Enables Timer 0 if is 1   
- Bit 1 : Enables underflow interrupt of Timer 0 if is 1    
- Bit 2-3 : Unused
- Bit 4 : Enables Timer 1 if is 1   
- Bit 5 : Enables underflow interrupt of Timer 1 if is 1    
- Bit 6-7 : Unused


HOW WORKS
=========

Each timer consists in a value register, TMRx_VAL, and a reload register,
TMRx_RELOAD. When a timer is enabled, in each internal clock tick, decreases the
TMRx_VAL by one. When does a underflow (0-1 = 0xFFFF), sets TMRx_VAL to
TMRx_RELOAD value. If Interrupt is enabled for Timer X, when the underflow
happens, will send a interrupt signal.

Writing at TMRx_RELOAD, overwrites TMRx_VAL with the same value.

The operation of each Timer, allow to work as frequency divisor setting
TMRx_RELOAD to the appropriated value. For example, to obtain periodic interrupts
each second (1Hz) in Timer 0 with a CPU of 100KHz you only need to set bit 0 and bit 1 in TMR_CFG
to 1, and set TMR0_RELOAD to 100000. To get a periodic interruption of the Timer
1 at a rate of 20Hz, you only need to set bit 4 and 5 to 1, and set TMR1_RELOAD
to 5000.

Values of TMRx_RELOAD under 100 aren't recommended, also note that value 0
disables the Timer.

For measuring precise times, you can set a Timer to a periodic interval that you
consider appropriate, and in the ISR read TMRx_VAL to measure the time since the
underflow period happen and the ISR executed.

