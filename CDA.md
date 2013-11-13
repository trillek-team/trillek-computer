Color Display Adapter
=====================
Version 0.3 (WIP) 

Color Display Adapter (CDA) device that allows to display text modes and graphics modes in color, using a 16 programmable color palette.

- Allowed Text modes : 40x30
- Allowed Graphics modes : 256x192 and 320x200

Text modes uses a 8x8 pixel glyph cells, and the user can define a user Font.
Graphics modes, uses a bit plane to set a pixel between foreground and
background color. The screen is divided in attribute cells that defines a foreground and background color.

The video signals uses a vertical refresh rate of 50Hz and a horizontal refresh
rate of 21.8 KHz.

- Device Class    : 0x0E (Graphics Device)
- Device Build    : 0xXXXX
- Device ID       : 0x0001 (CDA standard) 
- Device Version  : 0x0000

Jumper 1 accepts values from 0 to 3

RESOURCES
---------

A basic CDA card exposes 9600 Bytes of Video RAM. The address were is mapped depend of the Jumper 1 value. Also, a interrupt on V-Sync can be used, and the message value depend of the Jumper 1 value.

### Jumper 1 = 0

- Interrupt Message = 0x0000005A
- Address 0xFF0A0000 to 0xFF0A2580: Video RAM
- Address 0xFF0ACC00 (Read/Write byte): SETUP

### Jumper 1 = 1

- Interrupt Message = 0x0000105A
- Address 0xFF0B0000 to 0xFF0B2580: Video RAM
- Address 0xFF0BCC00 (Read/Write byte): SETUP

### Jumper 1 = 2

- Interrupt Message = 0x0000205A
- Address 0xFF0C0000 to 0xFF0C2580: Video RAM
- Address 0xFF0CCC00 (Read/Write byte): SETUP

### Jumper 1 = 3

- Interrupt Message = 0x0000305A
- Address 0xFF0C0000 to 0xFF0C2580: Video RAM
- Address 0xFF0CCC00 (Read/Write byte): SETUP

OPERATION
---------

Writing at SETUP register, sets the video mode and if is enabled V-Sync
Interrupt. SETUP register format :

- BIT 0-2 : Sets video mode
- BIT 3 : If is 0, sets the CDA card to Text Mode, if not, then sets CDA card to
  Graphics mode.
- BIT 4 : Unused
- BIT 5 : If the CDA is in text mode, enables the use of the RAM user font.
- BIT 6 : Enables the use of the RAM user color palette.
- BIT 7 : Enables V-Sync refresh interrupt. Does a interrupt every time that the
  screen is refreshed at a rate of 25Hz.

### Video modes
Bits 3 to 0 of SETUP register :

- 0 000 : Video Mode 0  40x30 Text mode without border
- 1 000 : Video Mode 8  256x192 Gaphics mode, 8x8 Attribute cell, with border
- 1 001 : Video Mode 9  256x192 Gaphics mode, 4x4 Attribute cell, with border
- 1 010 : Video Mode 10 320x240 Graphics mode, B&W, without border.
- Rest are reserved for future compatible video adapters.

### Text mode

In text modes, the Video RAM represents a glyph and the color attributes using a word to store both. In each word, is defined the  glyph index to use and the background/foreground color attributes.
The first word in the Text Area is the character at row 0, column 0. The next
word is the character at row 0, column 1, etc.

Formula : address = video ram address + (column % MAX_COLUMNS)*2 + (MAX_COLUMNS*2 * row)
Were MAX_COLUMNS = 40 or other value depending of what text mode is.

The format of each word is (Little Endian format):

    15 14 13 12 11 10  9  8  7  6  5  4  3  2  1  0
    -----------------------------------------------
     b  b  b  b  f  f  f  f  g  g  g  g  g  g  g  g

- gggggggg Chooses one of the 256 glyphs in the font
- ffff Chooses the foreground color from the palette
- bbbb Chooses the background color from the palette


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

     (0,0) ------------------------- (255,0)
       |                                or
       |                             (319,0)
       |                                |
       |                                |
       |                                |
       |                                |
       |                                |
       |                                |
       |                                |
       |                                |
    (0,199) ----------------------- (255,199)
       or                               or
    (0,239)                         (319,239)

In Graphics Mode, the CDA uses a 1 bit biplane to define if a pixel is active or
not. An active pixel uses the foreground color, and inactive pixel uses the background color. 
Using Little Endian scheme, the LSB bit of a byte defines the first pixel
of a row of 8 pixels, and the MSB bit defines the last pixel of the same row.
The first byte in the Frame Buffer defines the first row of 8 pixels in the
screen, pixels from (0,0) to (7,0).

#### Mode 8 and 9

In the **Video Mode 8 and 9** the first line of the screen, (0,0) to (255,0), is defined
by the first 32 bytes of the Frame Buffer, the next line is defined by the next 32 bytes, etc.... Each line uses 32 bytes, so the full screen uses
256x192/8 = 6144 bytes

Formula : pixel address (X,Y) = video ram address + ((X % 256)/8) + (256 * Y / 8)
          pixel bit (X) = X%8

The foreground and background colors are defined for 8x8 cells in mode 8, 4x4 cells in mode 9 just before the bitplane. Each byte of this area defines the foreground and background colors of each cell on the screen. Uses the same color format that the Text Mode attributes. The first Byte defines the cell (0,0), the next byte, defines the cell (1,0), etc.

