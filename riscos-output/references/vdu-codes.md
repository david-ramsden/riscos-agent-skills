# RISC OS VDU Codes Reference

This document summarises the VDU (Visual Display Unit) codes used in RISC OS for text and graphics output. VDU codes are byte values sent to the VDU driver to control screen display, cursor movement, colour selection, and printing.

## Overview

VDU codes range from 0 to 255. Codes 0-31 are control characters, code 127 is Delete, and codes 32-126 and 128-255 are printable characters (with 128-255 also usable for background colour selection). Code 23 is a special prefix that provides access to numerous miscellaneous sub-commands.

## Basic VDU Codes (0-31, 127)

| Code | Name | Syntax | Description |
|------|------|--------|-------------|
| 0 | Null | `VDU 0` | No operation. Used to pad unused parameters in multi-byte VDU 23 commands. |
| 1 | Send to printer | `VDU 1, n` | Sends the next byte `n` to the printer stream only (printer must be enabled with VDU 2). |
| 2 | Enable printer | `VDU 2` | Enables the printer stream. Characters sent to VDU drivers are also sent to the printer. |
| 3 | Disable printer | `VDU 3` | Disables the printer stream. |
| 4 | Split cursors | `VDU 4` | Splits text and graphics cursors (default mode). Text prints at text cursor position with text colours. |
| 5 | Join cursors | `VDU 5` | Joins text and graphics cursors. Text prints at graphics cursor position with graphics colours. Allows arbitrary text positioning. |
| 6 | Enable screen | `VDU 6` | Enables screen output (reverses VDU 21). |
| 7 | Bell | `VDU 7` | Sounds the bell. Bell parameters set via OS_Byte 211-214. |
| 8 | Backspace | `VDU 8` | Moves cursor back one position (negative X direction). Does not delete character. |
| 9 | Forward tab | `VDU 9` | Advances cursor one character width (positive X direction). |
| 10 | Line feed | `VDU 10` | Moves cursor down one line (positive Y direction). |
| 11 | Vertical tab | `VDU 11` | Moves cursor up one line (negative Y direction). |
| 12 | Clear screen | `VDU 12` | Clears the current window (text window in VDU 4 mode, graphics window in VDU 5 mode) and homes the cursor. |
| 13 | Carriage return | `VDU 13` | Moves cursor to start of current line (negative X direction at constant Y). |
| 14 | Page mode on | `VDU 14` | Enables page mode. Scrolling pauses until Shift is pressed. |
| 15 | Page mode off | `VDU 15` | Disables page mode. |
| 16 | Clear graphics | `VDU 16` | Clears the graphics window to the graphics background colour. Equivalent to BASIC's `CLG`. |
| 17 | Set text colour | `VDU 17, colour` | Sets text colour. Values 0-127 set foreground; 128-255 set background to `colour AND 128`. Actual colour used is `colour AND maxcol` (maxcol depends on mode). |
| 18 | Set graphics colour | `VDU 18, action, colour` | Sets graphics colour and plotting action. Action 0-7: foreground operations; 8-15: same with transparent background; 16-95: colour patterns 1-4; 80-95: giant pattern. |
| 19 | Set palette | `VDU 19, colour, mode, r, g, b` | Defines RGB palette values. Mode 0-15: logical to actual colour mapping; 16: RGB; 17/18: flash colours; 24: border; 25: pointer. Add 128 to set supremacy bit. |
| 20 | Restore colours | `VDU 20` | Restores default palette. Text/graphics foreground = white, background = black. Graphics actions = 0 (overwrite). |
| 21 | Disable screen | `VDU 21` | Disables screen output. Characters still sent to printer. Multi-byte VDU sequences are parsed but ignored. Reverse with VDU 6. |
| 22 | Change mode | `VDU 22, mode` | Selects screen mode. Use `OS_ScreenMode` in preference. Read current mode with `OS_Byte 135`. |
| 23 | Miscellaneous | `VDU 23, cmd, p1, p2, p3, p4, p5, p6, p7, p8, p9` | Prefix for miscellaneous commands. Nine parameters must always be present (use `\|` in BASIC to supply eight zero bytes). |
| 24 | Graphics window | `VDU 24, lx; by; rx; ty;` | Defines graphics window. Coordinates are in OS units, relative to graphics origin, and are **inclusive**. |
| 25 | Plot | `VDU 25, command, x; y;` | Executes a plot command. Limited to signed 16-bit coordinates. Prefer `OS_Plot` SWI for 32-bit coordinates. |
| 26 | Restore windows | `VDU 26` | Restores default (full screen) text and graphics windows. Resets graphics origin to (0,0), moves cursors to home. |
| 27 | Null | `VDU 27` | No operation. Issued when Escape is pressed. |
| 28 | Text window | `VDU 28, lx, by, rx, ty` | Defines text window in character coordinates. `(lx, by)` is bottom-left, `(rx, ty)` is top-right. |
| 29 | Graphics origin | `VDU 29, x; y;` | Sets the graphics origin (0,0) position. All graphics operations are relative to this point. |
| 30 | Home cursor | `VDU 30` | Moves cursor to home position (usually top-left of window). Text cursor in VDU 4 mode, graphics cursor in VDU 5 mode. |
| 31 | Position cursor | `VDU 31, x, y` | Moves text cursor to `(x, y)` relative to text window top-left. In VDU 5 mode, moves graphics cursor to `(x * x_spacing, y * y_spacing)`. |
| 127 | Delete | `VDU 127` | Deletes the previous character. Fills character position with background colour. |

