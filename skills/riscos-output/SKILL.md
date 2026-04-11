---
name: riscos-output
description: Details of how the RISC OS output system works. Use when creating text and graphics output  using text positioning, graphics, drawing shapes, sprite or using fonts. It is not needed for plain textual output.
license: MIT
---
# RISC OS text and graphics output

This skill covers writing text, graphics, and fonts to the screen (or other output devices) on RISC OS.

## Terminology

* VDU output covers the character and control codes which are output to the display, and which control the positioning and colour through those codes. They are output through the system calls `OS_WriteC`, `OS_WriteN`, and `OS_Write0`. They are vectored through `WrchV` although this is usually not relevant. The sequences take their name from the `VDU` command in BBC BASIC.
* Graphics output covers the `OS_Plot` operations which produce lines, circles and simple shapes. It also includes sprite, draw font and other rendering systems.
* Draw provides graphics output using arbitrary paths, through the `Draw_*` SWI calls. For simple shapes it is not necessary to use Draw paths, but for curves and filled paths, it is useful to use the Draw operations.
* Outline fonts ('Fonts') are provided through the Font Manager, using `Font_*` SWI calls. Outline fonts can use any orientation and size and will be antialiased if possible.
* Sprite rendering is through the `OS_SpriteOp` system calls, and can render scaled and transformed images. Sprites have names, and attributes such as their 'mode', width and height. They may have palettes (if they are 256 colours or less) and masks (which may be 1BPP, or 8BPP).
* Sprite areas are collections of sprites. Sprite Files are collections of sprites in a file on disc.
* 'Modes' define pixel aspect ratios, number of colours and resolutions. They are commonly used to refer to screen display modes, but many parts of the system refer to modes for their properties rather than their resolution. Modes may be numbered (old style) or defined by a 'mode selector'.
* Colours may be managed through `COLOUR` (`VDU 17`) and `GCOL` (`VDU 18`) operations (which control the text and graphics colours respectively) in paletted modes. In non-paletted modes, and in code which is mode-independant, the `ColourTrans` operations are used to perform colour selection. When rendering sprites, you will always need to define a pixel translation table to allow the sprites to be rendered to the destination matching the intended colours.

## Coordinate system

### Graphics

Graphics co-ordinates on RISC OS use:

* Y=0 at the bottom, positive up the screen, negative below the screen.
* X=0 at the left, positive right across the screen, negative to the left of the screen.
* Units are (for most modes) double the resolution, so a 640x480 screen has addressable units of 1280x960.

Before you declare anything complete when drawing graphics, check that you've really got the co-ordinates right.

### Text

Text co-ordinates on RISC OS use:

* Y=0 at the top, positive down the screen.
* X=0 at the left, positive right across the screen.

To write text on the screen at the text cursor at a specific position, use `VDU 31, <xchar>, <ychar>`.
Text characters are 8 pixels by 8 pixels.

## Simple plotting

* SWI `OS_Plot` is used for plot operations.
* SWI `OS_Plot 4, x, y` is used to move the graphics cursor.
* SWI `OS_Plot 5, x, y` is used to draw a line to a new position.
* SWI `OS_Plot 85, x, y` is used to draw a filled triangle (using the prior two points and the new point).
* SWI `OS_Plot 69, x, y` is used to plot a single point.
* SWI `OS_Plot 101, x, y` is used to plot a rectangle (using the prior point and the current point).

## Fonts

Coordinates: **millipoints** = 1/72000 inch (internal); **OS units** = 1/180 inch (screen); default 400 millipoints per OS unit. Point sizes passed to `Font_FindFont` are multiplied by 16.

### Font Handles

| SWI | Number | Summary |
|-----|--------|---------|
| `Font_FindFont` | &40081 | Obtain a handle for a named font at a given size (R1=id, R2/R3=x/y size×16, R4/R5=DPI). |
| `Font_LoseFont` | &40082 | Release a font handle; decrements usage count (must match every FindFont). |
| `Font_ReadDefn` | &40083 | Read identifier string, point sizes, DPI, age, and usage count for a handle. |
| `Font_SetFont` | &4008A | Set the current font for subsequent Paint and measurement calls. |
| `Font_CurrentFont` | &4008B | Query current handle, background/foreground colours, and anti-alias offset. |
| `Font_FutureFont` | &4008C | Return the font/colour state that will result after the next string scan. |

