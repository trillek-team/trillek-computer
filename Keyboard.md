Generic Keyboard controller
================
Version 0.5 (WIP) 

Generic Keyboard controller. Handles an internal buffer to store key events.

- Device Class    : 0x03 (HID)
- Device Build    : 0xXXXX
- Device ID       : 0x0001 (Generic Keyboard)
- Device Version  : 0xXXXX


RESOURCES
---------

The keyboard controller uses few registers to see the status of the keyboard, the key buffer and send commands. Also, have a configuration *Jumper*.

- Event interrupt message = in function of *Jumper* value, will be :
    - 1 -> 0x00000109
    - 2- > 0x00000209
    - 3 -> 0x00000309

### Preferred Address Block
The keyboard controller try to use this address blocks:

- Address 0x110160 (Read/Write word): KEY_REG
- Address 0x110162 (Read/Write byte): KEY_CMD

HOW WORKS
---------

The Keyboard controller have a internal ring buffer that stores a "keyboard event" when a user types a key. Each entry of the buffer contains information of the key pressed and if was any function key pressed at same time. Also, there is two types of entries in the buffer: RAW events and KEY events.

**RAW** events uses **scancodes** that represents what physical physical key was pressed. This scancodes are usually refereed against a US English keyboard for reference. For example scancode 0x02F is the key that in a US keyboard is a semicolon symbol, so we call these key "semicolon".