## VDU 23 Sub-commands

VDU 23 provides access to miscellaneous functions. The command byte follows VDU 23, then nine parameters (which must always be supplied, even if unused).

### VDU 23 Command Summary

| Command | Description |
|---------|-------------|
| `23, 0` | Set interlace and cursor appearance |
| `23, 1` | Control cursor visibility |
| `23, 2-5` | Define ECF (Extended Character Fill) pattern and colours |
| `23, 6` | Set dot-dash line style |
| `23, 7` | Scroll text window or screen |
| `23, 8` | Clear block of text window |
| `23, 9` | Set duration for first flash colour |
| `23, 10` | Set duration for second flash colour |
| `23, 11` | Set Master 128 compatible pattern mode |
| `23, 12-15` | Define simple ECF pattern and colours |
| `23, 16` | Control cursor movement after printing |
| `23, 17` | Miscellaneous operations (tint, character size, etc.) |
| `23, 18` | Teletext emulation options (RISC OS 5.00+) |
| `23, 19-25, 28-31` | Unknown VDU 23 vector (`ukvdu23v`) |
| `23, 32-255` | Redefine character in system font |

### VDU 23, 0 - Interlace and Cursor Appearance

```
VDU 23, 0, action, mode, 0, 0, 0, 0, 0, 0
```

| Action | Mode | Effect |
|--------|------|--------|
| 8 | 0 | Set interlace to opposite of current TV setting |
| 8 | 1 | Set interlace to current TV setting |
| 8 | &80 | Turn interlace off |
| 8 | &81 | Turn interlace on |
| 10 | 0-31 | Set cursor start line (bits 0-4); bits 5-6: 00=steady, 01=off, 02=fast flash, 03=slow flash |
| 11 | 0-255 | Set cursor end line |

### VDU 23, 1 - Cursor Control

```
VDU 23, 1, mode, 0, 0, 0, 0, 0, 0, 0
```

| Mode | Effect |
|------|--------|
| 0 | Hide cursor |
| 1 | Show cursor |
| 2 | Steady cursor |
| 3 | Flashing cursor |

Effect is cancelled when cursor editing occurs.

### VDU 23, 2-5 - ECF Patterns (8-byte)

```
VDU 23, nr, byte1, byte2, byte3, byte4, byte5, byte6, byte7, byte8
```

Defines ECF pattern `nr-1` (nr = 2-5 for patterns 1-4). Each byte defines one row of the 8x8 pattern. Two interpretation modes exist (BBC and RISC OS), selected with VDU 23, 17, 4.

### VDU 23, 6 - Dot-Dash Line Style

```
VDU 23, 6, pattern_byte1, pattern_byte2, ..., pattern_byte8
```

Sets the dot-dash pattern for dotted plot commands. Most significant bit used first. Default pattern is &AAAAAAAA. Dash length set with `OS_Byte 163` (default 8).

### VDU 23, 7 - Scroll Window/Screen

```
VDU 23, 7, extent, direction, movement, 0, 0, 0, 0, 0
```