### Sizing and Measurement

| SWI | Number | Summary |
|-----|--------|---------|
| `Font_ReadInfo` | &40084 | Read the overall bounding box of a font (all characters) in OS units. |
| `Font_CharBBox` | &4008E | Read the bounding box of a single character; R2 bit 4 selects millipoints/OS units. |
| `Font_StringBBox` | &40097 | Return the bounding box of a string in millipoints without rendering it. |
| `Font_StringWidth` | &40085 | Measure string width; stop at split character or max offset (deprecated, use ScanString). |
| `Font_ScanString` | &400A1 | Comprehensive scan: width, caret position, bounding box, split count; R2 flags select mode. |
| `Font_ReadFontMetrics` | &4009F | Read full IntMetrics data (bboxes, widths, kern pairs) into caller-supplied buffers. |

### Coordinate Conversion

| SWI | Number | Summary |
|-----|--------|---------|
| `Font_ConverttoOS` | &40088 | Convert millipoints → OS units (R1/R2 in, R1/R2 out). |
| `Font_Converttopoints` | &40089 | Convert OS units → millipoints (R1/R2 in, R1/R2 out; R0 corrupted). |
| `Font_ReadScaleFactor` | &4008F | Read millipoints-per-OS-unit factors (default 400 on both axes). |
| `Font_SetScaleFactor` | &40090 | Set millipoints-per-OS-unit factors (avoid under Wimp desktop). |

### Rendering

| SWI | Number | Summary |
|-----|--------|---------|
| `Font_Paint` | &40086 | Paint a string; R2 flags control coordinates (OS/mp), justification, kerning, matrix, R-to-L. |
| `Font_Caret` | &40087 | Paint a text cursor at a coordinate; R2 bit 4 selects millipoints/OS units. |
| `Font_FindCaret` | &4008D | Find nearest character position to given coordinates (deprecated; use ScanString). |
| `Font_FindCaretJ` | &40096 | As FindCaret but for a justified string (deprecated; use ScanString). |
| `Font_SwitchOutputToBuffer` | &4009E | Redirect Font_Paint output into a Draw-file path buffer (R1=buffer/0/8/−1). |
| `Font_MakeBitmap` | &40099 | Pre-render and cache a font at a specific size to speed subsequent use. |

### Colours and Anti-aliasing

| SWI | Number | Summary |
|-----|--------|---------|
| `Font_SetFontColours` | &40092 | Set font handle, background/foreground logical colours, and anti-alias offset (0–14). |
| `Font_SetPalette` | &40093 | Directly set physical background/foreground palette entries for anti-aliasing (avoid under Wimp). |
| `Font_ReadColourTable` | &40098 | Read the 16-entry logical colour table used for anti-aliasing into a buffer. |
| `Font_ReadThresholds` | &40094 | Read the anti-aliasing threshold table (controls offset vs. rendered size). |
| `Font_SetThresholds` | &40095 | Write a new anti-aliasing threshold table (rarely needed). |

### Cache and Configuration

| SWI | Number | Summary |
|-----|--------|---------|
| `Font_CacheAddr` | &40080 | Return Font Manager version (×100), total cache size, and used cache size. |
| `Font_ListFonts` | &40091 | Enumerate fonts or build a Wimp font menu; call repeatedly until R2 returns −1. |
| `Font_DecodeMenu` | &400A0 | Convert a Wimp menu selection back to a font identifier string. |
| `Font_ReadFontMax` | &4009C | Read FontMax and FontMax1–5 cache threshold settings. |
| `Font_SetFontMax` | &4009B | Set FontMax and FontMax1–5, triggering a cache reorganisation. |
| `Font_UnCacheFile` | &4009A | Evict (R2=0) or reload (R2=1) a specific font file from/into the cache. |
| `Font_ReadFontPrefix` | &4009D | Return the full directory path for a font handle's source directory. |

