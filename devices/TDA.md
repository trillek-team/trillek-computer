---
---
Nya Elektriska Text Display Adapter
=====================================
Version 0.1b (WIP) 

The Text Generator Adapter (TDA) device  usesa programable character generator 
that allows to display text with color and could use user defined font.

 - Allowed Text modes : 40x30 8x8 pixel font cell

The refresh rate should be around 25 hz.

 - Device Type     : 0x0E (Graphics Device)
 - Device SubType  : 0x01 
 - Device Builder  : 0x1C6C8B36 (Nya Elektriska)
 - Device ID       : 0x01 (Nya Elektriska TDA)

RESOURCES
---------

A basic TDA card uses 2400 Bytes of computer RAM for the Text buffer and 
optionally can use 2KiB of ram for a user defined font. Also, could generate a 
interrupt for VSync events.

COMMANDS
--------

 - 0x0000 : **MAP_BUFFER** :  
   Sets text buffer address to B:A value. If is a valid RAM address, will set 
   C register to 0x0000. If not, will set C register to 0xFFFF and will ignore 
   the value. Setting B:A to 0, disables output.
 - 0x0001 : **MAP_FONT** :  
   Sets user font address to B:A value. If is a valid RAM address, will set C
   register to 0x0000. If not, will set C register to 0xFFFF. Setting B:A to 0,
   disables the user font, and restores the default font.
 - 0x0002 : **SET_INT** :  
   Sets interrupt message to A register value. If A is 0x0000, disables VSync 
   interrupt. At boot/reset time the VSync is disabled.

### Text buffer

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


### User defined font

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


### Color palette

The color palette in Little-endian RGB8 format (Arne 16 color palette) : 

 - 0   0x000000 Black
 - 1   0xEFDCB2 Light Blue
 - 2   0xF2A231 Mid Blue
 - 3   0xFF0000 Blue
 - 4   0x32261B Dark Blue
 - 5   0x27CEA3 Light Green
 - 6   0x1A8944 Green
 - 7   0x4E482F Swamp Green
 - 8   0x6BE2F7 Yellow 
 - 9   0x3189EB Copper
 - 10  0x2264A4 Brown
 - 11  0x2B3C49 Dark Brown
 - 12  0x8B6FE0 Pink
 - 13  0x3326BE Red
 - 14  0x9D9D9D Gray
 - 15  0xFFFFFF White

![Palette](img/palette.png "Palette")


