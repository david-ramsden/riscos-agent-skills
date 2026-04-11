# OS_Plot and RISC OS Drawing System Reference

## Overview

RISC OS provides a graphics drawing system based on OS units (external coordinates). The primary
drawing call is `OS_Plot` (SWI &45), equivalent to `VDU 25` but significantly faster as it bypasses
the VDU stream overhead.

Coordinates are specified in OS units. The origin (0,0) is normally at the bottom-left of the
screen; y increases upward. The graphics origin can be relocated with `VDU 29`.

---

## OS_Plot — SWI &45

Direct plot command, bypasses the VDU stream unless spooling or redirection is active, or WrchV
has been claimed.

**On entry:**
- R0 = plot code
- R1 = x coordinate (OS units, relative to graphics origin)
- R2 = y coordinate (OS units, relative to graphics origin)

**On exit:**
- R0–R2 corrupted

**C usage:**
```c
#include "swis.h"
_swix(OS_Plot, _INR(0,2), plot_code, x, y);
```

---

## Plot Code Structure

The plot code is an 8-bit value. The lower 3 bits select the **plotting manner**; the upper 5 bits
(multiples of 8) select the **drawing operation**.

### Lower 3 bits — Plotting Manner

| Bits 2:0 | Effect                                         |
|----------|------------------------------------------------|
| 0        | Move cursor, relative to previous point        |
| 1        | Draw using foreground colour, relative         |
| 2        | Draw using inverse colour, relative            |
| 3        | Draw using background colour, relative         |
| 4        | Move cursor, absolute                          |
| 5        | Draw using foreground colour, absolute         |
| 6        | Draw using inverse colour, absolute            |
| 7        | Draw using background colour, absolute         |

"Relative" means the coordinate is an offset from the last graphics cursor position.
"Absolute" means the coordinate is relative to the current graphics origin.

### Upper 5 bits (× 8) — Drawing Operation

| Base code | Operation                                              |
|-----------|--------------------------------------------------------|
| 0         | Solid line, both endpoints included                    |
| 8         | Solid line, excluding the final point                  |
| 16        | Dotted line, both endpoints included, pattern restarted|
| 24        | Dotted line, excluding final point, pattern restarted  |
| 32        | Solid line, excluding the initial point                |
| 40        | Solid line, both endpoints excluded                    |
| 48        | Dotted line, initial point excluded, pattern continued |
| 56        | Dotted line, both endpoints excluded, pattern continued|
| 64        | Single pixel plot                                      |
| 72        | Horizontal line fill — rightward to non-background     |
| 80        | Triangle fill (uses current, previous, and new point)  |
| 88        | Horizontal line fill — rightward to background         |
| 96        | Rectangle fill (uses current point and new point)      |
| 104       | Horizontal line fill — leftward and rightward to foreground |
| 112       | Parallelogram fill                                     |
| 120       | Horizontal line fill — rightward to non-foreground     |
| 128       | Flood fill to non-background colour                    |
| 136       | Flood fill to foreground colour                        |
| 144       | Circle outline (centre = previous point, passes through new point) |
| 152       | Circle fill                                            |
| 160       | Circular arc (centre = older point, start = previous, end = new)  |
| 168       | Chord (segment — filled area between chord and arc)    |
| 176       | Sector (pie slice — filled area between two radii and arc) |
| 184       | Block copy/move (relative)                             |
| 192       | Ellipse outline                                        |
| 200       | Ellipse fill                                           |
| 208       | Reserved / font printing                               |
| 232       | Sprite plot                                            |
| 240–248   | User-defined graphics operations                       |

### Common Combined Codes

| Code | Meaning                                              |
|------|------------------------------------------------------|
| 4    | Move cursor to absolute position                     |
| 0    | Move cursor to relative position                     |
| 5    | Draw solid line to absolute position (FG, incl. end) |
| 25   | Draw dotted line to absolute (both endpoints)        |
| 69   | Plot a single point at absolute position             |
| 85   | Fill triangle (absolute)                             |
| 101  | Fill rectangle (absolute)                            |
| 149  | Draw circle outline (absolute)                       |
| 153  | Fill circle (absolute)                               |
| 161  | Draw circular arc (absolute)                         |
| 177  | Draw sector / pie slice (absolute)                   |
| 189  | Block copy (absolute)                                |
| 193  | Draw ellipse outline (absolute)                      |
| 201  | Fill ellipse (absolute)                              |

---

## Colour and Plotting Action

### VDU 18 — Set Graphics Colour

```
VDU 18,action,colour
```

- **action** (0–95): Selects the logical plotting operation and ECF pattern.
- **colour** (0–127): Foreground colour. Add 128 to set background colour instead.