### Key Constants and Formats

| Item | Value / Format |
|------|----------------|
| Millipoints per inch | 72 000 |
| OS units per inch | 180 |
| Millipoints per OS unit (default) | 400 |
| Point size encoding | points × 16 |
| Palette entry format | `&BBGGRR00` |
| Max anti-alias offset | 14 (15 levels) |
| `Font_FindFont` DPI | 0 = default 90; −1 = current screen |
| String terminators | null (0) or CR (13) |


## ColourTrans

All palette entries use `&BBGGRR00` format. Fixed-point values use 16.16 format (1.0 = &10000).

### Graphics Colours

| SWI | Number | Summary |
|-----|--------|---------|
| `ColourTrans_ReturnGCOL` | &40742 | Return closest GCOL to a palette entry in the current mode (does not set it). |
| `ColourTrans_SetGCOL` | &40743 | Set graphics foreground or background to the closest GCOL; R3 flags select fg/bg/ECF, R4 is GCOL action. |
| `ColourTrans_ReturnGCOLForMode` | &40745 | Return closest GCOL for a specified mode (R1) and palette (R2). |
| `ColourTrans_ReturnOppGCOL` | &40747 | Return the most contrasting GCOL to a palette entry in the current mode. |
| `ColourTrans_SetOppGCOL` | &40748 | Set graphics colour to the most contrasting GCOL; useful for readable overlaid graphics. |
| `ColourTrans_ReturnOppGCOLForMode` | &4074A | Return most contrasting GCOL for a specified mode and palette. |
| `ColourTrans_ReturnColourNumber` | &40744 | Return closest colour number (256-colour modes) for a palette entry. |
| `ColourTrans_ReturnColourNumberForMode` | &40746 | Return closest colour number for a specified mode and palette. |
| `ColourTrans_GCOLToColourNumber` | &4074C | Convert a GCOL to a colour number (256-colour modes only). |
| `ColourTrans_ColourNumberToGCOL` | &4074D | Convert a colour number to a GCOL (256-colour modes only). |
| `ColourTrans_SetColour` | &4075E | Apply a known GCOL directly to foreground/background without colour matching. |

### Text Colours

| SWI | Number | Summary |
|-----|--------|---------|
| `ColourTrans_SetTextColour` | &40761 | Set text foreground or background to the closest GCOL; R3 bit 7 selects fg/bg. |
| `ColourTrans_SetOppTextColour` | &40762 | Set text colour to the most contrasting GCOL; ensures readable text on any background. |

## Font Colours (Anti-aliased)

| SWI | Number | Summary |
|-----|--------|---------|
| `ColourTrans_ReturnFontColours` | &4074E | Return background/foreground logical colours and maximum anti-alias offset for a font without setting them. |
| `ColourTrans_SetFontColours` | &4074F | Find the best anti-alias colour ramp and set it in the Font Manager; R3 requests up to 14 intermediate levels. |
| `ColourTrans_InvalidateCache` | &40750 | Flush ColourTrans's colour cache after external palette changes (VDU 19, Wimp_SetPalette, etc.). |

### Sprite Pixel Translation Tables

| SWI | Number | Summary |
|-----|--------|---------|
| `ColourTrans_SelectTable` | &40740 | Build a pixel translation table mapping source sprite colours to destination mode/palette; pass R4=0 to query buffer size first. |
| `ColourTrans_SelectGCOLTable` | &40741 | Build a simple GCOL list for each source palette entry, translated for the destination mode. |
| `ColourTrans_GenerateTable` | &40763 | Enhanced `SelectTable` with strict flag validation; preferred in RISC OS 3.10+. |

### Colour Conversion

