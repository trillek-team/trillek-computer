---
layout : default
title : FAT boot sector
cat : Formats
---

Information of a FAT filesystem boot sector
===========================================

Structure
---------

```
Bytes   Content
0-2     Jump to bootstrap (E.g. eb 3c 90; on i86: JMP 003E NOP. One finds either eb xx 90, or e9 xx xx.  The position of the bootstrap varies.)
3-10    OEM name/version (E.g. "IBM  3.3", "IBM 20.0", "MSDOS5.0", "MSWIN4.0".  Various format utilities leave their own name, like "CH-FOR18". Sometimes just garbage. Microsoft recommends "MSWIN4.1".)
        /* BIOS Parameter Block starts here */
11-12   Number of bytes per sector (512). Must be one of 512, 1024, 2048, 4096.
13      Number of sectors per cluster (1). Must be one of 1, 2, 4, 8, 16, 32, 64, 128.
        A cluster should have at most 32768 bytes. In rare cases 65536 is OK.
14-15   Number of reserved sectors (1) FAT12 and FAT16 use 1. FAT32 uses 32.
16      Number of FAT copies (2)
17-18   Number of root directory entries (224). 0 for FAT32. 512 is recommended for FAT16.
19-20   Total number of sectors in the filesystem (2880) (in case the partition is not FAT32 and smaller than 32 MB)
21      Media descriptor type (f0: 1.4 MB floppy, f8: hard disk; see below)
22-23   Number of sectors per FAT (9) 0 for FAT32.
24-25   Number of sectors per track (12)
26-27   Number of heads (2, for a double-sided diskette)
28-29   Number of hidden sectors (0). Hidden sectors are sectors preceding the partition.
        /* BIOS Parameter Block ends here */
30-509  Bootstrap code (or empty if is not bootable)
510-511 Signature 0x55 0xAA
```
The signature is found at offset 510-511. This will be the end of the sector only in case the sector size is 512.

Media descriptor byte
---------------------

The ancient media descriptor type codes are:

For 8" floppies: fc, fd, fe - Various interesting formats

For 5.25" floppies:
```
Value  DOS version  Capacity  sides  tracks  sectors/track
ff     1.1           320 KB    2      40      8
fe     1.0           160 KB    1      40      8
fd     2.0           360 KB    2      40      9
fc     2.0           180 KB    1      40      9
fb                   640 KB    2      80      8
fa                   320 KB    1      80      8
f9     3.0          1200 KB    2      80     15
```

For 3.5" floppies:
```
Value  DOS version  Capacity  sides  tracks  sectors/track
fb                   640 KB    2      80      8
fa                   320 KB    1      80      8
f9     3.2           720 KB    2      80      9
f0     3.3          1440 KB    2      80     18
f0                  2880 KB    2      80     36
```

For RAMdisks: fa

For hard disks:
```
Value  DOS version
f8     2.0
```
This code is also found in the first byte of the FAT.
IBM defined the media descriptor byte as 11111red, where r is removable, e is eight sectors/track, d is double sided.

Cluster size
------------

The default number of sectors per cluster for floppies (with FAT12) is
```
Drive size Secs/cluster   Cluster size
   360 KB       2             1 KiB
   720 KB       2             1 KiB
   1.2 MB       1           512 bytes
  1.44 MB       1           512 bytes
  2.88 MB       2             1 KiB
The default number of sectors per cluster for fixed disks is (with FAT12 below 16 MB, FAT16 above):
Drive size Secs/cluster   Cluster size
 <  16 MB       8             4 KiB
 < 128 MB       4             2 KiB
 < 256 MB       8             4 KiB
 < 512 MB      16             8 KiB
 <   1 GB      32            16 KiB
 <   2 GB      64            32 KiB
 <   4 GB     128            64 KiB   (Windows NT only)
 <   8 GB     256           128 KiB   (Windows NT 4.0 only)
 <  16 GB     512           256 KiB   (Windows NT 4.0 only)
Usually people have file systems with average file size a few KB. With a cluster size of 64 KiB or more the amount of slack (wasted space) can easily become more than half the total space. FAT32 allows larger file systems with a small cluster size:
Drive size Secs/cluster   Cluster size
 < 260 MB       1           512 bytes
 <   8 GB       8             4 KiB
 <  16 GB      16             8 KiB
 <  32 GB      32            16 KiB
 <   2 TB      64            32 KiB
```

FAT16
-----

FAT16 uses the above BIOS Parameter Block, with some extensions:
```
11-27   (as before)
28-31   Number of hidden sectors (0)
32-35   Total number of sectors in the filesystem
        (in case the total was not given in bytes 19-20)
36      Logical Drive Number (for use with INT 13, e.g. 0 or 0x80)
37      Reserved (Earlier: Current Head, the track containing the Boot Record)
        Used by Windows NT: bit 0: need disk check; bit 1: need surface scan
38      Extended signature (0x29)
        Indicates that the three following fields are present.
        Windows NT recognizes either 0x28 or 0x29.
39-42   Serial number of partition
43-53   Volume label or "NO NAME    "
54-61   Filesystem type (E.g. "FAT12   ", "FAT16   ", "FAT     ", or all zero.)
62-509  Bootstrap
510-511 Signature 55 aa
```

FAT32
-----

FAT32 uses an extended BIOS Parameter Block:
```
11-27   (as before)     
28-31   Number of hidden sectors (0)
32-35   Total number of sectors in the filesystem
36-39   Sectors per FAT
40-41   Mirror flags
        Bits 0-3: number of active FAT (if bit 7 is 1)
        Bits 4-6: reserved
        Bit 7: one: single active FAT; zero: all FATs are updated at runtime
        Bits 8-15: reserved
42-43   Filesystem version
44-47   First cluster of root directory (usually 2)
48-49   Filesystem information sector number in FAT32 reserved area (usually 1)
50-51   Backup boot sector location or 0 or 0xffff if none (usually 6)
52-63   Reserved
64      Logical Drive Number (for use with INT 13, e.g. 0 or 0x80)
65      Reserved - used to be Current Head (used by Windows NT)
66      Extended signature (0x29)
        Indicates that the three following fields are present.
67-70   Serial number of partition
71-81   Volume label
82-89   Filesystem type ("FAT32   ")
```

Source
------
http://www.win.tue.nl/~aeb/linux/fs/fat/fat-1.html
