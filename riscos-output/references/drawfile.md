# DrawFile Renderer and DrawFile Format Reference

## Overview

The DrawFile renderer module (RISC OS 3.5+) handles `.aff` files — serialised sequences of
Draw objects. It renders them to screen or printer, returns their bounding box, and declares
their fonts to the printer driver.

The Draw module path drawing API (SWIs, path data, transformation matrix, fill style, etc.)
is documented in `draw.md`. The transformation matrix format used here is identical to that
described there.

---

## Coordinate System

DrawFile coordinates are in *internal Draw units*: 1 OS unit = 256 internal Draw units.
To convert OS units to internal Draw units, multiply by 256.

The transformation matrix passed to the DrawFile SWIs uses the same format as the Draw module:
six 32-bit words (a, b, c, d, e, f) where a–d are in transform units (&00010000 = 1.0) and
e–f are translations in internal Draw units.

---

## DrawFile SWIs

### DrawFile_Render — SWI &45540

Renders a Draw file to the screen or to the printer (via printer driver interception).

**On entry:**
- R0 = flags:
  - bit 0: draw dotted red bounding box rectangles around each object (debug mode)
  - bit 1: do not render objects (dry-run / bounding box pass only)
  - bit 2: use R5 as the flatness parameter
- R1 = pointer to Draw file data in memory
- R2 = size of Draw file data in bytes
- R3 = pointer to transformation matrix, or 0 for identity (renders at screen origin)
- R4 = pointer to clipping rectangle — 4 words (x0, y0, x1, y1) in OS units — or 0 for no clipping
- R5 = flatness value (only used if bit 2 of R0 is set)

**On exit:** R0–R5 preserved

The rendering position is set by the translation components (e, f) of the matrix. With identity
matrix (R3 = 0) the file renders with its bottom-left at screen coordinate (0, 0).

The clipping rectangle is typically the redraw rectangle returned by `Wimp_RedrawWindow` /
`Wimp_GetRectangle`. Passing 0 renders the entire file regardless of the graphics window.

All rendering calls used internally are printer-driver-compatible, so this SWI works correctly
during a print redraw loop (after `PDriver_DrawPage`). Do not call `Wimp_ReportError` for any
error returned from within a Wimp redraw loop.

**Render at 1:1 scale at OS position (px, py):**
```c
#include "swis.h"
#include <stdint.h>

void render_drawfile(void *file_data, int file_size,
                     int px, int py, int *clip_rect)
{
    int32_t matrix[6];
    matrix[0] = 0x10000; matrix[1] = 0;
    matrix[2] = 0;       matrix[3] = 0x10000;
    matrix[4] = px * 256;
    matrix[5] = py * 256;
    _swix(DrawFile_Render, _INR(0,4), 0, file_data, file_size, matrix, clip_rect);
}
```

**2× magnification at position (px, py):**
```c
int32_t matrix[] = { 0x20000, 0, 0, 0x20000, px * 256, py * 256 };
_swix(DrawFile_Render, _INR(0,4), 0, file_data, file_size, matrix, 0);
```

---

### DrawFile_BBox — SWI &45541

Returns the bounding box that a Draw file would occupy after a given transformation.

**On entry:**
- R0 = 0 (all flag bits reserved)
- R1 = pointer to Draw file data
- R2 = size of Draw file data in bytes
- R3 = pointer to transformation matrix, or 0 for identity
- R4 = pointer to a 4-word output buffer to receive (x0, y0, x1, y1) in internal Draw units

**On exit:** R0–R4 preserved

**C usage:**
```c
int32_t bbox[4];   /* x0, y0, x1, y1 in internal Draw units */
_swix(DrawFile_BBox, _INR(0,4), 0, file_data, file_size, matrix, bbox);
/* Convert to OS units: divide each value by 256 */
```

---

### DrawFile_DeclareFonts — SWI &45542