| SWI | Number | Summary |
|-----|--------|---------|
| `ColourTrans_ConvertRGBToCIE` | &40755 | Convert RGB to CIE XYZ (all values 16.16 fixed-point, 0–1). |
| `ColourTrans_ConvertCIEToRGB` | &40756 | Convert CIE XYZ back to RGB. |
| `ColourTrans_ConvertRGBToHSV` | &40758 | Convert RGB to hue (0–360°), saturation, and value. |
| `ColourTrans_ConvertHSVToRGB` | &40759 | Convert HSV back to RGB. |
| `ColourTrans_ConvertRGBToCMYK` | &4075A | Convert RGB to cyan, magenta, yellow, key (black) for print separations. |
| `ColourTrans_ConvertCMYKToRGB` | &4075B | Convert CMYK back to RGB. |
| `ColourTrans_ConvertDeviceColour` | &40753 | Map a single device colour to a standard colour via the calibration table. |
| `ColourTrans_ConvertDevicePalette` | &40754 | Batch-convert an array of device palette entries to standard colours. |

### Palette & Calibration

| SWI | Number | Summary |
|-----|--------|---------|
| `ColourTrans_ReadPalette` | &4075C | Read the palette for a mode or sprite; pass R2=0 to query required buffer size. |
| `ColourTrans_WritePalette` | &4075D | Write a palette buffer to the screen or a sprite. |
| `ColourTrans_SetCalibration` | &40751 | Load a new screen calibration table for device-independent colour output. |
| `ColourTrans_ReadCalibration` | &40752 | Read the current calibration table; pass R0=0 to query size. |
| `ColourTrans_WriteCalibrationToFile` | &40757 | Export the current calibration as star commands to an open file. |



## Screen and sprite modes

### Key SWIs

| SWI | Number | Summary |
|-----|--------|---------|
| `OS_ScreenMode 0` | &65, R0=0 | Select a mode; R1 = mode number, selector pointer, or mode string. |
| `OS_ScreenMode 1` | &65, R0=1 | Return current mode specifier in R1. |
| `OS_ScreenMode 2` | &65, R0=2 | Enumerate available modes into a buffer; call repeatedly until done. |
| `OS_CheckModeValid` | &26 | Validate a mode specifier; N set if invalid, R1 = suggested alternative. |
| `OS_ReadModeVariable` | &35 | Read one mode variable (R0=mode/-1, R1=var number) → R2=value, C if invalid. |
| `OS_ReadVduVariables` | &31 | Read multiple mode/VDU variables in one call (R0=input list, R1=output buffer). |
| `OS_Byte 135` | — | R1 = char at cursor, R2 = current mode number (0–127 only). |

### VDU Sequence

| Sequence | Effect |
|----------|--------|
| `VDU 22, n` | Change to mode *n* (0–127; add 128 for shadow bank). |

### Mode Variable Numbers (for OS_ReadModeVariable / OS_ReadVduVariables)

| Number | Name | Returns |
|--------|------|---------|
| 0 | `ModeFlags` | Bit flags: bit 0=non-graphic, bit 1=teletext, bit 2=gap, bit 4=colour |
| 1 | `ScrRCol` | Text columns − 1 |
| 2 | `ScrBCol` | Text rows − 1 |
| 3 | `NColour` | Colours − 1 (1, 3, 15, 255, 65535, &FFFFFFFF) |
| 4 | `XEigFactor` | OS units per pixel in X = 2^n |
| 5 | `YEigFactor` | OS units per pixel in Y = 2^n |
| 6 | `LineLength` | Bytes per screen row |
| 7 | `ScreenSize` | Total screen memory in bytes |
| 8 | `YShftFactor` | log₂(LineLength/2) — for fast row arithmetic |
| 9 | `Log2BPP` | log₂(bits per pixel): 0=1bpp … 5=32bpp |
| 10 | `Log2BPC` | log₂(bits per character cell) |
| 11 | `XWindLimit` | Rightmost pixel column (pixels − 1) |
| 12 | `YWindLimit` | Topmost pixel row (pixels − 1) |

### Mode Selector Block (for arbitrary modes)

