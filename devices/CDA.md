---
layout : default
title : Color Display Adapter (CDA)
cat : NotYet
---
Investronics Color Display Adapter
==================================
Version 0.5.1 (WIP, perhaps would be droped or totally rewrite) 

The Color Display Adapter (CDA) device allows to display text and graphics in 
color, using a 16 programmable color palette. It's compatible with TGA device.

 - Allowed Text modes : 40x30 8x8 pixel font cell
 - Allowed Graphics modes : 256x192 and 320x240

The refresh rate should be around 25 hz.

 - Device Class    : 0x0E (Graphics Device)
 - Device SubType  : 0x01 (TGA compatible)
 - Device Builder  : 0x494E5645 (Investronics)
 - Device ID       : 0x01 


RESOURCES
---------

A basic CDA card exposes 9600 Bytes of Video RAM for graphics modes, and can 
generate a interrupt for Vsync events.
Like the TGA, can use computer RAM for a Text buffer and defined user font.

COMMANDS
--------

 - 0x0000 : **MAP_BUFFER** :  
   Sets text buffer address to B:A value. If is a valid RAM address, will set 
   C register to 0x0000. If not, will set C register to 0xFFFF and will ignore 
   the value. Setting B:A to 0, disables **text** output, if the card is in 
   text mode.
 - 0x0001 : **MAP_FONT** :  
   Sets user font address to B:A value. If is a valid RAM address, will set C
   register to 0x0000. If not, will set C register to 0xFFFF. Setting B:A to 0,
   disables the user font, and restores the default font.
 - 0x0002 : **SET_INT** :  
   Sets interrupt message to A register value. If A is 0x0000, disables VSync 
   interrupt. At boot/reset time the VSync is disabled.
 - 0x0003 : **SET_MODE** :  
   Sets the video mode. A register value selects one of the valid video modes. 
 - 0x0004 : **SET_VRAM** :  
   Sets Video RAM base address to B:A value. If is a not used address, will set
   C register to 0x0000. If not, will set C register to 0xFFFF and will ignore 
   the value. Setting B:A to 0, disables **graphics** output, if the card is in
   grpahics mode.
 - 0x0005 : **SET_PALETTE** :  
   Sets Palette RAM base address to B:A value. If is a not used address, will 
   set C register to 0x0000. If not, will set C register to 0xFFFF and will 
   ignore the value. Setting B:A to 0, disables custom palette and restores 
   default palette.
 - 0x0006: **SET_BORDER_COLOR** :
   Sets the border color to LSB byte of A register value. Border will only 
   displayed in video modes that support it.


### Valid Video modes

- 0x0000 : Text  Mode 0 40x30 Text mode without border
- 0x1000 : Video Mode 0 256x192 Gaphics mode, 8x8 Attribute cell, with border
- 0x1001 : Video Mode 1 256x192 Gaphics mode, 4x4 Attribute cell, with border
- 0x1010 : Video Mode 2 320x240 Graphics mode, B&W, without border.
- Rest are reserved for future compatible video adapters.

### Text mode
#### Text buffer

The Text buffer represents a glyph and the color attributes using a word to 
store both. In each word, is defined the glyph index to use and the 
background/foreground color attributes.
The first word in the Text Area is the character at row 0, column 0. The next
word is the character at row 0, column 1, etc.

Formula :  
    character address = 
    text buffer address + (column % MAX_COLUMNS)*2 + (MAX_COLUMNS*2 * row)

Were MAX_COLUMNS = 40 or other value depending of what text mode is.

The format of each word is (Little Endian format):

    15 14 13 12 11 10  9  8  7  6  5  4  3  2  1  0
    -----------------------------------------------
     b  b  b  b  f  f  f  f  g  g  g  g  g  g  g  g

- gggggggg Chooses one of the 256 glyphs in the font
- ffff Chooses the foreground color from the palette
- bbbb Chooses the background color from the palette


#### User defined font

Sending command MAP_FONT with B:A pointing to a RAM address, enables the user 
defined font in RAM. It uses the 2KiB of RAM to define a custom 8x8 font.
A glyph of the font, is defined by contiguous 8 bytes, being each byte, a row 
of 8 pixels of the glyph, and the MSB bit defines the first pixel of the row. 
For example, to define the glyph of the character 'F' :

    BYTE |76543210 <- BIT
    -----+--------
      0  |01111111 = 0x7F
      1  |01000000 = 0x40
      2  |01000000 = 0x40
      3  |01111000 = 0x78
      4  |01000000 = 0x40
      5  |01000000 = 0x40
      6  |01000000 = 0x40
      7  |01000000 = 0x40

Formula :  
    glyph address = user font address + glyph index * 8

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


#### Mode 0 and 1
In this video modes, the CDA divides the Video RAM in two sections, the 
**Frame buffer** and the **Attribute buffer**.