Declares all fonts used in a Draw file to the printer driver by calling `PDriver_DeclareFont`
for each font found. Must be called once per Draw file at the font-declaration stage of printing.
After all files have been declared, make one final `PDriver_DeclareFont` call with R1 = 0 to
signal end of the font list (this is your responsibility — `DrawFile_DeclareFonts` does not make
the terminating call).

All fonts are declared as kerned (which includes the non-kerned case).

**On entry:**
- R0 = flags:
  - bit 0: download font data to printer (set for PostScript drivers that need embedded fonts)
- R1 = pointer to Draw file data
- R2 = size of Draw file data in bytes

**On exit:** R0–R2 preserved

---

## DrawFile Format

A Draw file (RISC OS filetype &AFF) is a fixed 40-byte file header followed by a sequence of
zero or more objects packed end-to-end. All values are little-endian 32-bit words unless stated
otherwise. All coordinates are in internal Draw units (multiply OS units by 256).

### File Header (40 bytes)

| Offset | Size | Field         | Description                                         |
|--------|------|---------------|-----------------------------------------------------|
| 0      | 4    | Magic         | ASCII `"Draw"` = &77617244 (little-endian)          |
| 4      | 4    | Major version | 201                                                 |
| 8      | 4    | Minor version | 0                                                   |
| 12     | 24   | Creator name  | Null-padded ASCII string (24 bytes)                 |
| 36     | 4    | Bbox x0       | Left edge of file bounding box (internal Draw units)|
| 40     | 4    | Bbox y0       | Bottom edge of file bounding box                    |
| 44     | 4    | Bbox x1       | Right edge of file bounding box                     |
| 48     | 4    | Bbox y1       | Top edge of file bounding box                       |

*(Note: the bounding box occupies bytes 28–44 of the 40-byte header, immediately after the
24-byte creator string. Total header = 4 + 4 + 4 + 24 + 16 = 52 bytes in some references;
verify against actual content when writing a parser.)*

Objects begin immediately after the file header.

---

### Common Object Header (24 bytes)

Every object begins with a 24-byte header:

| Offset | Size | Field   | Description                                          |
|--------|------|---------|------------------------------------------------------|
| 0      | 4    | Type    | Object type number (see table below)                 |
| 4      | 4    | Size    | Total object size in bytes, including this header    |
| 8      | 4    | Bbox x0 | Bounding box left (internal Draw units)              |
| 12     | 4    | Bbox y0 | Bounding box bottom (internal Draw units)            |
| 16     | 4    | Bbox x1 | Bounding box right (internal Draw units)             |
| 20     | 4    | Bbox y1 | Bounding box top (internal Draw units)               |

Object-specific data follows at offset 24. To advance to the next object, add the Size field
to the object's start address. Objects are word-aligned.

---

### Object Types

| Type | Name               | Description                                               |
|------|--------------------|-----------------------------------------------------------|
| 0    | Font table         | Maps local font numbers to font names; must appear first  |
| 1    | Text               | Single line of text at a given position and colour        |
| 2    | Path               | Vector path with fill/stroke style (uses Draw path data)  |
| 4    | Sprite             | RISC OS sprite image                                      |
| 5    | Group              | Container for a sequence of nested objects                |
| 6    | Tagged             | Any object preceded by a 4-byte application-defined tag   |
| 9    | Text area          | Multi-line formatted text block                           |
| 10   | Text column        | Column definition for a text area                         |
| 11   | Options            | File-level options (grid size, zoom factor, etc.)         |
| 12   | Transformed sprite | Sprite with an affine transformation matrix               |
| 13   | JPEG               | JPEG image (RISC OS 3.6+)                                 |

---

### Font Table Object (type 0)

Contains a mapping from 1-byte font numbers (1–255) to font name strings. Must appear before
any text objects that reference fonts.

After the 24-byte common header, the data is a sequence of:

