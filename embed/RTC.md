---
layout : default
title : Real Time Clock
cat : Embed
---
Real Time Clock
===============
Version 0.1.2

The Real Time Clock (**RTC**) device can be polled for the current date and time of 
the universe. The clock does not raise any interrupts, so it is the job of software 
to keep itself synced with universe time.

RESOURCES
---------
Memory Address:

```
  0x11E030: (Byte) Seconds (0-59)
  0x11E031: (Byte) Minutes (0-59)
  0x11E032: (Byte) Hours (0-23)
  0x11E033: (Byte) Days (0-31)
  0x11E034: (Byte) Months (1-12)
  0x11E035: (Word) Year(0-65535)
```

IMPLEMENTATION
--------------
The clock returns the real life UTC server time plus as offset to set it to the 
time were the gamelore say that happens all (ie if the lore say, for example, 
that we are on 2424, then we should add 110 to the year) 

