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
0x03   uint8   VERSION  1        VCD format version
0x04   uint8   TYPE     <type>   Disk Type Enumeration
0x05   uint8   WP       0-1      (Boolean) Write protected flag
0x06   uint8   SIDES    1-255    Number of sides on disk
0x07   uint8   TRACKS   1-255    Tracks per Side
0x08   uint8   SECTORS  1-255    Sectors per Track
0x09   uint16  BYTES    1-65535  Bytes per Sector (must be power of 2)
```
Data
----
The first Side of the disk starts at 0x0B. This is stored as the raw disk data
that the virtual computer has written. The size of this region = SIDES*TRACKS*
SECTORS*BYTES in the header. This region can only be modified if the WP flag is
set to false in the header.

Bad Sectors
-----------
The bad sector map starts in the next byte directly after the last sector of 
Data. This is a bit field of flags determining if the corresponding data sector
is marked as bad. Attempting to write to a sector that is marked as bad will 
fail with an error. The size of this region is = (SIDES*TRACKS*SECTORS)/8. Each
byte contains 8 flags (bits).