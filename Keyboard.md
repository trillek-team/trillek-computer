Generic Keyboard
================
Version 0.4 (WIP) 

Generic Compatible Keyboard device. Handles an internal buffer to
store key events.

- Device Class    : 0x03 (HID)
- Device Build    : 0xXXXX
- Device ID       : 0x0001 (Generic Keyboard)
- Device Version  : 0xXXXX


Jumper 1 acepts values from 0 to 3

RESOURCES
---------

### Jumper 1 = 0

- KeyDown Interrupt Message = 0x00000009
- KeyUp Interrupt Message = 0x00000109
- Address 0xFF000060 (Read/Write word): KEY_REG
- Address 0xFF000062 (Read/Write byte): KEY_STATUS
- Address 0xFF000063 (Write byte): KEY_CMD

### Jumper 1 = 1

- KeyDown Interrupt Message = 0x00001009
- KeyUp Interrupt Message = 0x00001109
- Address 0xFF000160 (Read/Write word): KEY_REG
- Address 0xFF000162 (Read/Write byte): KEY_STATUS
- Address 0xFF000163 (Write byte): KEY_CMD

### Jumper 1 = 2

- KeyDown Interrupt Message = 0x00002009
- KeyUp Interrupt Message = 0x00002109
- Address 0xFF000260 (Read/Write word): KEY_REG
- Address 0xFF000262 (Read/Write byte): KEY_STATUS
- Address 0xFF000263 (Write byte): KEY_CMD

### Jumper 1 = 3

- KeyDown Interrupt Message = 0x00003009
- KeyUp Interrupt Message = 0x00003109
- Address 0xFF000360 (Read/Write word): KEY_REG
- Address 0xFF000362 (Read/Write byte): KEY_STATUS
- Address 0xFF000363 (Read/Write byte): KEY_CMD

OPERATION
---------

Reading at KEY_REG, gets the last keyevent.
Reading at KEY_STATE, gets the state byte.
Reading value from KEY_CMD depends of the command.

Writing at KEY_REG, push a keyevent to the keyboard buffer
Writing at KEY_STATE, sets the state byte.
Writing at KEY_CMD, sends a command to the keyboard:

     VALUE |  NAME      | BEHAVIOUR
    -------+------------+----------------------------------------------------------
      0x00 | CLEAR      | Clear keyboard buffer
      0x01 | COUNT      | Reading KEY_CMD returns the number of keyevents stored in
           |            | the buffer when the command is send.
      0x02 | D-INT-DOWN | Disables interrupt when a Key Down hapens
      0x03 | E-INT-DOWN | Enables interrupts when a Key Down hapens
      0x04 | D-INT-UP   | Disables interrupts when a Key Up hapens
      0x05 | E-INT-UP   | Enables interrupts when a Key Up hapens
    -------+------------+----------------------------------------------------------

When interrupts are enabled, the keyboard will trigger an interrupt when one or
more keys have been pressed or released.

KeyEvent Format:

    15 14 13 12 11 10 9  8  7  6  5  4  3  2  1  0
    ----------------------------------------------
    | unused |  G  C  S  A  k  k  k  k  k  k  k  k

Where : 

 - G (Alt Gr mod) If the bit is set to 1, this means that the Alt Gr Key is being pressed at same time
 - C (Control mod) If the bit is set to 1, this means that the Control Key is being pressed at same time
 - S (Shift mod) If the bit is set to 1, this means that the Shift Key is being pressed at same time
 - A (Action Bit) If the bit is at 1, this means that key is being pressed 
     (Key Down). If is 0, means that the key was released (Key Up).
 - kkkkkkkk Key code (scan code) of the key pressed or released 

Key codes are:

