Debug Serial Console
=====================================
Version 0.1 (WIP)

This device is only for debuging the Virtual Computer. NOT WILL BE IN GAME.

COMMANDS
--------

 - 0x0000 : **READ_BYTE** :
   Sets register A to the new read byte from the serial console link. If there 
   is nothing to read, then sets A to 0.
 - 0x0001 : **SEND_BYTE** :
   Reads register A and send a byte to the serial console link.
 - 0x0002 : **SET_RXINT** :  
   Sets interrupt RX message to A register value. If A is 0x0000, disables RX 
   interrupt. At boot/reset time the RX insterrupt is disabled. RX interrupt is
   generate when a new byte is ready to be read with **READ_BYTE** command.


