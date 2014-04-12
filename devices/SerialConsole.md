---
layout : default
title : Debug Serial Console
cat : Devices
---
Debug Serial Console
=====================================
Version 0.1a (WIP)

This device is only for debuging the Virtual Computer. NOT WILL BE IN GAME.

 - Device Type     : 0x02 (Communications device)
 - Device SubType  : 0xFF (Serial Console)
 - Device Builder  : 0x00000000
 - Device ID       : 0x01

COMMANDS
--------

 - 0x0000 : **READ_WORD** :
   Sets register A to the new read word from the serial console link. If there 
   is nothing to read, then sets A to 0.
 - 0x0001 : **SEND_WORD** :
   Reads register A and send a word to the serial console link.
 - 0x0002 : **SET_RXINT** :  
   Sets interrupt RX message to A register value. If A is 0x0000, disables RX 
   interrupt. At boot/reset time the RX insterrupt is disabled. RX interrupt is
   generate when a new byte is ready to be read with **READ_WORD** command.


