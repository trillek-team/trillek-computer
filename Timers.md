Programmable Interval Timer (PIT)
================================
Version 0.1 (WIP) 

The Programmable Interval Timer includes two 32 bit timers capable of generating
a interrupt to the CPU. Allows precise timings and periodic interrupts.
Uses an internal precision clock of 100KHz as clock base time.

RESOURCES
---------

- Interrupt Message = 0x00000000 if TMR0 does underflow
- Interrupt Message = 0x00000100 if TMR0 does underflow
- Address 0xFF000040 (Read word): TMR0_VAL
- Address 0xFF000040 (Write word): TMR0_RELOAD
- Address 0xFF000044 (Read word): TMR1_VAL
- Address 0xFF000044 (Write word): TMR1_RELOAD
- Address 0xFF000048 (Write/Read byte): TMR_CFG


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
each second (1Hz) in Timer 0, you only need to set bit 0 and bit 1 in TMR_CFG
to 1, and set TMR0_RELOAD to 100000. To get a periodic interruption of the Timer
1 at a rate of 20Hz, you only need to set bit 4 and 5 to 1, and set TMR1_RELOAD
to 5000.

Values of TMRx_RELOAD under 100 aren't recommended, also note that value 0
disables the Timer.

For measuring precise times, you can set a Timer to a periodic interval that you
consider appropriate, and in the ISR read TMRx_VAL to measure the time since the
underflow period happen and the ISR executed.

EXAMPLE OF USE
==============

### System Clock
This example illustrates how setup Timer 0 to generate interrupts every 50 ms (20
Hz rate), and update a clock ram var to obtain seconds transcurred since the
Timer 0 was setup.


    ; Setup the Timer 0 as System Clock at a rate of 20 Hz (50 ms)
    LOAD.B 0xFF000048, %r0                      ; Recover the previous Timer setup
    OR %r0, 3, %r0                              ; Enables Timer 0 and Timer 0 interrupt
    SAVE.B 0xFF000048, %r0
    SET %r0, 5000
    SAVE.W 0xFF000040, %r0                      ; Sets the reload value



    ;At General ISR
    ...
    BEQ %r0, 0                                  ; Timer 0 interrupt
        CALL system_clock
    ...
    ...
    RFI

    ; RAM vars
    clk_seconds: .dd 0                          ; Seconds counter
    clk_tickst:  .dd 0                          ; Ticks counter

    ; Timer 0 ISR
    system_clock:
        PUSH %r1                                ; Preserve used registers
        PUSH %r2

        LOAD clk_seconds, %r0                   ; Load vars from RAM
        LOAD clk_tickst, %r1
        
        ADD %r0, 1, %r0                         ; Increment seconds counter

        LOAD 0xFF000040, %r2                    ; Read ho0w many clock ticks happen
        SUB 5000, %r2, %r2                      ; Ticks = overload - value
        ADD %r1, %r2, %r1                       ; Increments ticks counter
        BUL %r1, 5000                           
            JMP system_clock_save_state
        SUB %r1, 5000, %r1                      ; Ticks > 5000, Increments seconds counter
        ADD %r0, 1, %r0                         ; To compesate ISR latency. 
        ; Note that this value should a bit bigger to compesate the ISR execution time
        
        system_clock_save_state:
        STORE clk_seconds, %r0
        STORE clk_ticks, %r0

        POP %r2                                 ; Recovers %r1 and %r2 values
        POP %r1
        RET