```
DCD 1           ; flags: bit 0=1, bits 1-7=format (0)
DCD x_pixels    ; X resolution
DCD y_pixels    ; Y resolution
DCD depth_code  ; 0=1bpp, 1=2bpp, 2=4bpp, 3=8bpp, 4=16bpp, 5=32bpp
DCD frame_rate  ; Hz, or -1 for highest available
DCD -1          ; terminator (optional variable pairs precede this)
```

### Mode String Format

`"X640 Y480 C256 F60"` — space or comma separated:
`X` = width, `Y` = height, `C` = colours (2/4/16/64/256/32K/16M), `F` = Hz, `EX`/`EY` = EIG factors.

### Common Mode Reference

| Mode | Resolution | OS units | Colours | bpp | Hz | Notes |
|------|-----------|----------|---------|-----|----|-------|
| 12 | 640×256 | 1280×1024 | 16 | 4 | 50 | Classic RISC OS desktop; all monitors |
| 13 | 320×256 | 1280×1024 | 256 | 8 | 50 | Games/paint; wide pixels (XEig=2) |
| 15 | 640×256 | 1280×1024 | 256 | 8 | 50 | Highest-colour 50 Hz mode; 160K |
| 27 | 640×480 | 1280×960 | 16 | 4 | 60 | VGA; square pixels; type 1,3,4,5 |
| 28 | 640×480 | 1280×960 | 256 | 8 | 60 | VGA 256-colour desktop; 300K |
| 31 | 800×600 | 1600×1200 | 16 | 4 | 56 | SVGA; type 1,4 only; 234K |

### Pixel Depth Encoding Summary

| Code | bpp | NColour | Log2BPP |
|------|-----|---------|---------|
| 0 | 1 | 1 | 0 |
| 1 | 2 | 3 | 1 |
| 2 | 4 | 15 | 2 |
| 3 | 8 | 255 | 3 |
| 4 | 16 | 65535 | 4 |
| 5 | 32 | &FFFFFFFF | 5 |

### Monitor Types

| Type | Standard | Typical Hz |
|------|----------|-----------|
| 0 | TV / PAL | 50 |
| 1 | Multi-frequency | 50–64 |
| 2 | 64 Hz hi-res mono | 64 |
| 3 | VGA | 60 |
| 4 | Super-VGA | 56–75 |
| 5 | LCD | varies |

### Key Mode Facts

- OS units per pixel = `1 << EigFactor` (typically XEig=1→2 OS units; YEig=2→4 OS units)
- Shadow bank: add 128 to mode number; OS_Byte 112/113 control write/display bank
- VIDC1 (Archimedes): 8bpp = only 64 base colours + 4 tints; VIDC20 (RiscPC+): full 256 palette
- Mode selectors require RISC OS 3.5+; mode strings require RISC OS 3.5+

## Sprites

All sprite operations use `OS_SpriteOp` (SWI &2E). R0 encodes both the reason code and the sprite addressing mode.

### R0 Encoding

```
R0 = reason_code + addressing_flags
```

| Addition | R1 | R2 |
|----------|----|----|
| +0 | (system area, not used) | Pointer to sprite name |
| +256 | Pointer to user sprite area | Pointer to sprite name |
| +512 | Pointer to user sprite area | Pointer to sprite control block |

### Sprite Area / File Operations

| Reason | R0 base | Operation |
|--------|---------|-----------|
| 8 | 8 | Read area control block → R2=size, R3=count, R4=first offset, R5=free offset |
| 9 | 9 | Initialise sprite area (pre-fill offset 0 = size, offset 8 = 16) |
| 10 | 10+flags | Load sprite file, overwriting existing sprites |
| 11 | 11+flags | Merge sprite file into area |
| 12 | 12+flags | Save sprite area to file |
| 2 | 2 | Screen save to file (R2=filename, R3=palette flag) |
| 3 | 3 | Screen load from file, changing mode if needed |

### Sprite Creation and Capture

