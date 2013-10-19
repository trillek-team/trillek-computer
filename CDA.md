Color Display Adapter
=====================
Version 0.1 (WIP) 

Color Display Monitor that allows to display text modes and graphics modes in color,
using a 16 fixed color palette.

Allowed Text modes : 40x25 and 80x25
Allowed Graphics modes : 320x200, 640x200

Text modes uses a 8x8 pixel glyph cells, and the user can define a user Font.
Graphics modes, uses a 1 bit bit plane to set a pixel between foreground and
background color. The screen is divided in cells of 8x8 cells that have defines
a foreground and background color.

The video signals uses a vertical refresh rate of 50Hz and a horizontal refresh
rate of 21.8 KHz.

Device Class    : 0x0E (Graphics Device)
Device Build    : 0xXXXX
Device ID       : 0x0001 (CDA standard) 
Device Version  : 0x0000

Jumper 1 accepts values from 0 to 3

RESOURCES
---------

### Jumper 1 = 0

- Interrupt Message = 0x0000005A
- Address 0xFF0A0000 to 0xFF0A3E79: Frame Buffer/Text Area
- Address 0xFF0A3E80 to 0xFF0A4649: Frame Buffer Attribute Area
- Address 0xFF0A4650 to 0xFF0A4E49: Character RAM
- Address 0xFF0A4E60 (Read/Write byte): SETUP

### Jumper 1 = 1

- Interrupt Message = 0x0000105A
- Address 0xFF0B0000 to 0xFF0B3E79: Frame Buffer/Text Area
- Address 0xFF0B3E80 to 0xFF0B4649: Frame Buffer Attribute Area
- Address 0xFF0B4650 to 0xFF0B4E49: Character RAM
- Address 0xFF0B4E60 (Read/Write byte): SETUP

### Jumper 1 = 2

- Interrupt Message = 0x0000205A
- Address 0xFF0C0000 to 0xFF0C3E79: Frame Buffer/Text Area
- Address 0xFF0C3E80 to 0xFF0C4649: Frame Buffer Attribute Area
- Address 0xFF0C4650 to 0xFF0C4E49: Character RAM
- Address 0xFF0C4E60 (Read/Write byte): SETUP

### Jumper 1 = 3

- Interrupt Message = 0x0000305A
- Address 0xFF0D0000 to 0xFF0D3E79: Frame Buffer/Text Area
- Address 0xFF0D3E80 to 0xFF0D4649: Frame Buffer Attribute Area
- Address 0xFF0D4650 to 0xFF0D4E49: Character RAM
- Address 0xFF0D4E60 (Read/Write byte): SETUP

OPERATION
---------

Writing at SETUP register, sets the video mode and if is enabled V-Sync
Interrupt. SETUP register format :

- BIT 0-3 : Sets video mode
- BIT 4 : If is 0, sets the CDA card to Text Mode, if not, then sets CDA card to
  Graphics mode.
- BIT 5 : Unused
- BIT 6 : Enables the use of the RAM user font if is 1 and is using a Text Mode.
- BIT 7 : Enable V-Sync refresh interrupt. Does a interrupt every time that the
  screen is refreshed at a rate of 50Hz.

### Video modes
Bits 4 to 0 of SETUP register :

- 0 000 : Video Mode 0 40x25 Text mode
- 0 001 : Video Mode 1 80x25 Text mode
- 1 000 : Video Mode 4 320x200 Graphics mode
- 1 001 : Video Mode 5 640x200 Graphics mode

### Text mode

In text modes, the Text Area uses a word per screen glyph. In each word, is
defined the font glyph to use and the background/foreground color attributes.
The first word in the Text Area is the character at row 0, column 0. The next
word is the character at row 0, column 1, etc.

The format of each word is (Little Endian format):

    15 14 13 12 11 10  9  8  7  6  5  4  3  2  1  0
    -----------------------------------------------
     b  b  b  b  f  f  f  f  g  g  g  g  g  g  g  g

- gggggggg Chooses one of the 256 glyphs in the font
- ffff Chooses the foreground color from the palette
- gggg Chooses the background color from the palette

The color palette is :

- 0  Black
- 1  Blue
- 2  Green
- 3  Cyan
- 4  Red
- 5  Magenta
- 6  Brown
- 7  White
- 8  Gray
- 9  Light Blue
- 10 Light Green
- 11 Light Cyan
- 12 Light Red
- 13 Light Magenta
- 14 Yellow
- 15 High Intensity White

#### User defined font

Enabling bit 6 in SETUP register when is in a text mode, allows to use a user
defined font mapped in Character RAM region. The user font, consists in 256
glyphs of 8x8 pixels. A glyph is defined by contigous 8 bytes, being each byte,
a row of 8 pixels of the glyph, and the LSB bit defines the first pixel of the 
row. For example, to define the glyph of the
character 'F' :

    11111110 -> 0xFE
    00000010 -> 0x02
    00000010 -> 0x02
    00011110 -> 0x1E
    00000010 -> 0x02
    00000010 -> 0x02
    00000010 -> 0x02
    00000010 -> 0x02


### Graphics Mode

The CDA uses screen coordinates centered in the upper left corner of the
screen, and increases as we displace to the right or down.

    (0,0) -------------------- (639,0) or (319,0)
      |                                |
      |                                |
      |                                |
      |                                |
      |                                |
   (0,200) ----------------- (639,200) or (319,200)

In Graphics Mode, the CDA uses a 1 bit biplane to define if a pixel is set or
not. Using Little Endian scheme, the LSB bit of a byte defines the first pixel
of a row of 8 pixels, and the MSB bit defines the last pixel of the same row.
The first byte in the Frame Buffer defines the first row of 8 pixels in the
screen, pixels from (0,0) to (7,0).

In the Video Mode 4 the first line of the screen, (0,0) to (319,0), is defined
by the first 40 bytes of the Frame Buffer, the next line is defined by the bytes
next 40 bytes, etc.... Each line uses 40 bytes, so the full screen uses
320x200/8 = 8000 bytes

In the Video Mode 5 the first line of the screen, (0,0) to (639,0), is defined
by the first 80 bytes of the Frame Buffer, the next line is defined by the bytes
next 80 bytes, etc... Each line uses 80 bytes, so the full screen uses 
640x200/8 = 16000 bytes

A set pixel uses the foreground color, and an unset pixel uses the background
color.

The foreground and background colors are defined for 8x8 cells in the Frame 
Buffer Attribute Area. Each byte of this area defines the foreground and
background colors of a 8x8 cell on the screen. Uses the same format and color palette that the Text Mode attributes. The first Byte defines the cell (0,0),  the next byte, defines the cell (1,0), etc.


    (0,0) -------------------- (40,0) or (80,0)
      | | | | | | | | | | | | | | | | |
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
      | | | | | | | | | | | | | | | | | 
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
      | | | | | | | | | | | | | | | | | 
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
      | | | | | | | | | | | | | | | | | 
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
      | | | | | | | | | | | | | | | | | 
   (0,25) ------------------- (40,25) or (80,25)