With 8x8 cells :

     (0,0) ------------------------- (31,0)
       | | | | | | | | | | | | | | | | |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
       | | | | | | | | | | | | | | | | | 
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
       | | | | | | | | | | | | | | | | | 
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
       | | | | | | | | | | | | | | | | | 
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
       | | | | | | | | | | | | | | | | | 
    (0,23) ------------------------ (31,23)

With 4x4 cells :

     (0,0) ------------------------- (63,0)
       | | | | | | | | | | | | | | | | |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
       | | | | | | | | | | | | | | | | | 
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
       | | | | | | | | | | | | | | | | | 
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
       | | | | | | | | | | | | | | | | | 
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
       | | | | | | | | | | | | | | | | | 
    (0,47) ------------------------ (63,47)

In Mode 8 :

pixel attribute address (X,Y) = video ram address + 6144 + ((X % 256)/8) + (256 * Y / 8)/8

In Mode 9 :

pixel attribute address (X,Y) = video ram address + 6144 + ((X % 256)/4) + (256 * Y / 4)/4

In addition, mode 8 and mode 9 have a border around the active screen. This border fills the screen to a 320x240 rectangle, giving borders of 32x24 pixels. The color of the border are defined by the four LSB bits of the byte just
before of the attribute ram.

In Mode 8 :

border color address = video ram address + 6912 

In Mode 9 :

border color address = video ram address + 9216 

#### Mode 10

In the **Video Mode 10** the first line of the screen, (0,0) to (319,0), is defined
by the first 40 bytes of the Frame Buffer, the next line is defined by the next 40 bytes, etc... Each line uses 40 bytes, so the full screen uses 
320x240/8 = 9600 bytes

Formula : pixel address (X,Y) = video ram address + ((X % 320)/8) + (320 * Y / 8)
          pixel bit (X) = X%8


### Color palette

If the bit 6 of SETUP is enable, then the CDA allow to define a color palette. The palette is defined before Frame
Buffer / Text buffer ends, and uses 48 bytes.
The palette is an array were each element is defined by 3 bytes, were the first byte is the RED component, the second
byte is the GREEN component and the third byte is the BLUE component. In other words, this is a 24 bit value that
represents a color in RGB8 format.

If the bit 6 is not enabled, then the default palette is used. The default color palette is :

- 0  <span style="background-color:#000000;">&nbsp;&nbsp;</span> 0x000000 Black
- 1  <span style="background-color:#0000CD;">&nbsp;&nbsp;</span> 0xCD0000 Dark Blue
- 2  <span style="background-color:#00CD00;">&nbsp;&nbsp;</span> 0x00CD00 Dark Green
- 3  <span style="background-color:#00CDCD;">&nbsp;&nbsp;</span> 0xCDCD00 Dark Cyan
- 4  <span style="background-color:#CD0000;">&nbsp;&nbsp;</span> 0x0000CD Dark Red
- 5  <span style="background-color:#CD00CD;">&nbsp;&nbsp;</span> 0xCD00CD Dark Magenta
- 6  <span style="background-color:#AA5500;">&nbsp;&nbsp;</span> 0x0055AA Brown
- 7  <span style="background-color:#CDCDCD;">&nbsp;&nbsp;</span> 0xCDCDCD Light Gray
- 8  <span style="background-color:#555555;">&nbsp;&nbsp;</span> 0x555555 Dark Gray 
- 9  <span style="background-color:#0000FF;">&nbsp;&nbsp;</span> 0xFF0000 Blue
- 10 <span style="background-color:#00FF00;">&nbsp;&nbsp;</span> 0x00FF00 Green
- 11 <span style="background-color:#00FFFF;">&nbsp;&nbsp;</span> 0xFFFF00 Cyan
- 12 <span style="background-color:#FF0000;">&nbsp;&nbsp;</span> 0x0000FF Red
- 13 <span style="background-color:#FF00FF;">&nbsp;&nbsp;</span> 0xFF00FF Magenta
- 14 <span style="background-color:#FFFF00;">&nbsp;&nbsp;</span> 0x00FFFF Yellow
- 15 <span style="background-color:#FFFFFF;">&nbsp;&nbsp;</span> 0xFFFFFF White

## Memory Maps

### Mode 0 : Text mode of 40x30

    |-----------------------------------------------------------------------------|
    |                                  |          |                               |
    |            Text buffer           | Palette  |          User Font            |
    |                                  |          |                               |
    |----------------------------------|----------|-------------------------------|
    Base Address                   +0x4B0       +0x4DB                       +0xCDB

### Mode 8 : Graphics mode of 256x192 with 8x8 attribute cells

    |-----------------------------------------------------------------------------|
    |                               |      Attribute      |   Border   |          |
    |        Frame buffer           |                     |            | Palette  |
    |                               |         RAM         |   Color    |          |
    |-------------------------------|---------------------|------------|----------|
    Base Address                +0x1800               +0x1B00      +0x1B01  +0x1B31

### Mode 9 : Graphics mode of 256x192 with 4x4 attribute cells

    |-----------------------------------------------------------------------------|
    |                               |      Attribute      |   Border   |          |
    |        Frame buffer           |                     |            | Palette  |
    |                               |         RAM         |   Color    |          |
    |-------------------------------|---------------------|------------|----------|
    Base Address                +0x1800               +0x2400      +0x2401  +0x2431

### Mode 10 : B&W Graphics mode of 320x240 

    |-------------------------------|
    |                               |
    |        Frame buffer           |
    |                               |
    |-------------------------------|
    Base Address                +0x2580