| Reason | R0 base | Operation |
|--------|---------|-----------|
| 15 | 15+flags | Create blank sprite (R2=name, R3=palette, R4=w, R5=h, R6=mode) |
| 14 | 14+flags | Get sprite from current graphics window (R2=name, R3=palette) |
| 16 | 16+flags | Get sprite from explicit OS-unit coords (R4=x0, R5=y0, R6=x1, R7=y1) |

### Sprite Information and Lookup

| Reason | R0 base | Operation |
|--------|---------|-----------|
| 13 | 13+flags | Return sprite name by index → R3=length (R4=index, R2=buffer) |
| 40 | 40+flags | Read sprite info → R3=w px, R4=h px, R5=mask, R6=mode |
| 37 | 37+flags | Palette: R3=−1 read, 0 remove, 1+ create |

### Sprite Modification

| Reason | R0 base | Operation |
|--------|---------|-----------|
| 25 | 25+flags | Delete sprite |
| 26 | 26+flags | Rename sprite (R3=new name) |
| 27 | 27+flags | Copy sprite (R3=new name) |
| 29 | 29+flags | Create mask (initialised solid) |
| 30 | 30+flags | Remove mask |
| 31 | 31+flags | Insert row at R3 (0=bottom) |
| 32 | 32+flags | Delete row R3 |
| 33 | 33+flags | Flip about X axis (vertical mirror) |
| 45 | 45+flags | Insert column at R3 (0=left) |
| 46 | 46+flags | Delete column R3 |
| 47 | 47+flags | Flip about Y axis (horizontal mirror) |
| 35 | 35+flags | Append sprite (R2=dest, R3=src, R4=0 horiz/1 vert) |
| 54 | 54+flags | Remove left-hand wastage (word-align left edge) |
| 57 | 57+flags | Insert/delete multiple rows (R3=start, R4=count ±) — *SpriteExtend* |
| 58 | 58+flags | Insert/delete multiple columns — *SpriteExtend* |

### Pixel Access

| Reason | R0 base | Operation |
|--------|---------|-----------|
| 41 | 41+flags | Read pixel colour → R5=colour, R6=tint (R3=x, R4=y) |
| 42 | 42+flags | Write pixel colour (R3=x, R4=y, R5=colour, R6=tint) |
| 43 | 43+flags | Read mask pixel → R5=0 transparent/1 solid (R3=x, R4=y) |
| 44 | 44+flags | Write mask pixel (R3=x, R4=y, R5=0 or 1) |

### Rendering

| Reason | R0 base | Operation |
|--------|---------|-----------|
| 28 | 28+flags | Put sprite at graphics cursor (R5=action) |
| 34 | 34+flags | Put sprite at explicit coords (R3=x, R4=y, R5=action) |
| 48 | 48+flags | Plot mask at graphics cursor |
| 49 | 49+flags | Plot mask at explicit coords (R3=x, R4=y) |
| 52 | 52+flags | Put sprite scaled (R3=x, R4=y, R5=action, R6=scale, R7=table) — *SpriteExtend* |
| 53 | 53+flags | Put sprite grey-scaled — *SpriteExtend*, 4bpp grey only |
| 50 | 50+flags | Plot mask scaled (R6=scale) — *SpriteExtend* |
| 51 | 51 | Paint character scaled (R1=char, R6=scale) — *SpriteExtend* |
| 56 | 56+flags | Put sprite transformed (R3=flags, R5=action, R6=matrix/coords, R7=table) — *SpriteExtend* |
| 55 | 55+flags | Plot mask transformed — *SpriteExtend* |
| 24 | 24+flags | Select sprite for VDU 25,232-239 plotting |
| 36 | 36+flags | Set hardware pointer shape from sprite |

### Output Redirection

| Reason | R0 base | Operation |
|--------|---------|-----------|
| 62 | 62+flags | Query save area size → R3=bytes needed (R2=0 for screen) |
| 60 | 60+flags | Switch output to sprite image (R2=sprite or 0=screen, R3=save area) |
| 61 | 61+flags | Switch output to sprite mask (same parameters) |

### Plot Action Values (R5)

