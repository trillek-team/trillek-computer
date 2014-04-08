Random Number Generator
=======================
Version 0.1

The Random Number Generator (**RNG**) is a device that can generate pseudo random
32 bit numbers from a seed.

IMPLEMENTATION
--------------
Reading and writing to the address calls rand() and srand().

RESOURCES
---------
Memory Address:
0x11E040: (Write DWord) Set the PRNG seed.
0x11E040: (Read DWord)  Get a 32 bit pseudo random number.