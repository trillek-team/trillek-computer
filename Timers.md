Programmable Interval Timer (PIT)
================================
Version 0.1c (WIP) 

The Programmable Interval Timer includes two 32 bit timers capable of generating
a interrupt to the CPU. Allows precise timings and periodic interrupts.
Uses CPU clock as source.

RESOURCES
---------

- Interrupt Message = 0x0001 if TMR0 does underflow
- Interrupt Message = 0x1001 if TMR1 does underflow
- Address 0x113040 (Read dword) : TMR0_VAL
- Address 0x113044 (Write dword): TMR0_RELOAD
- Address 0x113048 (Read dword) : TMR1_VAL
- Address 0x11304C (Write dword): TMR1_RELOAD
- Address 0x113050 (Write/Read byte)     : TMR_CFG


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

EXAMPLE OF USE
==============

### System Clock
This example illustrates how setup Timer 0 to generate interrupts every 50 ms (20
Hz rate), and update a clock ram var to obtain seconds passing since the Timer 0 was setup.


    ; Setup the Timer 0 as System Clock at a rate of 20 Hz (50 ms)
    OVERLOAD .EQU 5000
    
    LOAD.B 0x110050, %r0                        ; Recover the previous Timer setup
    OR %r0, %r0, 3                              ; Enables Timer 0 and Timer 0 interrupt
    SAVE.B 0x110050, %r0
    SET %r0, OVERLOAD
    SAVE.W 0x110044, %r0                        ; Sets the reload value (time measured from here)
    ...
    
    
    ; At Timers ISR
    ...
    BEQ %r0, 0x00000001                         ; Timer 0 interrupt
        CALL system_clock
    ...
    RFI
    ...
    
    
    ; Timer 0 ISR
    system_clock:
        PUSH %r1                                ; Preserve used registers
        PUSH %r2

        LOAD clk_seconds, %r0                   ; Load vars from RAM
        LOAD clk_tickst, %r1
        
        ADD %r0, 1, %r0                         ; Increment seconds counter

        LOAD 0x110040, %r2                      ; Read how many clock ticks happen
        SUB OVERLOAD, %r2, %r2                  ; Ticks = overload - value
        ADD %r1, %r2, %r1                       ; Increments ticks counter
        BUL %r1, OVERLOAD                       ; If Ticks not overload skip and store clock state
            JMP system_clock_save_state
        SUB %r1, %r1, OVERLOAD                  
        ADD %r0, 1, %r0                         ; Ticks > 5000, Increments seconds counter
        ; Note that this value should a bit bigger to compensate the ISR execution time
        
    system_clock_save_state:
        STORE clk_seconds, %r0
        STORE clk_ticks, %r1

        POP %r2                                 ; Recovers %r1 and %r2 values
        POP %r1
        RET
        
    ; RAM vars
    clk_seconds: .dd 0                          ; Seconds counter
    clk_ticks:   .dd 0                          ; Ticks counter (allow compensate ISR latency)

