Generic Keyboard
================
Version 0.1 (WIP) 

Generic Compatible Keyboard of RC3200 systems. Handles an internal buffer to
store key events.

Device Class    : 0x03 (HID)
Device Build    : 0xXXXX
Device ID       : 0x0001 (Generic Keyboard)
Device Version  : 0xXXXX

Jumper 1 acepts values from 0 to 3

RESOURCES
---------

### Jumper 1 = 0

- Interrupt Message = 0x00000009
- Address 0xFF000060 (Read/Write byte): KEY_REG
- Address 0xFF000061 (Read/Write byte): KEY_STATUS
- Address 0xFF000062 (Write byte): KEY_CMD

### Jumper 1 = 1

- Interrupt Message = 0x00000109
- Address 0xFF000160 (Read/Write byte): KEY_REG
- Address 0xFF000161 (Read/Write byte): KEY_STATUS
- Address 0xFF000162 (Write byte): KEY_CMD

### Jumper 1 = 2

- Interrupt Message = 0x00000209
- Address 0xFF000260 (Read/Write byte): KEY_REG
- Address 0xFF000261 (Read/Write byte): KEY_STATUS
- Address 0xFF000262 (Write byte): KEY_CMD

### Jumper 1 = 3

- Interrupt Message = 0x00000309
- Address 0xFF000360 (Read/Write byte): KEY_REG
- Address 0xFF000361 (Read/Write byte): KEY_STATUS
- Address 0xFF000362 (Write byte): KEY_CMD

OPERATION
---------

Reading at KEY_REG, gets the last keyevent.
Reading at KEY_STATUS, gets the status byte.

Writing at KEY_REG, push a keyevent to the keyboard buffer
Writing at KEY_STATUS, sets the status byte.
Writing at KEY_CMD, sends a command to the keyboard:

     VALUE |  NAME      | BEHAVIOUR
    -------+------------+----------------------------------------------------------
      0x00 | CLEAR      | Clear keyboard buffer
      0x01 | D-INTERRUPT| Disables interrupts
      0x02 | E-INTERRUPT| Enables interrupts
    -------+------------+----------------------------------------------------------

When interrupts are enabled, the keyboard will trigger an interrupt when one or
more keys have been pressed or released.

Keyevent Format:

    8  7  6  5  4  3  2  1  0
    -------------------------
    A  k  k  k  k  k  k  k  k

Where :

 - A (Action Bit) If the bit is at 1, this means that key is being pressed 
     (Key Down). If is 0, means that the key was released (Key Up).
 - kkkkkkkk Key code of the key pressed or released 

Key codes are:

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
- 0x1B: Escape/Break
- 0x20-0x7E: ASCII characters
- 0x7F: Delete
- Other values: Reserved for advanced keyboards or localized keyboards

If the return value is 0, then means that the buffer is empty.

Status byte format:

    8  7  6  5  4  3  2  1  0
    -------------------------
    |      not used      |  C
 
Where:

 - C (CapsLock) Show if CapsLocks mode is active or not

### Status LEDs
A keyboard must have at least this LED state: Caps Locks Enable.

### Key code events buffer
The buffer can store at least 64 keyevents. Each time that a key is pressed 
or released, the appropriate key code event is pushed to the buffer.
The buffer operates in FIFO mode, in addition if the buffer is filled, then
the oldest key code stored is deleted.
Reading at port 0x60, extracts the oldest keyevent in the buffer(POP buffer).
Writing at Port 0x60, push a keyevent at the begin of the buffer (PUSH 
buffer), acting like a LIFO buffer. Writing at port 0x60 can be used to
simulate keyboard events by some programs or allow to intercept keyevents to
the OS.


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
when leaves this mode, is set to Off. 


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

