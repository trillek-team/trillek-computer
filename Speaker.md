Speaker Driver
================================
Version 0.1 

The Sepaker Driver consistns in a square wave generator with choosable frecuency

RESOURCES
---------

- Address 0xFF000028 (Write word): SPKR_FRQ


OPERATION
---------

Writing at SPKR_FRQ, sets the output frequnecy from 0 to 16383Hz. Values over
16383Hz are ignored. Setting at 0Hz stops the sound gneration.

