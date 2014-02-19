Color Display Adapter
=====================
Version 0.4 (WIP) 

Color Display Adapter (CDA) device that allows to display text modes and graphics modes in color, using a 16 programmable color palette.

- Allowed Text modes : 40x30
- Allowed Graphics modes : 256x192 and 320x240

Text modes uses a 8x8 pixel glyph cells, and the user can define a user Font.
Graphics modes, uses a bit plane to set a pixel between foreground and
background color. The screen is divided in attribute cells that defines a foreground and background color.

The refresh rate should be around 25 hz.

- Device Class    : 0x0E (Graphics Device)
- Device Builder  : 0xXXXX
- Device ID       : 0x0001 (CDA standard) 


RESOURCES
---------

A basic CDA card exposes 9600 Bytes of Video RAM, and can generate a interrupt for Vsync events.

COMMANDS
--------

    |  Value  |   Name   | Description
    +---------+----------+-----------------------------------------------------
    | 0x0000  | SET_ADDR | Sets base address to map to A:B value. If the 
    |         |          | address is valid, and the device can map these
    |         |          | address, will set C register to 0x0000. If not, will
    |         |          | set A register to 0xFFFF. Setting A:B to 0, disables
    |         |          | the address map, and disables the graphics output.
    | 0x0001  | GET_ADDR | Return base address maped. Sets A:B register to the 
    |         |          | base address being used by the device. When the 
    |         |          | computer boots, this base address is 0.
    | 0x0002  | GET_ASIZE| Returns the address block size. Sets A register to 
    |         |          | the address block size being used by the device. In 
    |         |          | this case is 9600
    | 0x0003  | SET_INT  | Sets interrupt message to A register value. If A is 
    |         |          | 0x0000, then disables VSync interrupt. This is set
    |         |          | to 0 when the computer boots.
    | 0x0004  | GET_INT  | Gets interrupt message. Sets A register to the value
    |         |          | of the interrupt message.
    | 0x0005  | SET_MODE | Sets the video mode. A register value selects one of
    |         |          | the valid video modes. B register bit 0 could enable
    |         |          | custom RAM color palette and bit 1 could enable
    |         |          | custom RAM font in text modes.
    | 0x0006  | GET_MODE | Gets the actual video mode. Sets A and B registers
    |         |          | to the actual video mode and configuration bits.
    +---------+----------+-----------------------------------------------------


### Valid Video modes

- 0x0000 : Text  Mode 0 40x30 Text mode without border
- 0x1000 : Video Mode 0 256x192 Gaphics mode, 8x8 Attribute cell, with border
- 0x1001 : Video Mode 1 256x192 Gaphics mode, 4x4 Attribute cell, with border
- 0x1010 : Video Mode 2 320x240 Graphics mode, B&W, without border.
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

#### Mode 0 and 1

In the **Video Mode 8 and 9** the first line of the screen, (0,0) to (255,0), is defined
by the first 32 bytes of the Frame Buffer, the next line is defined by the next 32 bytes, etc.... Each line uses 32 bytes, so the full screen uses
256x192/8 = 6144 bytes

Formula : pixel address (X,Y) = video ram address + ((X % 256)/8) + (256 * Y / 8)
          pixel bit (X) = X%8

The foreground and background colors are defined for 8x8 cells in mode 0, 4x4 cells in mode 1 just before the bitplane. Each byte of this area defines the foreground and background colors of each cell on the screen. Uses the same color format that the Text Mode attributes. The first Byte defines the cell (0,0), the next byte, defines the cell (1,0), etc.

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

Formula for Mode 0 :

pixel attribute address (X,Y) = video ram address + 6144 + ((X % 256)/8) + (256 * Y / 8)/8

Formula for Mode 1 :

pixel attribute address (X,Y) = video ram address + 6144 + ((X % 256)/4) + (256 * Y / 4)/4

In addition, mode 0 and mode 1 have a border around the active screen. This border fills the screen to a 320x240 rectangle, giving borders of 32x24 pixels. The color of the border are defined by the four LSB bits of the byte just
before of the attribute ram.

Formula for Mode 0 :

border color address = video ram address + 6912 

Formula for Mode 1 :

border color address = video ram address + 9216 

#### Mode 2

In the **Video Mode 2** the first line of the screen, (0,0) to (319,0), is defined
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

### Text Mode 0 : 40x30

    |-----------------------------------------------------------------------------|
    |                                  |          |                               |
    |            Text buffer           | Palette  |          User Font            |
    |                                  |          |                               |
    |----------------------------------|----------|-------------------------------|
    Base Address                   +0x960       +0x990                      +0x1190

### Video Mode 0 : 256x192 with 8x8 attribute cells

    |-----------------------------------------------------------------------------|
    |                               |      Attribute      |   Border   |          |
    |        Frame buffer           |                     |            | Palette  |
    |                               |         RAM         |   Color    |          |
    |-------------------------------|---------------------|------------|----------|
    Base Address                +0x1800               +0x1B00      +0x1B01  +0x1B31

### Video Mode 1 : 256x192 with 4x4 attribute cells

    |-----------------------------------------------------------------------------|
    |                               |      Attribute      |   Border   |          |
    |        Frame buffer           |                     |            | Palette  |
    |                               |         RAM         |   Color    |          |
    |-------------------------------|---------------------|------------|----------|
    Base Address                +0x1800               +0x2400      +0x2401  +0x2431

### Video Mode 2 : 320x240 B&W 

    |-------------------------------|
    |                               |
    |        Frame buffer           |
    |                               |
    |-------------------------------|
    Base Address                +0x2580


