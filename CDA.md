Color Display Adapter
=====================
Version 0.2 (WIP) 

Color Display Monitor that allows to display text modes and graphics modes in color,
using a 16 fixed color palette.

- Allowed Text modes : 40x25 and 80x25
- Allowed Graphics modes : 320x200, 640x200

Text modes uses a 8x8 pixel glyph cells, and the user can define a user Font.
Graphics modes, uses a 1 bit bit plane to set a pixel between foreground and
background color. The screen is divided in cells of 8x8 cells that have defines
a foreground and background color.

The video signals uses a vertical refresh rate of 50Hz and a horizontal refresh
rate of 21.8 KHz.

- Device Class    : 0x0E (Graphics Device)
- Device Build    : 0xXXXX
- Device ID       : 0x0001 (CDA standard) 
- Device Version  : 0x0000

Jumper 1 accepts values from 0 to 3

RESOURCES
---------

A basic CDA card exposes 17KiB of Video RAM, the Used address depend of the Jumper 1 value. Also,e a interrupt on V-Sync can be used, and the message value depend of the Jumper 1 value.

### Jumper 1 = 0

- Interrupt Message = 0x0000005A
- Address 0xFF0A0000 to 0xFF0A43FF: Video RAM
- Address 0xFF0ACC00 (Read/Write byte): SETUP

### Jumper 1 = 1

- Interrupt Message = 0x0000105A
- Address 0xFF0B0000 to 0xFF0B43FF: Video RAM
- Address 0xFF0BCC00 (Read/Write byte): SETUP

### Jumper 1 = 2

- Interrupt Message = 0x0000205A
- Address 0xFF0C0000 to 0xFF0C43FF: Video RAM
- Address 0xFF0CCC00 (Read/Write byte): SETUP

### Jumper 1 = 3

- Interrupt Message = 0x0000305A
- Address 0xFF0C0000 to 0xFF0C43FF: Video RAM
- Address 0xFF0CCC00 (Read/Write byte): SETUP

OPERATION
---------

Writing at SETUP register, sets the video mode and if is enabled V-Sync
Interrupt. SETUP register format :

- BIT 0-2 : Sets video mode
- BIT 3 : If is 0, sets the CDA card to Text Mode, if not, then sets CDA card to
  Graphics mode.
- BIT 4 : Unused
- BIT 5 : If the CDA is in text mode, the bright background color bit is used a blink attribute instead of bright.
- BIT 6 : If the CDA is in text mode, enables the use of the RAM user font.
- BIT 7 : Enable V-Sync refresh interrupt. Does a interrupt every time that the
  screen is refreshed at a rate of 50Hz.

### Video modes
Bits 3 to 0 of SETUP register :

- 0 00 : Video Mode 0 40x25 Text mode
- 0 01 : Video Mode 1 80x25 Text mode
- 1 00 : Video Mode 4 320x200 Graphics mode
- 1 01 : Video Mode 5 640x200 Graphics mode

### Text mode

In text modes, the Video RAM represents a glyph and the color attributes using a word to store both. In each word, is defined the  glyph index to use and the background/foreground color attributes.
The first word in the Text Area is the character at row 0, column 0. The next
word is the character at row 0, column 1, etc.

Formula : address = video ram address + (column % MAX_COLUMNS)*2 + (MAX_COLUMNS*2 * row)
Were MAX_COLUMNS = 40 or 80 depending of what text mode is.

The format of each word is (Little Endian format):

    15 14 13 12 11 10  9  8  7  6  5  4  3  2  1  0
    -----------------------------------------------
     B  b  b  b  F  f  f  f  g  g  g  g  g  g  g  g

- gggggggg Chooses one of the 256 glyphs in the font
- fff Chooses the foreground color from the palette
- F Sets if the foreground color is bright or not, could be Blink attribute is bit 5 of SETUP is on.
- bbb Chooses the background color from the palette
- B Sets if the background color is bright or not.

The color palette is :