| Extent | Effect |
|--------|--------|
| 0 | Scroll text window |
| 1 | Scroll whole screen |

| Direction | Scroll Direction |
|-----------|-----------------|
| 0 | Right |
| 1 | Left |
| 2 | Down |
| 3 | Up |
| 4 | Positive X |
| 5 | Negative X |
| 6 | Positive Y |
| 7 | Negative Y |

| Movement | Effect |
|----------|--------|
| 0 | Scroll by one character cell |
| 1 | Scroll by one character cell vertically, one byte horizontally |

### VDU 23, 8 - Clear Text Window Block

```
VDU 23, 8, base_start, base_end, x1, y1, x2, y2, 0, 0
```

Clears a rectangular block of the text window to the text background colour. Base positions:

| Value | Meaning | Value | Meaning |
|-------|---------|-------|---------|
| 0 | Window top-left | 1 | Cursor column top |
| 2 | Window off top-right | 4 | Cursor line left |
| 5 | Cursor position | 6 | Cursor line off right |
| 8 | Window bottom-left | 9 | Cursor column bottom |
| 10 | Window off bottom-right | | |

("off" means one character beyond in the positive X direction)

### VDU 23, 9 - First Flash Duration

```
VDU 23, 9, duration, 0, 0, 0, 0, 0, 0, 0
```

Sets duration of first flash colour in video frames. Equivalent to `OS_Byte 9`. Duration 0 = display first colour constantly.

### VDU 23, 10 - Second Flash Duration

```
VDU 23, 10, duration, 0, 0, 0, 0, 0, 0, 0
```

Sets duration of second flash colour in video frames. Equivalent to `OS_Byte 10`.

### VDU 23, 11 - Master 128 Pattern Mode

```
VDU 23, 11, 0, 0, 0, 0, 0, 0, 0, 0
```

Sets Master 128 compatible pattern mode and resets colour patterns to mode defaults.

### VDU 23, 12-15 - Simple ECF Patterns

```
VDU 23, nr, n1, n2, n3, n4, n5, n6, n7, n8
```

Defines ECF pattern `nr-11` (nr = 12-15 for patterns 1-4).

For 2, 4, and 16 colour modes, defines a repeated 2x4 pixel block:

```
n1  n2
n3  n4
n5  n6
n7  n8
```

In 256 colour modes, the eight parameters define the eight rows.

### VDU 23, 16 - Cursor Movement Control

```
VDU 23, 16, x, y, 0, 0, 0, 0, 0, 0
```

Modifies the cursor movement control byte: `new_value = (old_value AND y) EOR x`

| Bit | Effect when set |
|-----|----------------|
| 0 | Scroll protection enabled |
| 1 | Positive horizontal direction is right (default left) |
| 2 | Positive vertical direction is down (default up) |
| 3 | X direction is vertical, Y is horizontal |
| 4 | Cursor wraps to opposite side when moving out of window in Y |
| 5 | Cursor does not move after printing |
| 6 | In VDU 5 mode, cursor movement beyond window edge does not generate newlines |
| 7 | Undefined (normally unset) |

### VDU 23, 17 - Miscellaneous Operations

| Sub-command | Description |
|-------------|-------------|
| `23, 17, 0, tint` | Set text foreground tint (256-colour modes) |
| `23, 17, 1, tint` | Set text background tint |
| `23, 17, 2, tint` | Set graphics foreground tint |
| `23, 17, 3, tint` | Set graphics background tint |
| `23, 17, 4, 0` | Use ancient (BBC) ECF pattern interpretation |
| `23, 17, 4, 1` | Use new RISC OS ECF pattern interpretation |
| `23, 17, 5` | Swap text foreground and background colours |
| `23, 17, 6, x; y;` | Set ECF origin alignment point |
| `23, 17, 7, flags, x; y;` | Set character size and spacing (VDU 5 mode) |

Tint values: &00, &40, &80, or &C0 (uses only top 2 bits).

For VDU 23, 17, 7 flags:
- Bit 0: (unimplemented) Set VDU 4 character size
- Bit 1: Set size of VDU 5 characters (8x8 and 8x16 are fast)
- Bit 2: Set spacing between VDU 5 characters

### VDU 23, 18 - Teletext Options (RISC OS 5.00+)

