HARDWARE DEVICE ENUMERATOR
==========================
Version 0.2 (WIP) 

Device that allow to enumerate the hardware devices connected to the system.

Resources Used
--------------

- Address 0xFF000000 to 0xFF000002 (**HWN** register). To write/read a 16 bit value in *Little Endian*. 

Commands
--------
Write a value at **HWN** register to:


    | VALUE |    NAME     | BEHAVIOUR                                                |  
    |-------+-------------+----------------------------------------------------------|
    |0x0000 | GET-NUMBER  | Reading HWN, returns the number of devices attached from |
    |       |             | 0 to 32.                                                 |
    |-------+-------------+----------------------------------------------------------|
    |0x01xx | GET-CLASS   | Reading HWN, returns the Hardware class of the device xx |
    |       |             | attached. Will be 0 is xx correspond to a not attached   |
    |       |             | device.                                                  |
    |-------+-------------+----------------------------------------------------------|
    |0x02xx | GET-BUILDER | Reading HWN, returns the Hardware Builder of the device  |
    |       |             | xx attached. Will be 0 is xx correspond to a not         |
    |       |             | attached device.                                         |
    |-------+-------------+----------------------------------------------------------|
    |0x03xx | GET-ID      | Reading HWN, returns the Hardware ID of the device xx    |
    |       |             | attached. Will be 0 is xx correspond to a not attached   |
    |       |             | device.                                                  |
    |-------+-------------+----------------------------------------------------------|
    |0x04xx | GET-VERSION | Reading HWN, returns the Hardware Version of the device  |
    |       |             | xx attached. Will be 0 is xx correspond to a not         |
    |       |             | attached device.                                         |
    |-------+-------------+----------------------------------------------------------|
    |0x10xx | GET-JMP-1   | Reading HWN, returns the jumper 1 value of device xx     |
    |-------+-------------+----------------------------------------------------------|
    |0x20xx | GET-JMP-2   | Reading HWN, returns the jumper 2 value of device xx     |
    |-------+-------------+----------------------------------------------------------|
           
           
The quadruple of {Class, Builder, ID, Version}, identify a **single specific hardware device**
 . A device with the same {Class, Builder, ID} but different Version, is expect to share some kind backwards compatibility. 
 
The value Jumpers 1 and 2 allows, if the device implements it, to change the device configuration of resources used.
Jumpers are set physicalily in the device board.

Device Class values
-------------------

- 0x01 : Audio devices (Sound Cards)
- 0x02 : Communications devices
- 0x03 : HID (Human Interface Device)
- 0x06 : Image/Video Input Devices
- 0x07 : Printer (2D and 3D) Devices
- 0x08 : Mass Storage Device (Floppy drives, Hard disks, Tape recorders)
- 0x0E : Graphics Devices (Graphics card)
- 0x0F : HoloGraphics Devices
- 0x10 : Ship Sensors (DRADIS, Air, Hull integrity, etc...)
- 0x11 : Power Management Systems (Generators)
- 0x12 : Hydraulic/Pneumatic Actuators (Doors, air-locks, landing gears)
- 0x13 : Electric Engines (Wheels)
- 0x1A : Defensive Systems (Shields)
- 0x1B : Offensive Systems (Weapons)
- 0x1C : Sub-FTL Navigational and Engine Systems (Thrusters and Engines)
- 0x1D : FTL Navigational Systems (Warp Engines)

Example of Usage
================

### Generates arrays with each device information

    
    MOV %r0, 0
    STORE.W %0xFF000000, %r0
    LOAD.B %r1, 0xFF000002
    STORE.W n_devices, %r1                  ; Number of devices in n_devices and in %r1
    
    MOV %r5, 0                              ; Index in the table
    
    loop:
    LLS %r2, 1, %r5                         ; %r2 = %r5 x2 = ptr to the a word size element
    OR %r2, 0x0100, %r0                     ; Get Class of dev %r2
    STORE.W 0xFF000000, %r0
    LOAD.W %r3, 0xFF000000                  ; Read class and put in %r3
    STORE.W dev_class + %r2, %r3            ; Writes at %r2 class entry

    OR %r2, 0x0200, %r0                     ; Get Builder of dev %r2
    STORE.W 0xFF000000, %r0
    LOAD.W %r3, 0xFF000000                  ; Read class and put in %r3
    STORE.W dev_build + %r2, %r3            ; Writes at %r2 class entry
    
    OR %r2, 0x0300, %r0                     ; Get ID of dev %r2
    STORE.W 0xFF000000, %r0
    LOAD.W %r3, 0xFF000000                  ; Read class and put in %r3
    STORE.W dev_id + %r2, %r3               ; Writes at %r2 class entry
    
    OR %r2, 0x0400, %r0                     ; Get Version of dev %r2
    STORE.W 0xFF000000, %r0
    LOAD.W %r3, 0xFF000000                  ; Read class and put in %r3
    STORE.W dev_ver + %r2, %r3              ; Writes at %r2 class entry
            
    ADD %r5, 1 
    IFL %r5, %r1                            ; for(%r5=0; %r2 < num_devices; %r2++)
        JMP loop
    ...

    n_devices: .resb 1
    dev_class: .resw 32
    dev_build: .resw 32 
    dev_id:    .resw 32
    dev_ver:   .resw 32