- 0  <span style="background-color:#000000;">&nbsp;&nbsp;a</span> Black
- 1  <span style="background-color:#0000CD;">&nbsp;&nbsp;</span> Dark Blue
- 2  <span style="background-color:#00CD00;">&nbsp;&nbsp;</span> Dark Green
- 3  <span style="background-color:#00CDCD;">&nbsp;&nbsp;</span> Dark Cyan
- 4  <span style="background-color:#CD0000;">&nbsp;&nbsp;</span> Dark Red
- 5  <span style="background-color:#CD00CD;">&nbsp;&nbsp;</span> Dark Magenta
- 6  <span style="background-color:#AA5500;">&nbsp;&nbsp;</span> Brown
- 7  <span style="background-color:#CDCDCD;">&nbsp;&nbsp;</span> Light Gray

With the bright bite enable, the colors are :

- 0  <span style="background-color:#555555;">&nbsp;&nbsp;</span> Dark Gray 
- 1  <span style="background-color:#0000FF;">&nbsp;&nbsp;</span> Blue
- 2  <span style="background-color:#00FF00;">&nbsp;&nbsp;</span> Green
- 3  <span style="background-color:#00FFFF;">&nbsp;&nbsp;</span> Cyan
- 4  <span style="background-color:#FF0000;">&nbsp;&nbsp;</span> Red
- 5  <span style="background-color:#FF00FF;">&nbsp;&nbsp;</span> Magenta
- 6  <span style="background-color:#FFFF00;">&nbsp;&nbsp;</span> Yellow
- 7  <span style="background-color:#FFFFFF">&nbsp;&nbsp;</span> White


#### User defined font

Enabling bit 6 in SETUP register when is in a text mode, allows to use a user
defined font mapped in the last 2KiB of the Video RAM. The user font, consists in 256
glyphs of 8x8 pixels. A glyph is defined by contiguous 8 bytes, being each byte,
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

Formula : glyph address = (video ram address + video ram size - 2048) + glyph index * 8

### Graphics Mode

The CDA uses screen coordinates centered in the upper left corner of the
screen, and increases as we displace to the right or down.

     (0,0) -------------------- (639,0) or (319,0)
       |                                |
       |                                |
       |                                |
       |                                |
       |                                |
    (0,199) ----------------- (639,199) or (319,199)

In Graphics Mode, the CDA uses a 1 bit biplane to define if a pixel is active or
not. An active pixel uses the foreground color, and inactive pixel uses the background color. 
Using Little Endian scheme, the LSB bit of a byte defines the first pixel
of a row of 8 pixels, and the MSB bit defines the last pixel of the same row.
The first byte in the Frame Buffer defines the first row of 8 pixels in the
screen, pixels from (0,0) to (7,0).

In the Video Mode 4 the first line of the screen, (0,0) to (319,0), is defined
by the first 40 bytes of the Frame Buffer, the next line is defined by the bytes
next 40 bytes, etc.... Each line uses 40 bytes, so the full screen uses
320x200/8 = 8000 bytes

Formula : pixel address (X,Y) = video ram address + ((X % 320)/8) + (320 * Y / 8)
          pixel bit (X) = X%8


In the Video Mode 5 the first line of the screen, (0,0) to (639,0), is defined
by the first 80 bytes of the Frame Buffer, the next line is defined by the bytes
next 80 bytes, etc... Each line uses 80 bytes, so the full screen uses 
640x200/8 = 16000 bytes

Formula : pixel address (X,Y) = video ram address + ((X % 640)/8) + (640 * Y / 8)
          pixel bit (X) = X%8

The foreground and background colors are defined for 8x8 cells just before the bitplane. Each byte of this area defines the foreground and background colors of a 8x8 cell on the screen. Uses the same format and color palette that the Text Mode attributes. The first Byte defines the cell (0,0), the next byte, defines the cell (1,0), etc.


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

In 320x200 :

Formula : pixel attribute address (X,Y) = video ram address + 8000 + ((X % 320)/8) + (320 * Y / 8)/8

In 640x200 :

Formula : pixel attribute address (X,Y) = video ram address + 16000 + ((X % 640)/8) + (640 * Y / 8)/8