**KEY** events returns a [Latin-1](http://en.wikipedia.org/wiki/ISO/IEC_8859-1) (8bit ASCII) representation of the key pressed and apply on it effects like Shift, uppercasing, localization of the keyboard layout (for example : 'ñ' or 'ç'), etc...

KEY events are more useful for basic typing commands or text, when RAW events are more useful if you need to do actions with keys in a particular position.

Also, there is there modes of operation in the keyboard controller : RAW mode, KEY mode, and RAW+KEY mode. In RAW mode, each time that a user type a key in the keyboard, only push in the buffer a RAW events. In KEY mode, each time that a user type a key in the keyboard, only push in the buffer a KEY event. And finally, in RAW+KEY mode, each time that a user type in the keyboard, only push a RAW event followed by a KEY event.

OPERATION
---------

Reading at KEY_REG, gets the last event.
Reading value from KEY_CMD depends of the command.

Writing at KEY_REG, push a event to the keyboard buffer
Writing at KEY_CMD, sends a command to the keyboard.

### Commands

Accepted values to write in KEY_CMD are in this list :

     VALUE |  NAME      | BEHAVIOUR
    -------+------------+----------------------------------------------------------
      0x00 | CLEAR      | Clear keyboard buffer
      0x01 | COUNT      | Reading KEY_CMD returns the number of events stored in
           |            | the buffer when the command is send.
      0x02 | D-INT      | Disables interrupt when a Key event happens.
      0x03 | E-INT      | Enables interrupts when a Key event happens.
      0x04 | KEY-MODE   | Switch to KEY mode.
      0x05 | RAW-MODE   | Switch to RAW mode.
      0x06 | RK-MODE    | Switch to RAW+KEY mode.
    -------+------------+----------------------------------------------------------

### Events

If interrupts are enabled, the keyboard will trigger an interrupt when a keyboard event happens.

RAW Event Format (value read from KEY_REG):

    15 14 13 12 11 10 9  8  7  6  5  4  3  2  1  0
    ----------------------------------------------
    1  G  C  S  X  X  s  s  s  s  s  s  s  s  s  s
    
Where : 

 - G (Alt Gr mod) If the bit is set to 1, this means that the Alt Gr Key is being pressed at same time
 - C (Control mod) If the bit is set to 1, this means that the Control Key is being pressed at same time
 - S (Shift mod) If the bit is set to 1, this means that the Shift Key is being pressed at same time
 - X reserved for future use
 - ssssssssss 10 bit scancode
 
 
KEY Event Format (value read from KEY_REG):

    15 14 13 12 11 10 9  8  7  6  5  4  3  2  1  0
    ----------------------------------------------
    0  G  C  S  X  X  0  0  k  k  k  k  k  k  k  k

Where : 

 - G (Alt Gr mod) If the bit is set to 1, this means that the Alt Gr Key is being pressed at same time
 - C (Control mod) If the bit is set to 1, this means that the Control Key is being pressed at same time
 - S (Shift mod) If the bit is set to 1, this means that the Shift Key is being pressed at same time
 - X reserved for future use
 - kkkkkkkk Latin-1 representation of the key. 


### Events buffer
The buffer can store at least 64 events. Each time that a key is typed, the appropriate event is pushed to the buffer.
The buffer operates in FIFO fashion. So when a user types a key, a new event is inserted in the buffer. Reading KEY_REG, extracts the most old event stored in the buffer (POP buffer). But writing to KEY_REG, inserts a new event in the buffer like if was the oldest entry, in other words, PUSH a event to the buffer. Pushing could be do only, if the buffer have enough size to store it. If there is not enight space, then pushing fails silent. When the buffer fills and the keyboard inserts a new event, the most oldest event is removed to allow to insert the new event.

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

- KEY_UNKNOWN       0x3FF
- KEY_NULL          0    -> Read value when the buffer is empty    
- KEY_SPACE         32
- KEY_APOSTROPHE    39   -> '
- KEY_COMMA         44   -> ,
- KEY_MINUS         45   -> -
- KEY_PERIOD        46   -> .
- KEY_SLASH         47   -> /
- KEY_0   48
- KEY_1   49
- KEY_2   50
- KEY_3   51
- KEY_4   52
- KEY_5   53
- KEY_6   54
- KEY_7   55
- KEY_8   56
- KEY_9   57
- KEY_SEMICOLON     59   -> ;
- KEY_EQUAL         61   -> =
- KEY_A   65
- KEY_B   66
- KEY_C   67
- KEY_D   68
- KEY_E   69
- KEY_F   70
- KEY_G   71
- KEY_H   72
- KEY_I   73
- KEY_J   74
- KEY_K   75
- KEY_L   76
- KEY_M   77
- KEY_N   78
- KEY_O   79
- KEY_P   80
- KEY_Q   81
- KEY_R   82
- KEY_S   83
- KEY_T   84
- KEY_U   85
- KEY_V   86
- KEY_W   87
- KEY_X   88
- KEY_Y   89
- KEY_Z   90
- KEY_LEFT_BRACKET   91  /* [ */
- KEY_BACKSLASH      92  /* \ */
- KEY_RIGHT_BRACKET  93  /* ] */
- KEY_GRAVE_ACCENT   96  /* ` */
- KEY_WORLD_1        161 /* non-US #1 */
- KEY_WORLD_2        162 /* non-US #2 */
- KEY_ESCAPE         256
- KEY_ENTER          257
- KEY_TAB            258
- KEY_BACKSPACE      259
- KEY_INSERT         260
- KEY_DELETE         261
- KEY_RIGHT          262
- KEY_LEFT           263
- KEY_DOWN           264
- KEY_UP             265
- KEY_LEFT_SHIFT     340
- KEY_LEFT_CONTROL   341
- KEY_LEFT_ALT       342
- KEY_RIGHT_SHIFT    344
- KEY_RIGHT_CONTROL  345
- KEY_RIGHT_ALT      346

### Keys list

Key codes are:

- 0x00: No input (buffer is empty)
- 0x01: Unknow key

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
- 0x20: Spacebar

- 0x21 to 0x7E: Latin-1 characters
- 0xA0 to 0xFF: Latin-1 characters

- Other values: Reserved for advanced keyboards or non western localized keyboards


Integrated Keyboard controller in the motherboard
=================================================

The integrated keyboard controller in the motherboard is not listed in the hardware enumerator and not hate a *Jumper*. Uses this resources :

- Event interrupt message = 0x00000009
- Address 0x110060 (Read/Write word): KEY_REG
- Address 0x110062 (Read/Write byte): KEY_CMD


Stuff to read
=============

 - http://retired.beyondlogic.org/keyboard/keybrd.htm
 - flint.cs.yale.edu/cs422/doc/art-of-asm/pdf/CH20.PDF
 - http://bos.asmhackers.net/docs/keyboard/docs/KeyboardFAQ.txt
 - http://en.wikipedia.org/wiki/ISO/IEC_8859-1