```
<1-byte font number> <null-terminated font name string>
```

repeated until the end of the object. Font number 0 is the system font and is never listed.
The object is padded to a word boundary with zero bytes.

---

### Text Object (type 1)

After the common header:

| Offset | Size | Field        | Description                                         |
|--------|------|--------------|-----------------------------------------------------|
| 24     | 4    | Text colour  | Foreground colour as a palette entry (&BBGGRR00)    |
| 28     | 4    | Background   | Background hint for anti-aliasing (&BBGGRR00)       |
| 32     | 4    | Font style   | Font number (0 = system font) in byte 0             |
| 36     | 4    | Font size x  | x size in internal Draw units                       |
| 40     | 4    | Font size y  | y size in internal Draw units                       |
| 44     | 4    | Baseline x   | x coordinate of text baseline (internal Draw units) |
| 48     | 4    | Baseline y   | y coordinate of text baseline (internal Draw units) |
| 52     | var  | Text string  | Null-terminated string, padded to word boundary     |

---

### Path Object (type 2)

After the common header:

| Offset | Size | Field        | Description                                                   |
|--------|------|--------------|---------------------------------------------------------------|
| 24     | 4    | Fill colour  | Fill colour as palette entry (&BBGGRR00), or &FFFFFFFF = none |
| 28     | 4    | Outline colour | Stroke colour, or &FFFFFFFF = none                          |
| 32     | 4    | Outline width | Line thickness in internal Draw units (0 = hairline)         |
| 36     | 4    | Path style   | Packed word describing join, cap, and winding (see below)     |
| 40     | var  | Path data    | Draw path elements (see `draw.md` path element types)         |

**Path style word:**

| Bits  | Field            | Values                                          |
|-------|------------------|-------------------------------------------------|
| 0–2   | Join style       | 0=mitred, 1=round, 2=bevelled                   |
| 3–5   | Start cap        | 0=butt, 1=round, 2=projecting square, 3=triangular |
| 6–8   | End cap          | Same as start cap                               |
| 9     | Winding rule     | 0=non-zero, 1=even-odd                          |
| 10    | Dash pattern     | 0=solid, 1=dashed (dash data follows path data) |
| 11    | Reserved         | 0                                               |
| 16–23 | Triangle cap width  | In 1/16ths of line width                     |
| 24–31 | Triangle cap length | In 1/16ths of line width                     |

If bit 10 (dash) is set, a dash pattern block follows immediately after the path data (within the
object). The dash pattern format is identical to that used by `Draw_Stroke` (see `draw.md`).

---

### Sprite Object (type 4)

After the common header, object data is a standard RISC OS sprite (beginning with the sprite
area header, or just the sprite itself — check the size field). No additional transformation
is applied; the sprite is rendered at its natural scale mapped to the bounding box.

---

### Group Object (type 5)

After the common header, contains a sequence of complete DrawFile objects (each with its own
24-byte header), packed end-to-end until the end of the group as indicated by the Size field.
Groups may be nested.

---

### Tagged Object (type 6)

A 4-byte application-defined tag word at offset 24, followed by a complete DrawFile object
(with its own 24-byte header) starting at offset 28. The renderer ignores the tag and renders
the embedded object normally.

---

### Options Object (type 11)

After the common header, contains file-level display settings (grid unit, grid spacing, grid
locked state, auto-adjust, toolbox visibility, entry mode, undo buffer size, etc.). The renderer
ignores this object; it is used by the !Draw application only.

---

## DrawFile Error Numbers

