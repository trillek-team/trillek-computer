5.25" Floppy Drive
==================
Version 0.1

Name: Mackapar 5.25" Floppy Drive (M5FDD) 

 - Device Class    : 0x08 (Mass Storage Device)
 - Device Builder  : 0x1EB37E91 (Mackapar Media)
 - Device ID       : 0x01 (5.25" Floppy drive) 
 - Device Rev      : 0x00  

The Mackapar 5.25" Floppy Drive is compatible with all standard 5.25" 1200 KiB 
and 360KiB floppy disks. The floppies uses sectors of 512 bytes.
The M5FDD works is asynchronous (does a DMA operation), and has a raw read/write
speed of 160kbit/s. This means that transfer a dword by DMA every 25 device 
clock ticks. 

The floppies are divided in tracks, having the 1200KiB floppies a total of 
80 tracks with each track 30 sectors (512*30*80=1200KiB), and the 360KiB 
floppies a total of 40 tracks with each track 24 sectors.
Track seeking time is about 2.4 ms per track.

COMMANDS
--------
   
Reading D Register returns always the current status code.
Reading E Register returns always the current error code.

 - 0x0000 : **SET_INT** :  
	 Set interrupt. Enables interrupts and sets the message to A value if X is 
	 anything other than 0. Disables interrupts if A value is 0. When interrupts 
	 are enabled, the M5FDD will trigger an interrupt whenever the state or error
	 codes changes (C and D values).
 - 0x0001 : **READ_SECTOR** :  
   Reads sector C and stores in the RAM starting at address B:A.
   Sets D to 1 if reading is possible and has been started, anything else if it
   fails. Reading is only possible if the state (D register value) is 
	 STATE_READY or STATE_READY_WP.
   Protects against partial reads.
 - 0x0002 : **WRITE_SECTOR** :  
   Writes sector C reading it from the RAM at address B:A.
   Sets D to 1 if writing is possible and has been started, anything else if it
   fails. Writing is only possible if the state (D register value) is 
	 STATE_READY.
   Protects against partial writes.
 - 0x0003 : **QUERY_MEDIA** :
   Sets A register with the number of total sectors that have the floppy. If 
	 is not present a floppy in the drive, then will return 0.

STATE CODES
-----------
  
 - 0x0000 STATE_NO_MEDIA   There's no floppy in the drive.
 - 0x0001 STATE_READY      The drive is ready to accept commands.
 - 0x0002 STATE_READY_WP   Same as ready, except the floppy is write protected.
 - 0x0003 STATE_BUSY       The drive is busy either reading or writing a sector.

ERROR CODES
-----------
    
 - 0x0000 ERROR_NONE       There's been no error since the last poll.
 - 0x0001 ERROR_BUSY       Drive is busy performing an action
 - 0x0002 ERROR_NO_MEDIA   Attempted to read or write with no floppy inserted.
 - 0x0003 ERROR_PROTECTED  Attempted to write to write protected floppy.
 - 0x0004 ERROR_EJECT      The floppy was removed while reading or writing.
 - 0x0005 ERROR_BAD_SECTOR The requested sector is broken, the data on it is lost.
 - 0xffff ERROR_BROKEN     There's been some major software or hardware problem,
                           try turning off and turning on the device again.




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