| Value | Effect | +8 |
|-------|--------|-----|
| 0 | Overwrite | Use mask |
| 1 | OR | Use mask |
| 2 | AND | Use mask |
| 3 | XOR | Use mask |
| 4 | Invert screen | Use mask |
| 5 | No change | — |
| 6 | AND NOT | Use mask |
| 7 | OR NOT | Use mask |

### Scale Factors Block (R6 for scaled operations)

```
DCD  x_mult, y_mult   ; multipliers
DCD  x_div,  y_div    ; divisors
; scaled size = original × mult / div
; R6 = 0  →  no scaling (1:1 plot)
```

### Sprite Mode Word

```
mode_word = (type << 27) | (ydpi << 14) | (xdpi << 1) | 1
```

| Type | bpp | Colours |
|------|-----|---------|
| 1 | 1 | 2 |
| 2 | 2 | 4 |
| 3 | 4 | 16 |
| 4 | 8 | 256 |
| 5 | 16 | 32K |
| 6 | 32 | 16M |

Bit 31 set = alpha channel; clear = binary mask. Typical DPI values: 180, 90, 45, 22.

### Key Sprite Facts

- Sprite names: max 12 characters, case-insensitive.
- Row 0 = bottom; column 0 = left.
- Mask bits: 1 = solid, 0 = transparent.
- New-type sprite masks are always 1bpp regardless of image depth.
- Sprite palettes in new-type sprites require RISC OS 3.6+.
- SpriteExtend reason codes (50–58) require the SpriteExtend module.
- While output is redirected to a sprite, do not call `Wimp_Poll`.



## References

* For details about `VDU` control codes, see [references/vdu-codes.md](references/vdu-codes.md).

* For details about `OS_Plot`, codes see [references/os_plot.md](references/os_plot.md).

* For details about Draw, see [references/draw.md](references/draw.md).
* For details about DrawFiles, see [references/drawfile.md](references/drawfile.md).

* For details about text colours with ColourTrans, see [references/colourtrans-text-colours.md](references/colourtrans-text-colours.md).
* For details about graphics colours with ColourTrans, see [references/colourtrans-graphics-colours.md](references/colourtrans-graphics-colours.md).
* For details about Font colours, see [references/colourtrans-font-colours.md](references/colourtrans-font-colours.md).
* For details about pixel translation tables for sprite colours, see [references/colourtrans-sprite-translation.md](references/colourtrans-sprite-translation.md).
* For details about converting colour types with ColourTrans, see [references/colourtrans-colour-conversion.md](references/colourtrans-colour-conversion.md).

* For details about Font rendering, see [references/font-rendering.md](references/font-rendering.md).
* For details about Font sizing, see [references/font-sizing.md](references/font-sizing.md).
* For details about Font system configuration, see [references/font-manager-config.md](references/font-manager-config.md).
* For details about Font files, see [references/font-files-reference.md](references/font-files-reference.md).
* For details about Font colours, see [references/font-colours.md](references/font-colours.md).

* For details about reading mode information, see [references/modes-mode-information.md](references/modes-mode-information.md).
* For details about mode numbers, see [references/modes-mode-numbers.md](references/modes-mode-numbers.md).
* For details about selecting screen modes and reading information about available modes, see [references/modes-mode-selection.md](references/modes-mode-selection.md).
* For details about specifiers used to select and describe screen modes, see [references/modes-mode-specifiers.md](references/modes-mode-specifiers.md).

* For details about sprite areas and the sprite format, see [references/sprites-areas-and-format.md](references/sprites-areas-and-format.md).
* For details about sprite rendering, see [references/sprites-rendering.md](references/sprites-rendering.md).
* For details about sprite redirection (changing the graphics destination to a sprite), see [references/sprites-redirection.md](references/sprites-redirection.md).
* For details about sprite creation and modification, see [references/sprites-creation-modification.md](references/sprites-creation-modification.md).
* For details about sprite area management (initialisation, loading, saving, copying, reading information, deleting, etc), see [references/sprites-management.md](references/sprites-management.md).
