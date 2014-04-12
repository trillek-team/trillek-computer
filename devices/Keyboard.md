---
layout : default
title : Generic Keyboard
cat : Devices
---
Generic Western/Latin Keyboard controller
================
Version 0.6a (WIP)

Generic Keyboard controller. Handles an internal buffer to store key events.

 - Device Type     : 0x03 (HID)
 - Device SubType  : 0x01 (Western/Latin Keyboard)
 - Device Builder  : 0x00000000
 - Device ID       : 0x01 (Generic Keyboard standard)


RESOURCES
---------

The keyboard controller uses few commands to see the status of the keyboard,
the key buffer and manipulate the key buffer.

COMMANDS
--------


 - 0x0000 : **CLR-BUFF** :
   Clears the key buffer.
 - 0x0001 : **PULL-KEY** :
   Pull the key buffer. Sets C register to the status bits of the key event, 
   sets B register to the scancode and sets A register to key-code (Latin-1 
   encoding).
 - 0x0002 : **PUSH-KEY** :
   Push the key buffer. C register must have the status bits, B register must 
   be the scancode and A register must have the key-code.
 - 0x0003 : **SET-INT** :
   Sets interrupt message to A register value. If A is 0x0000, then disables
   Key event interrupt. This is set to 0 when the computer boots.

HOW WORKS
---------

The Keyboard controller have a internal ring buffer that stores a
"keyboard event" when a user types a key. Each entry of the buffer contains
information of the key pressed and if was any function key pressed at same time.
When we pull a key event from the buffer, we get the **scancode** and the **key-code** 
of the event, plus status bits.

**Scancodes** represents what physical physical key was pressed. This scancodes 
are usually refereed against a US English keyboard for reference. For example 
scancode 0x002F is the key that in a US keyboard is a semicolon symbol, so we 
call these key "semicolon". Scancodes are 16 bit values.

**Key-codes** represents the translation of the pressed key to [Latin-1](http://en.wikipedia.org/wiki/ISO/IEC_8859-1)
(8bit ASCII) applying on it effects like Shift, uppercasing, localization of the
keyboard layout (for example : 'ñ' or 'ç'), etc... Key-codes are usually more 
useful when we are processing text input. Key-codes are 8 bit values.

In addition to the commands, reading **E hardware register** returns always 
the number of key events stored in the key buffer.  

### Status Bits

Status bits (D register and C register value on PULL/PUSH-KEY) have this format :

    15 14 13 12 11 10  9  8  7  6  5  4  3  2  1  0
    -----------------------------------------------
     0  0  0  0  0  0  0  0  0  0  0  0  0  G  C  S

Were **G** is 1 if "Alt. Gr." key is pressed, **C** is 1 if "Control" key is pressed
 and **S** is 1 if "Shift" key is pressed.


### Events buffer
The buffer can store at least 64 events. Each time that a key is typed, the
appropriate event is pushed to the buffer.
The buffer operates in FIFO fashion. So when a user types a key, a new event is
inserted in the buffer. Using **PULL-KEY** extracts the most old event stored 
in the buffer (POP buffer), but using **PUSH-KEY** inserts a new event in the 
buffer like if it was the oldest entry (PUSH buffer). If there is not enough 
space, pushing fails silently. When the buffer fills and the keyboard inserts a
new event, the most oldest event is removed to allow to insert a new event.

    PUSH inserts here
    ----------------->
                       |---|
    -----------------> |   | Oldest event
    POP extract this   |---|
                       |   |
                       |---|
                       .   .
                       .   .
                       .   .
                       |---|
                       |   | Newest event
                       |---|
                             <----------------------
                               Keyboard inserts here


### Scancodes list

 - KEY_UNKNOWN       0xFFFF
 - KEY_NULL          0    -> Read value when the buffer is empty
 - KEY_SPACE         32
 - KEY_APOSTROPHE    39   -> '
 - KEY_COMMA         44   -> ,
 - KEY_MINUS         45   -> -
 - KEY_PERIOD        46   -> .
 - KEY_SLASH         47   -> /
 - KEY_0             48
 - KEY_1             49
 - KEY_2             50
 - KEY_3             51
 - KEY_4             52
 - KEY_5             53
 - KEY_6             54
 - KEY_7             55
 - KEY_8             56
 - KEY_9             57
 - KEY_SEMICOLON     59   -> ;
 - KEY_EQUAL         61   -> =
 - KEY_A             65
 - KEY_B             66
 - KEY_C             67
 - KEY_D             68
 - KEY_E             69
 - KEY_F             70
 - KEY_G             71
 - KEY_H             72
 - KEY_I             73
 - KEY_J             74
 - KEY_K             75
 - KEY_L             76
 - KEY_M             77
 - KEY_N             78
 - KEY_O             79
 - KEY_P             80
 - KEY_Q             81
 - KEY_R             82
 - KEY_S             83
 - KEY_T             84
 - KEY_U             85
 - KEY_V             86
 - KEY_W             87
 - KEY_X             88
 - KEY_Y             89
 - KEY_Z             90
 - KEY_LEFT_BRACKET  91  -> [
 - KEY_BACKSLASH     92  -> \
 - KEY_RIGHT_BRACKET 93  -> ]
 - KEY_GRAVE_ACCENT  96  -> `
 - KEY_WORLD_1       161 -> non-US #1
 - KEY_WORLD_2       162 -> non-US #2
 - KEY_ESCAPE        256
 - KEY_ENTER         257
 - KEY_TAB           258
 - KEY_BACKSPACE     259
 - KEY_INSERT        260
 - KEY_DELETE        261
 - KEY_RIGHT         262
 - KEY_LEFT          263
 - KEY_DOWN          264
 - KEY_UP            265
 - KEY_LEFT_SHIFT    340
 - KEY_LEFT_CONTROL  341
 - KEY_LEFT_ALT      342
 - KEY_RIGHT_SHIFT   344
 - KEY_RIGHT_CONTROL 345
 - KEY_RIGHT_ALT     346

### Keys list

Key codes are:

 - 0x00: No input (buffer is empty)
 - 0x01: Unknown key
 - 0x05: Delete
 - 0x06: Alt key
 - 0x08: Backspace
 - 0x09: Tabulator
 - 0x0D: Return
 - 0x0E: Shift
 - 0x0F: Control
 - 0x10: Insert
 - 0x12: Arrow up
 - 0x13: Arrow down
 - 0x14: Arrow left
 - 0x15: Arrow right
 - 0x1B: Escape
 - 0x20: Space
 - 0x21 to 0x7E: Latin-1 characters
 - 0xA0 to 0xFF: Latin-1 characters
 - Other values: Reserved for advanced keyboards or non western localized
   keyboards


Stuff to read
=============

 - http://retired.beyondlogic.org/keyboard/keybrd.htm
 - flint.cs.yale.edu/cs422/doc/art-of-asm/pdf/CH20.PDF
 - http://bos.asmhackers.net/docs/keyboard/docs/KeyboardFAQ.txt
 - https://groups.google.com/forum/#!topic/comp.os.msdos.programmer/pAeBQo_htYo
 - http://en.wikipedia.org/wiki/ISO/IEC_8859-1

