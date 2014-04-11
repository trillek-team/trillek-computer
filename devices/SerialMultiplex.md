Serial Multiplex Device
=====================================
Version 0.1a 

This device outlines the basic specifications for communicating between multiple DataPort devices.

 - Device Type     : 0x04 (Expansion bus device)
 - Device SubType  : 0x01 (Serial Bus)
 - Device Builder  : 0xA87C900E (KaiComm)
 - Device ID       : 0x01 (Serial Multiplex)

COMMANDS
--------
For commands 0x0000 - 0x0001:
   B - Word
   A - DataPort ID
 - 0x0000 : **READ_WORD**:
   Set register B to the word read from the serial dataport as speificed by register A.
   If nothing to read, B set to 0
 - 0x0001 : **SEND_WORD** :
   Reads register B for the word to send, Reads register A for the DataPort to send the word on.
 - 0x0002 : **READ_PORTC** :
   Sets register A to the number of ports available.
 - 0x0003 : **GET_PORT_INFO** :
   Reads register A for the PortID, Sets register A current port status.
 - 0x0004 : **SET_RXINT** :  
   Sets interrupt RX message to A register value. If A is 0x0000, disables RX 
   interrupt. RX interrupt is generated when a new byte is ready to be read with **READ_WORD** command.

STATUS CODES
--------
0x0000 STATE_DISCONNECT Port is currently not connected to any device.

0x0001 STATE_UNIDIRECT Port is currently only connected locally, no remote device

0x0002 STATE_READY Port is currently connected locally and remotely. Communication is possible.