| Number  | Symbolic name         | Meaning                                         |
|---------|-----------------------|-------------------------------------------------|
| &20C00  | DrawFileNotDraw       | File does not begin with `"Draw"` magic number  |
| &20C01  | DrawFileVersion       | File major version is not supported             |
| &20C02  | DrawFileFontTab       | Font table object is malformed                  |
| &20C03  | DrawFileBadFontNo     | Font number in an object is not in font table   |
| &20C04  | DrawFileBadMode       | Bad rendering mode requested                    |
| &20C05  | DrawFileBadFile       | File data is corrupt or truncated               |
| &20C06  | DrawFileBadGroup      | Group object is malformed                       |
| &20C07  | DrawFileBadTag        | Tagged object is malformed                      |
| &20C08  | DrawFileSyntax        | General syntax error in file structure          |
| &20C09  | DrawFileFontNo        | Font number out of valid range                  |
| &20C0A  | DrawFileAreaVer       | Text area version number mismatch               |
| &20C0B  | DrawFileNoAreaVer     | Text area has no version number                 |

---

## Typical Usage Patterns

### Render a Draw file during a Wimp redraw loop

```c
#include "swis.h"
#include <stdint.h>

/* ox, oy = window work-area origin in screen OS units (from Wimp_RedrawWindow) */
/* clip   = pointer to the 4-word clipping rectangle returned by Wimp_GetRectangle */
void redraw_draw_file(void *file_data, int file_size,
                      int ox, int oy, int *clip)
{
    int32_t matrix[6];
    matrix[0] = 0x10000; matrix[1] = 0;
    matrix[2] = 0;       matrix[3] = 0x10000;
    matrix[4] = ox * 256;
    matrix[5] = oy * 256;
    _swix(DrawFile_Render, _INR(0,4), 0, file_data, file_size, matrix, clip);
}
```

### Get the bounding box of a Draw file in OS units

```c
int32_t bbox_du[4];  /* internal Draw units: x0, y0, x1, y1 */
int     bbox_os[4];  /* OS units */
int     i;

_swix(DrawFile_BBox, _INR(0,4), 0, file_data, file_size, 0, bbox_du);
for (i = 0; i < 4; i++)
    bbox_os[i] = bbox_du[i] / 256;
```

### Declare fonts before printing a Draw file

```c
/* Inside the print-job setup, before PDriver_DrawPage loop */
_swix(DrawFile_DeclareFonts, _INR(0,2), 0, file_data, file_size);
/* Signal end of font list */
_swix(PDriver_DeclareFont, _INR(0,2), 0, 0, 0);
```

### Load a Draw file from disc and render it

```c
#include "swis.h"
#include <stdlib.h>
#include <stdint.h>

int load_and_render(const char *filename, int px, int py)
{
    int    file_size;
    void  *file_data;
    int32_t matrix[6];

    /* Find file size */
    if (_swix(OS_File, _INR(0,1)|_OUT(4), 17, filename, &file_size))
        return 0;

    file_data = malloc(file_size);
    if (!file_data)
        return 0;

    /* Load file */
    _swix(OS_File, _INR(0,3), 16, filename, file_data, 0);

    matrix[0] = 0x10000; matrix[1] = 0;
    matrix[2] = 0;       matrix[3] = 0x10000;
    matrix[4] = px * 256;
    matrix[5] = py * 256;
    _swix(DrawFile_Render, _INR(0,4), 0, file_data, file_size, matrix, 0);

    free(file_data);
    return 1;
}
```

---

## Notes

- DrawFile objects must be word-aligned; pad all variable-length data (strings, path data) to a
  4-byte boundary.
- Bounding box values in the object headers must be accurate; `DrawFile_Render` uses them for
  clipping — objects whose bounding box does not intersect the clipping rectangle are skipped.
- The renderer handles unknown object types by calling a service call
  (`Service_DrawObjectRender` / `Service_DrawObjectDeclareFonts`), allowing modules to extend
  the object type set.
- `DrawFile_Render` is safe to call during a Wimp redraw loop; it does not call
  `Wimp_ReportError`. Report errors after the redraw loop has finished.
- For printing, the transformation matrix should map from Draw file coordinates to the printer's
  page coordinates using the scale and origin provided by `PDriver_DrawPage`.