**Action values:**

| Value  | Operation                                     |
|--------|-----------------------------------------------|
| 0      | Overwrite screen pixel with colour            |
| 1      | OR colour with screen pixel                   |
| 2      | AND colour with screen pixel                  |
| 3      | XOR colour with screen pixel                  |
| 4      | Invert screen pixel                           |
| 5      | Leave screen pixel unchanged                  |
| 6      | AND screen pixel with NOT colour              |
| 7      | OR screen pixel with NOT colour               |
| 8–15   | As 0–7 but background colour is transparent   |
| 16–79  | ECF patterns 1–4 combined with actions 0–15   |
| 80–95  | Giant combined ECF patterns                   |

### OS_SetColour — SWI &61

Preferred programmatic alternative to `VDU 18`.

**On entry:**
- R0 = flags:
  - bits 0–3: graphics plotting action (0–15, same as VDU 18 table above)
  - bit 4: set = alter background, clear = alter foreground
  - bit 5: set = R1 is pattern data pointer, clear = R1 is colour number
- R1 = colour number (from `ColourTrans_ReturnColourNumber`) or pattern data pointer

**C usage:**
```c
/* Set foreground to colour number, overwrite action */
_swix(OS_SetColour, _INR(0,1), 0, colour_number);

/* Set background to colour number, overwrite action */
_swix(OS_SetColour, _INR(0,1), (1 << 4), colour_number);
```

### VDU 19 — Set Palette

```
VDU 19,logical_colour,mode,red,green,blue
```

Maps a logical colour to a physical RGB value. Only the top 4 bits of each colour component are
significant (multiply by 17 to convert an 8-bit value to the correct range).

| Mode | Meaning                                   |
|------|-------------------------------------------|
| 0–15 | Set physical colour directly              |
| 16   | Set both flash colours simultaneously     |
| 17   | Set first flash colour                    |
| 18   | Set second flash colour                   |
| 24   | Set border colour                         |
| 25   | Set pointer colour                        |

---

## ColourTrans — Colour Selection for High Colour Modes

In 256-colour and higher modes the simple GCOL number system is insufficient. Use ColourTrans to
select the closest available colour to a given RGB palette entry.

### ColourTrans_ReturnColourNumber — SWI &40744

Returns the colour number closest to a given palette entry in the current mode.

**On entry:**
- R0 = palette entry (0x**BBGGRR**00 — blue, green, red in bits 31:8, low byte zero)

**On exit:**
- R0 = colour number

**C usage:**
```c
int colour;
_swix(ColourTrans_ReturnColourNumber, _IN(0)|_OUT(0),
      (blue << 24) | (green << 16) | (red << 8), &colour);
```

### ColourTrans_SetGCOL — SWI &40743

Finds the closest GCOL to a palette entry and sets it as the current colour.

**On entry:**
- R0 = palette entry (same format as above)
- R3 = flags:
  - bit 7: set = set background, clear = set foreground
  - bit 8: set = use ECFs for better colour approximation
- R4 = GCOL action (0–15)

**On exit:**
- R0 = GCOL number chosen

**C usage:**
```c
/* Set foreground colour, overwrite action */
_swix(ColourTrans_SetGCOL, _INR(0,0)|_INR(3,4),
      (blue << 24) | (green << 16) | (red << 8),
      0,      /* foreground, no ECF */
      0);     /* overwrite action */
```

### ColourTrans_SetColour — SWI &4075E

Sets the foreground or background colour from a GCOL number.

**On entry:**
- R0 = GCOL number (from `ColourTrans_ReturnColourNumber`)
- R3 = flags:
  - bit 7: set = background, clear = foreground
  - bit 9: set = set text colours instead of graphics colours

---

## Graphics Window

### VDU 24 — Define Graphics Clipping Window

```
VDU 24,x1;y1;x2;y2;
```

Sets a rectangular clipping region. Graphics operations outside this window are clipped.
Coordinates are inclusive and relative to the graphics origin. (x1,y1) is bottom-left;
(x2,y2) is top-right.

### VDU 26 — Restore Default Windows

```
VDU 26
```

Resets both the text and graphics windows to full screen. Also resets the graphics origin to
(0,0), moves the graphics cursor to (0,0), and homes the text cursor.

---

## Graphics Origin

### VDU 29 — Set Graphics Origin

```
VDU 29,x;y;
```

Sets the point (x,y) — in absolute screen OS units — as the new graphics origin. All subsequent
`OS_Plot` coordinates are relative to this point. Moving the origin does not reposition any
previously defined graphics window; its coordinates shift relative to the new origin.

---

## Reading Graphics State

### OS_ReadPoint — SWI &32

