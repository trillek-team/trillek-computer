HARDWARE DEVICE ENUMERATOR
==========================

Device that allow to enumerate the hardware devices connected to the system.

Resources Used
--------------

- Ports 255 and 254 . Read/Write both ports as a 16 bit value in *Little Endian*. This means that port 254 contains the **LSB** byte and the port 255 contains the **MSB**

Commands
--------

     VALUE |    NAME     | BEHAVIOUR
    -------+-------------+---------------------------------------------------------
    0x0000 | GET-NUMBER  | Reading port 254, returns the number of devices attached
           |             | from 0 to 255.
    -------+-------------+---------------------------------------------------------
    0x01xx | GET-CLASS   | Reading port 254, returns the Hardware class of the 
           |             | device xx attached. Will be 0 is xx correspond to a not 
           |             | attached device.
    0x02xx | GET-BUILDER | Reading port 254-255, returns the Hardware Builder of  
           |             | the device xx attached. Will be 0 is xx correspond to a 
           |             | not attached device.
    0x03xx | GET-ID      | Reading port 254-255, returns the Hardware ID of the 
           |             | device xx attached. Will be 0 is xx correspond to a not 
           |             | attached device.
    0x04xx | GET-VERSION | Reading port 254-255, returns the Hardware Version of
           |             | the device xx attached. Will be 0 is xx correspond to a 
           |             | not attached device.
    -------+-------------+---------------------------------------------------------
    0x1yxx | SET-JMP-1   | Set jumper 1 of device xx to the value y (0 to 15)
    0x2yxx | SET-JMP-2   | Set jumper 2 of device xx to the value y (0 to 15)
    0x3yxx | GET-JMP-1   | Reading port 254, returns the jumper 1 value of device xx
    0x4yxx | GET-JMP-2   | Reading port 254, returns the jumper 1 value of device xx
           
           
The quadruple of {Class, Builder, ID, Version}, identify a **single specific hardware device**
 . A device with the same {Class, Builder, ID} but different Version, is expect to share some kind backwards compatibility. 
 
The value Jumpers 1 and 2 allows, if the device implements it, to change the device configuration of resources used.

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

Pseudo Code -> Generates arrays with each device information
-----------
    byte n_devices
    byte dev_class[255]
    word dev_build[255]
    word dev_id[255]
    word dev_ver[255]
    
    OUT 0x00, 254
    OUT 0x00, 255
    INP [n_devices], 254
    for i = 0 to n_devices
        ; Grabs Device class of device X
        OUT i,    254
        OUT 0x01, 255
        INP dev_class[i], 254
        
        ; Grabs Device Builder of device X
        OUT i,    254
        OUT 0x02, 255
        INP dev_build[i], 255
        dev_build[i] = dev_build[i] << 8 
        INP dev_build[i], 254         
        
        ; Grabs Device ID of device X
        OUT i,    254
        OUT 0x03, 255
        INP dev_id[i], 255
        dev_id[i] = dev_id[i] << 8 
        INP dev_id[i], 254         
        
        ; Grabs Device Version of device X
        OUT i,    254
        OUT 0x04, 255
        INP dev_ver[i], 255
        dev_ver[i] = dev_ver[i] << 8 
        INP dev_ver[i], 254    
