Text Generator Adapter
=====================
Version 0.1 (WIP) 

Text Generator Adapter (TGA) device that allows to display text with color and an optional RAM defined font.

- Allowed Text modes : 40x30 8x8 pixel font cell


The refresh rate should be around 25 hz.

- Device Class    : 0x0E (Graphics Device)
- Device Builder  : 0x1C6C8B36 (Nya Elektriska)
- Device ID       : 0x01 (TGA standard) 
- Device Rev      : 0x00  

RESOURCES
---------

A basic TGA card uses 2400 Bytes of computer RAM for the Text buffer and optionally can use 2KiB of ram for a user defined font. Also, could generate a interrupt for Vsync events.

COMMANDS
--------

    |  Value  |   Name   | Description
    +---------+----------+-----------------------------------------------------
    | 0x0000  | SET_ADDR | Sets base address to map to B:A value. If the 
    |         |          | address is valid, and the device can map these
    |         |          | address, will set C register to 0x0000. If not, will
    |         |          | set A register to 0xFFFF. Setting B:A to 0, disables
    |         |          | the address map, and disables the graphics output.
    | 0x0001  | GET_ADDR | Return base address maped. Sets B:A register to the 
    |         |          | base address being used by the device. When the 
    |         |          | computer boots, this base address is 0.
    | 0x0002  | GET_ASIZE| Returns the address block size. Sets A register to 
    |         |          | the address block size being used by the device. In 
    |         |          | this case is 4448
    | 0x0003  | SET_INT  | Sets interrupt message to A register value. If A is 
    |         |          | 0x0000, then disables VSync interrupt. This is set
    |         |          | to 0 when the computer boots.
    | 0x0004  | GET_INT  | Gets interrupt message. Sets A register to the value
    |         |          | of the interrupt message.
    | 0x0005  | SET_FONT | Points to the user Font base address  in B:A value.
    |         |          | If the address is valid, and the device can map these
    |         |          | address, will set C register to 0x0000. If not, will
    |         |          | set A register to 0xFFFF. Setting B:A to 0, disables
    |         |          | the address map, and restores the default font.
    +---------+----------+-----------------------------------------------------


### Text buffer

The Text buffer represents a glyph and the color attributes using a word to store both. In each word, is defined the glyph index to use and the background/foreground color attributes.
The first word in the Text Area is the character at row 0, column 0. The next
word is the character at row 0, column 1, etc.

Formula : address = text buffer base address + (column % MAX_COLUMNS)*2 + (MAX_COLUMNS*2 * row)
Were MAX_COLUMNS = 40 or other value depending of what text mode is.

The format of each word is (Little Endian format):

    15 14 13 12 11 10  9  8  7  6  5  4  3  2  1  0
    -----------------------------------------------
     b  b  b  b  f  f  f  f  g  g  g  g  g  g  g  g

- gggggggg Chooses one of the 256 glyphs in the font
- ffff Chooses the foreground color from the palette
- bbbb Chooses the background color from the palette


#### User defined font

Sending command SET_FONT with B:A pointing to a RAM address, enables the user defined font in RAM.
It uses the 2KiB of RAM to define a custom 8x8 font.
A glyph of the font, is defined by contiguous 8 bytes, being each byte,
a row of 8 pixels of the glyph, and the MSB bit defines the first pixel of the 
row. For example, to define the glyph of the
character 'F' :

    01111111 -> 0x7F
    01000000 -> 0x40
    01000000 -> 0x40
    01111000 -> 0x78
    01000000 -> 0x40
    01000000 -> 0x40
    01000000 -> 0x40
    01000000 -> 0x40

Formula : glyph address = (font ram base address + video ram size - 2048) + glyph index * 8


### Color palette

The color palette is (Arne 16 color palette):

- 0  <span style="background-color:#000000;">&nbsp;&nbsp;</span> 0x000000 Black
- 1  <span style="background-color:#b2dcef;">&nbsp;&nbsp;</span> 0xB2DCEF Light Blue
- 2  <span style="background-color:#31a2f2;">&nbsp;&nbsp;</span> 0x31A2F2 Mid Blue
- 3  <span style="background-color:#0000ff;">&nbsp;&nbsp;</span> 0x0000FF Blue
- 4  <span style="background-color:#1b2632;">&nbsp;&nbsp;</span> 0x1B2632 Dark Blue
- 5  <span style="background-color:#a3ce27;">&nbsp;&nbsp;</span> 0xA3CE27 Light Green
- 6  <span style="background-color:#44891a;">&nbsp;&nbsp;</span> 0x44891A Green
- 7  <span style="background-color:#2f484e;">&nbsp;&nbsp;</span> 0x2F484E Swamp Green
- 8  <span style="background-color:#f7e26b;">&nbsp;&nbsp;</span> 0xF7E26B Yellow 
- 9  <span style="background-color:#eb8931;">&nbsp;&nbsp;</span> 0xEB8931 Copper
- 10 <span style="background-color:#a46422;">&nbsp;&nbsp;</span> 0xA46422 Brown
- 11 <span style="background-color:#493c2b;">&nbsp;&nbsp;</span> 0x493C2B Dark Brown
- 12 <span style="background-color:#e06f8b;">&nbsp;&nbsp;</span> 0xE06F8B Pink
- 13 <span style="background-color:#be2633;">&nbsp;&nbsp;</span> 0xBE2633 Red
- 14 <span style="background-color:#9d9d9d;">&nbsp;&nbsp;</span> 0x9D9D9D Gray
- 15 <span style="background-color:#FFFFFF;">&nbsp;&nbsp;</span> 0xFFFFFF White

![Palette](./palette.png "Palette")





