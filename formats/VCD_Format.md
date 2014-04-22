---
layout : default
title : VCD
cat : Formats
---

Virtual Computer Disk Format
============================
Version 1

The Virtual Computer Disk (VCD) format is a container for a magnetic disk 
storage medium. The entire file is stored in little endian format.

Header
------
The header contains various metrics of the disk.
```
Offset Size    Name     Value    Description
----------------------------------------------------------------------
0x00   char[3] MAGIC    "VCD"    Magic Number
0x03   uint8   VERSION  1        VCD file format version
0x04   uint8   TYPE     <type>   Disk Type Enumeration
0x05   uint8   WP       0-1      (Boolean) Write protected flag
0x06   uint8   SIDES    1-255    Number of sides on disk
0x07   uint8   TRACKS   1-255    Tracks per Side
0x08   uint8   SECTORS  1-255    Sectors per Track
0x09   uint16  BYTES    1-65535  Bytes per Sector (must be power of 2)
```
The Magic Number of the file will always be the chars "VCD". The format of the 
file may not be backwards compatable with previous versions. The TYPE enum is
an indicator of what media the disk was originally created to mimic. The device 
opening the file should know what enum is expected and use it as error checking.
Possible values of the TYPE enum are in the table below.
```
Value  Media
-------------------------
'F'    Floppy Disk
'H'    Hard Disk
'N'    Nonvolatile Memory
'P'    Punch Card(s)
'T'    Magnetic Tape
```
The WP flag determines if the the Data region (described below) should be able to
be written to. The SIDES, TRACKS, SECTORS and BYTES fields relate to various 
metrics of the storage device. These fields determine the size of the Data region.

Data
----
The first Side of the disk starts at 0x0B. This is stored as the raw disk data
that the virtual computer has written. The size of this region = SIDES*TRACKS*
SECTORS*BYTES in the header. This region should only be modified if the WP flag
is set to false in the header.

Bad Sectors
-----------
The bad sector map starts in the next byte directly after the last sector of 
Data. This is a bit field of flags determining if the corresponding data sector
is marked as bad. Attempting to write to a sector that is marked as bad should 
fail with an error. The size of this region is = (SIDES*TRACKS*SECTORS)/8. Each
byte contains 8 flags (bits).