Returns the colour of a pixel.

**On entry:**
- R0 = x coordinate (OS units, relative to graphics origin)
- R1 = y coordinate

**On exit:**
- R2 = colour number
- R3 = tint (0–255, amount of white mixed in)
- R4 = 0 if point is on screen; -1 if off screen (R2 = -1 also)

### OS_ReadVduVariables — SWI &31

Reads a sequence of VDU state variables into a caller-supplied buffer.

**On entry:**
- R0 = pointer to word-aligned input block (list of variable numbers, terminated by -1)
- R1 = pointer to word-aligned output block

**Key variable numbers for graphics:**

| Number | Name       | Meaning                                      |
|--------|------------|----------------------------------------------|
| 128    | GWLCol     | Left column of graphics window (internal)    |
| 129    | GWBRow     | Bottom row of graphics window (internal)     |
| 130    | GWRCol     | Right column of graphics window (internal)   |
| 131    | GWTRow     | Top row of graphics window (internal)        |
| 136    | OrgX       | x coordinate of graphics origin (external)   |
| 137    | OrgY       | y coordinate of graphics origin (external)   |
| 138    | GCsX       | x coordinate of graphics cursor (external)   |
| 139    | GCsY       | y coordinate of graphics cursor (external)   |
| 140    | OlderCsX   | x of oldest graphics cursor (internal)       |
| 141    | OlderCsY   | y of oldest graphics cursor (internal)       |
| 142    | OldCsX     | x of previous graphics cursor (internal)     |
| 143    | OldCsY     | y of previous graphics cursor (internal)     |
| 144    | GCsIX      | x of current graphics cursor (internal)      |
| 145    | GCsIY      | y of current graphics cursor (internal)      |
| 151    | GPLFMD     | GCOL action for foreground colour            |
| 152    | GPLBMD     | GCOL action for background colour            |
| 153    | GFCOL      | Graphics foreground colour                   |
| 154    | GBCOL      | Graphics background colour                   |
| 157    | GFTint     | Tint for graphics foreground                 |
| 158    | GBTint     | Tint for graphics background                 |

"(ic)" = internal coordinates; "(ec)" = external (OS unit) coordinates.

**C usage:**
```c
int input[]  = { 138, 139, -1 };  /* read GCsX, GCsY */
int output[2];
_swix(OS_ReadVduVariables, _INR(0,1), input, output);
/* output[0] = current x, output[1] = current y */
```

---

## Typical Drawing Patterns

### Move then draw a line

```c
_swix(OS_Plot, _INR(0,2), 4,  x1, y1);   /* move absolute */
_swix(OS_Plot, _INR(0,2), 5,  x2, y2);   /* draw line absolute, FG */
```

### Draw a filled rectangle

```c
_swix(OS_Plot, _INR(0,2), 4,   x1, y1);  /* move to bottom-left */
_swix(OS_Plot, _INR(0,2), 101, x2, y2);  /* fill rectangle to top-right */
```

### Draw a filled circle

```c
_swix(OS_Plot, _INR(0,2), 4,   cx, cy);  /* move to centre */
_swix(OS_Plot, _INR(0,2), 153, cx + r, cy); /* fill circle, radius r */
```

### Plot a single pixel

```c
_swix(OS_Plot, _INR(0,2), 69, x, y);     /* plot point absolute, FG */
```

### Set colour using ColourTrans and draw

```c
/* Palette entry format: 0xBBGGRR00 */
_swix(ColourTrans_SetGCOL, _INR(0,0)|_INR(3,4),
      (0 << 24) | (255 << 16) | (0 << 8),  /* green */
      0, 0);
_swix(OS_Plot, _INR(0,2), 4,   0,   0);
_swix(OS_Plot, _INR(0,2), 101, 200, 100);  /* green filled rectangle */
```

---

## Notes

- `OS_Plot` is preferred over `VDU 25` in C code for performance; it does not go through the
  VDU stream unless spooling, redirection, or WrchV interception is active.
- Coordinates are signed 32-bit integers; the visible screen typically spans from (0,0) to
  approximately (1280,1024) or (1024,768) depending on mode, but OS units can extend beyond.
- All `OS_Plot` operations update the graphics cursor position; subsequent relative-mode
  operations use the last cursor position as their origin.
- When drawing arcs, segments, and sectors, RISC OS uses three stored cursor positions:
  oldest (OlderCs), previous (OldCs), and current (NewPt). A sequence of `OS_Plot` calls
  builds up these positions before the final arc/sector call.
- `OS_ReadModeVariable` (SWI &35) with R0 = -1 reads variables for the current mode;
  relevant variables include pixel dimensions and bits-per-pixel.
