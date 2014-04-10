---
---
Beeper
================================
Version 0.2a

The Beeper device (or "Buzzer") consist in a basic multi-vibrator oscillator 
that allow to change the oscillation frequency in function of a 16 bit register.

RESOURCES
---------

- Address 0x11E020 (Write word): BEEP_FRQ


OPERATION
---------

Writing at BEEP_FRQ, sets the output frequency from 0 to 16383Hz. Values over
16383Hz are ignored. Setting at 0Hz stops the sound generation.