Supports Hi-Res teletext with 16x20 character grid (640x500 effective resolution).

| Sub-command | Description |
|-------------|-------------|
| `23, 18, 0, mode` | Switch transparency mode (0=Text, 1=Mix, 2=Box, 3=TV) |
| `23, 18, 1, suspend` | Suspend/resume bitmap updates (0=enabled, 1=suspended) |
| `23, 18, 2, reveal` | Reveal/conceal characters (0=conceal, 1=reveal) |
| `23, 18, 3, enable` | Enable/disable black foreground control codes (0=disable, 1=enable) |

### VDU 23, 32-255 - Character Redefinition

```
VDU 23, char_code, row1, row2, row3, row4, row5, row6, row7, row8
```

Redefines character `char_code` in the system font. Eight bytes define rows from top to bottom. Character patterns can be read with `OS_Word 10`.

Note: The delete character (127) can be redefined but still cannot be printed.

## VDU 4 vs VDU 5 Mode

| Feature | VDU 4 (Split) | VDU 5 (Joined) |
|---------|---------------|----------------|
| Cursor | Text cursor | Graphics cursor |
| Positioning | Character cell grid | Arbitrary pixel positions |
| Colours | Text foreground/background | Graphics foreground/action |
| Background | Filled with text background | Transparent (not plotted) |
| Speed | Faster | Slower |
| Cursor editing | Supported | Not supported |
| Text size | Fixed | Configurable via VDU 23, 17, 7 |

## Screen Modes (VDU 22)

Standard modes include:

| Mode | Columns | Resolution | Colours | Memory |
|------|---------|------------|---------|--------|
| 0 | 80x32 | 640x256 | 2 | 20K |
| 1 | 40x32 | 320x256 | 4 | 20K |
| 2 | 20x32 | 160x256 | 16 | 40K |
| 3 | 80x25 | Text only | 2 | 40K |
| 9 | 40x32 | 320x256 | 16 | 40K |
| 12 | 80x32 | 640x256 | 16 | 80K |
| 13 | 40x32 | 320x256 | 256 | 80K |
| 15 | 80x32 | 640x256 | 256 | 160K |
| 27 | 80x50 | 640x480 | 16 | 150K |
| 28 | 80x50 | 640x480 | 256 | 300K |
| 31 | 100x75 | 800x600 | 16 | 234K |
| 32 | 100x75 | 800x600 | 256 | 469K |

## Graphics Colour Actions (VDU 18)

| Action | Operation |
|--------|-----------|
| 0 | `s = c` (overwrite) |
| 1 | `s = s OR c` |
| 2 | `s = s AND c` |
| 3 | `s = s XOR c` |
| 4 | `s = NOT s` (invert) |
| 5 | `s = s` (no change) |
| 6 | `s = s AND NOT c` |
| 7 | `s = s OR NOT c` |
| 8-15 | As 0-7 but with transparent background |
| 16-31 | Colour pattern 1, using action 0-15 |
| 32-47 | Colour pattern 2, using action 0-15 |
| 48-63 | Colour pattern 3, using action 0-15 |
| 64-79 | Colour pattern 4, using action 0-15 |
| 80-95 | Giant pattern, using action 0-15 |

## Related OS Calls

- `OS_Byte 3` - Output stream status (printer/VDU control)
- `OS_Byte 117` - VDU status
- `OS_Byte 134` - Read text cursor position
- `OS_Byte 135` - Read current screen mode
- `OS_Byte 163` - Set dot-dash line length
- `OS_Word 10` - Read character patterns and ECF settings
- `OS_Plot` - Preferred alternative to VDU 25 (supports 32-bit coordinates)
- `OS_ScreenMode` - Preferred alternative to VDU 22
- `OS_ReadModeVariable` / `OS_ReadVduVariables` - Read mode information
- `OS_SetECFOrigin` - Alternative to VDU 23, 17, 6

## Notes

- Coordinates in VDU 24 (graphics window) are **inclusive** and relative to the graphics origin.
- VDU 25 is limited to signed 16-bit coordinates; use `OS_Plot` for larger resolutions.
- The desktop typically enables scroll protection (VDU 23, 16 bit 0) on startup.
- In BASIC, use `\|` after VDU 23 commands to supply the required eight zero padding bytes.