- 0x05: Delete
- 0x06: Alt Graphics
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
- 0x27: Apostrophe (' ")
- 0x2C: Comma (, < )
- 0x2D: Minus (- _)
- 0x2E: Period (. >)
- 0x2F: Slash (/ ?)
- 0x30-0x39: Decimal digits (0-9)
- 0x3A: Semicolon (; :)
- 0x3B: Equal (= +)
- 0x41-0x5A: Latin characters (A to Z)
- 0x5B: Left Bracket ([ {)
- 0x5C: Backslash (\ |)
- 0x5D: Right Bracket (] {)
- 0x60: Grave Accent (` ~)
- 0xFF: Unknow key
- Other values: Reserved for advanced keyboards or localized keyboards

This represent the keys of a US keyboard layout.

If the return value is 0, then means that the buffer is empty.

State byte format:

    8  7  6  5  4  3  2  1  0
    -------------------------
    |      not used      |  C
 
Where:

 - C (CapsLock) Show if CapsLocks mode is active or not

### Status LEDs
A keyboard must have at least this LED state: Caps Locks Enable.
The Statues leds can bet set on/off writing to KEY_STATUS register.

### Key code events buffer
The buffer can store at least 64 keyevents. Each time that a key is pressed 
or released, the appropriate key code event is pushed to the buffer.
The buffer operates in FIFO mode, in addition if the buffer is filled not 
new keyevents will be store, requiring to PUSH a keyevent at least or cleaning the buffer.
Reading KEY_REG LSB, extracts the oldest keyevent in the buffer (POP buffer).
Writing KEY_REG, push a keyevent at the begin of the buffer (PUSH 
buffer), acting like a LIFO buffer. Writing at KEY_REG can be used to
simulate keyboard events by some programs or allow to the OS intercept keyevents.


    PUSH inserts here          
    ----------------->       
                       |---|
    -----------------> |   | Oldest key code event
    POP extract this   |---|
                       |   |
                       |---|
                       .   .
                       .   .
                       .   .
                       |---|
                       |   | Last key code event
                       |---|  
                             <----------------------
                               Keyboard inserts here

### Shift and Upper case letters and symbols
The keyboard generates alone the appropriate symbol or upper case letter when 
a key is pressed/released at same time that Shift key is being keep pressed. 

### Caps Locks Mode 
When Caps Locks key is pressed, the keyboard enters in Caps Locks mode. The 
next time that the Caps Lock key is pressed, then the keyboard leaves the 
mode. In Caps Locks mode, the Caps Locks Enable status LED is set to On, and 
when leaves this mode, is set to Off. Also writing to KEY_STATUS can activate 
or desactivate Caps Locks mode.


Example of Use
==============

A basic type program can use a ISR that reads at 0x60 (POP) to extract a
keyevent stored in the keyboard buffer. If the Action Bit is On in each key
code event, then writes in screen or stores in a string buffer if is an 
appropriate character, if not ignores it.
A more advanced ISR can store if Ctrl or Shift keys are being pressed, to 
detect Ctrl+Key or Shift+Key special actions.
  
    ; ISR
    ...
    ...
    BEQ %r0, 0x0009                     ; If is a Keyboard Interrupt, call to the
        CALL keyb_isr                   ; routine to handle it
    ...
    ...
    RFI

    keyb_isr:
    LOAD.B 0xFFF00060, %r0
    BCLEAR %r0, 0b1000_0000             ; Is a Key Up event ?
        BGE %r0, 0x20                   ; Is a writable character ?
            BLE %r0, 0x7E               ; 0x20 >= C <= 0x7E
                JMP copy_char
    RET

    copy_char:                          ; Code that writes on screen a character
    LOAD cursor, %r1                    ; Gets the value of the cursor in r1
    OR %r0, 0x0700                      ; White on black text
    SAVE.W %r1, %r0                     ; Puts the character + attribute on video
                                        ; ram
    ADD %r1, 2, %r1                     ; Increments the cursor var and save it
    SAVE cursor, %r1

    RET                                 

    cursor: .dd 0xFFFB0000              ; Cursor points to Video RAM address



Stuff to read : 
 - http://retired.beyondlogic.org/keyboard/keybrd.htm
 - flint.cs.yale.edu/cs422/doc/art-of-asm/pdf/CH20.PDF
 - http://bos.asmhackers.net/docs/keyboard/docs/KeyboardFAQ.txt