In the Frame buffer, CDA uses a 1 bit biplane to define if a pixel is active or
not. An active pixel uses the foreground color, and inactive pixel uses the 
background color. Using Little Endian scheme, the LSB bit of a byte defines the
first pixel of a row of 8 pixels, and the MSB bit defines the last pixel of the
same row. The first byte in the Frame Buffer defines the first row of 8 pixels 
in the screen, pixels from (0,0) to (7,0).

The first line of the screen, (0,0) to (255,0), is defined by the first 32 
bytes of the Frame Buffer, the next line is defined by the next 32 bytes, etc...
Each line uses 32 bytes, so the frame buffer have a size of 256x192/8 = 6144 bytes

Formula :  
    pixel address (X,Y) =  
    video ram address + ((X % 256)/8) + (256 * Y / 8)

    pixel bit (X) = X%8

In the Attribute buffer, the foreground and background colors are defined for 
8x8 pixels cells in mode 0, and 4x4 pixels in mode 1. The first Byte defines 
the cell (0,0), the next byte, defines the cell (1,0), etc. The Attribute buffer is 
just before the Frame buffer. The format is :
    
     7  6  5  4  3  2  1  0
    -----------------------
     b  b  b  b  f  f  f  f

 - ffff Chooses the foreground color from the palette
 - bbbb Chooses the background color from the palette

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
    pixel attribute address (X,Y) = 
    video ram address + 6144 + ((X % 256)/8) + (256 * Y / 8)/8

Formula for Mode 1 :  
    pixel attribute address (X,Y) = 
    video ram address + 6144 + ((X % 256)/4) + (256 * Y / 4)/4

Also, 

In addition, mode 0 and mode 1 have a border around the active screen. This border fills the screen to a 320x240 rectangle, giving borders of 32x24 pixels. The color of the border are defined by the four LSB bits of the byte just
before of the attribute ram.

Formula for Mode 0 :

border color address = video ram address + 6912 

Formula for Mode 1 :

border color address = video ram address + 9216 

#### Mode 2

In this video mode, the CDA works ina Black&White mode, and uses the Video RAM
as 1 bit biplane to define if a pixel is active or not. An active pixel uses 
White color, and inactive pixel uses the Black color of the actual palette.

The first line of the screen, (0,0) to (319,0), is defined by the first 40 
bytes of the Video RAM, the next line is defined by the next 40 bytes, etc... 
Each line uses 40 bytes, so the full screen uses 320x240/8 = 9600 bytes

Formula :  
    pixel address (X,Y) =  
    video ram address + ((X % 320)/8) + (320 * Y / 8)
    
    pixel bit (X) = X%8


### Color palette

If SET_PALETTE was called with a valid, not assigned, address, the CDA will 
expose a 64 byte RAM were is defined the actual palette being used. When this 
command is called first time, or before doing a SET_PALETTE to 0, the default 
palette will be stored in this RAM.
The palette is an array were each element is defined by a 3 color bytes + a 
padding byte, were the first byte the RED component, the second byte is the 
GREEN component, the third byte is the BLUE component and the fourth byte is 
the padding byte. In other words, is stores in little endian RGBX8 format. 
In hexadecimal is : 0x00BBGGRR

The color palette in Little-endian RGB8 format (Arne 16 color palette) : 

 - 0   0x00000000 Black
 - 1   0x00EFDCB2 Light Blue
 - 2   0x00F2A231 Mid Blue
 - 3   0x00FF0000 Blue
 - 4   0x0032261B Dark Blue
 - 5   0x0027CEA3 Light Green
 - 6   0x001A8944 Green
 - 7   0x004E482F Swamp Green
 - 8   0x006BE2F7 Yellow 
 - 9   0x003189EB Copper
 - 10  0x002264A4 Brown
 - 11  0x002B3C49 Dark Brown
 - 12  0x008B6FE0 Pink
 - 13  0x003326BE Red
 - 14  0x009D9D9D Gray
 - 15  0x00FFFFFF White

![Palette](img/dia/palette.png "Palette")

## Video Memory Maps

### Video Mode 0 : 256x192 with 8x8 attribute cells

    |------------------------------------------------------
    |                               |      Attribute      |
    |        Frame buffer           |                     |
    |                               |         RAM         |
    |-------------------------------|---------------------|
    Base Address                +0x1800               +0x1B00

### Video Mode 1 : 256x192 with 4x4 attribute cells

    |------------------------------------------------------------------|
    |                               |      Attribute                   |
    |        Frame buffer           |                                  |
    |                               |         RAM                      |
    |-------------------------------|----------------------------------|
    Base Address                +0x1800                            +0x2400

### Video Mode 2 : 320x240 B&W 

    |-------------------------------|
    |                               |
    |        Frame buffer           |
    |                               |
    |-------------------------------|
    Base Address                +0x2580


