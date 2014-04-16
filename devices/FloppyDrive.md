---
layout : default
title : Floppy Drive
cat : Devices
---
Mackapar 5.25" Floppy Drive
===========================
Version 0.2

                                      .!.
                                     !!!!!. 
                                  .   '!!!!!. 
                                .!!!.   '!!!!!.
                              .!!!!!!!.   '!!!!!.
                            .!!!!!!!!!'   .!!!!!!!.
                            '!!!!!!!'   .!!!!!!!!!'
                              '!!!!!.   '!!!!!!!' 
                                '!!!!!.   '!!!'
                                  '!!!!!.   '
                                    '!!!!! 
                                      '!'   


                          M A C K A P A R    M E D I A   

Name: Mackapar 5.25" Floppy Drive (M5FDD) 

 - Device Types    : 0x08 (Mass Storage Device)
 - Device SubType  : 0x01 (Floppy Drive)
 - Device Builder  : 0x1EB37E91 (Mackapar Media)
 - Device ID       : 0x01

The Mackapar 5.25" Floppy Drive is compatible with all standard 5.25" floppy 
disks.
The M5FDD works is asynchronous, and has a raw read/write speed of 100kbit/s. 

### Floppy geometry

The floppies are divided in sides, tracks and sectors (CHS). Can ahve 1 or 2 
sides, or *Heads*, 40 or 80 tracks, or *Cylinders*, and 8 to 36 sectors per 
track. Each sector have asize of 512 Bytes. To calculate the floppy size 
based in his CHS paramters, we do a simple arithmetic :
```
  Size = Sides *  Tracks  * Sectors per Track * 512 B
       = Heads * Cylindes * Sectors per Track * 512 B
```
By nomenclature and convention, *Heads* and *Cyclenders* begin in 0, and 
*Sectors* begin to count from 1. 

When the floppy does changes of Track, does a Track seeking. TRack seeking 
takes a time of 3 ms per track. For example, if the floppay was on track 5, and 
changes to track 20, takes a time of `3 ms * (20-5) = 45 ms` 

### List of usual floppy formats

 Sides | Sectors Per Track | Tracks Per Side | Capacity 
:-----:|:-----------------:|:---------------:|:----------
   1   |        8          |       40        |  160 KiB
   2   |        8          |       40        |  320 KiB
   1   |        8          |       80        |  320 KiB
   2   |        9          |       40        |  360 KiB
   2   |        8          |       80        |  640 KiB
   2   |        9          |       80        |  720 KiB
   2   |        15         |       80        |  1200 KiB


COMMANDS and REGISTERS
----------------------
   
Reading E Register returns always the current error code.
Reading D Register returns always the current status code.

By convention, to store a CHS value in a register, we use this format:
```
    15 14 13 12 11 10 09 08 07 06 05 04 03 02 01 00
    -----------------------------------------------
     H  C  C  C  C  C  C  C  S  S  S  S  S  S  S  S  
```

 - 0x0000 : **SET_INT** :  
   Set interrupt. Enables interrupts and sets the message to A value if X is 
   anything other than 0. Disables interrupts if A value is 0. When interrupts 
   are enabled, the M5FDD will trigger an interrupt whenever the state or error
   codes changes (E register value).
 - 0x0001 : **READ** :  
   Reads the sector pointerd by the CHS value stored in register C and stores 
   in the RAM starting at address B:A.
   Reading is only possible if the state (D register value) is STATE_READY or 
   STATE_READY_WP before calling this command. D will be set to STATE_BUSY 
   when this command is being executed.
   When the command is executed and the floppy drive is ready, the M5FDD reads 
   the desired sector at a 100kbit/s, storing it in a internal buffer. When it 
   ends to read the data, copy the internal buffer data to the RAM doing a bulk
   DMA operation at a rate of 40KB/s (4 bytes every 10 device clock ticks -> 
   40000 Bytes per second), so a sector takes `0.04096s + 0.0128s ~= 0.0538s`.
   This allow to operate asynchronous and protects against partial reads.
   When the M3FDD ends the whole operation and does it correctly, will change D 
   and E registers to STATE_READY/STATE_READY_WP and ERROR_NONE.
 - 0x0002 : **WRITE** :  
   Writes the sector pointed by the CHS valued stored in C reading it from the 
   RAM at address B:A.
   Writing is only possible if the state (D register value) is STATE_READY 
   before calling this command. D will be set to STATE_BUSY when this command 
   is being executed.
   When the command is executed and the floppy drive is ready, the M5FDD does a
   bulk DMA transfer from the RAM to his internal buffer at a rate of 40KB/s. 
   When ends to copy the data, then begin to write to the floppy disk from his 
   internal buffer at a rate of 100kbit/s. This allow to operate asynchronous.
   When the M3FDD end the whole operation and does it correctly, will change D 
   and E registers to STATE_READY and ERROR_NONE.
 - 0x0003 : **QUERY_MEDIA** :  
   Sets A register with the CHS value of the floppy geometry 
   is not present a floppy in the drive, then will return 0.

STATE CODES
-----------

 VALUE | NAME             | Description
:-----:|:-----------------|:------------------------------------------------------------  
0x0000 | STATE_NO_MEDIA   | There's no floppy in the drive.
0x0001 | STATE_READY      | The drive is ready to accept commands.
0x0002 | STATE_READY_WP   | Same as ready, except the floppy is write protected.
0x0003 | STATE_BUSY       | The drive is busy either reading or writing a sector.

ERROR CODES
-----------

 VALUE | NAME             | Description
:-----:|:-----------------|:------------------------------------------------------------    
0x0000 | ERROR_NONE       | There's been no error since the last poll.
0x0001 | ERROR_BUSY       | Drive is busy performing an action
0x0002 | ERROR_NO_MEDIA   | Attempted to read or write with no floppy inserted.
0x0003 | ERROR_PROTECTED  | Attempted to write to write protected floppy.
0x0004 | ERROR_EJECT      | The floppy was removed while reading or writing.
0x0005 | ERROR_BAD_SECTOR | The requested sector is broken, the data on it is lost.
0x0006 | ERROR_BAD_CHS    | The CHS value is not valid. Check QUERY_MEDIA.
0xFFFF | ERROR_BROKEN     | There's been some major software or hardware problem, try turning off and turning on the device again.


