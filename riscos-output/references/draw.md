# Draw Module Reference

## Overview

The RISC OS Draw module provides a PostScript-like path drawing API. It handles Bezier curves,
line stroking with caps and joins, area filling, dash patterns, and coordinate transformations.

All output uses the current graphics colour set via `VDU 18`, `OS_SetColour`, or ColourTrans.
Coordinates are in *user units* transformed via a matrix into OS units.

For rendering `.aff` Draw files, see `drawfile.md`.

---

## Coordinate Systems

Three coordinate systems are used:

| System              | Size      | Description                                               |
|---------------------|-----------|-----------------------------------------------------------|
| OS units            | 1 unit    | 1/180th inch. Standard VDU/OS_Plot units.                 |
| Internal Draw units | 1/256 unit| Each OS unit divided into 256 parts. Top 24 bits = OS unit integer; bottom 8 bits = fraction. |
| User units          | variable  | Application-defined. Converted to internal Draw units via the transformation matrix. |
| Transform units     | fixed pt  | Used for matrix scale/rotation entries. 32-bit: top 16 bits integer, bottom 16 bits fraction. Scale 1.0 = &00010000. |

**Conversion examples:**
- 64 OS units = `&4000` internal Draw units
- Scale factor 1.0 = `&00010000` transform units
- Scale factor 2.0 = `&00020000` transform units

Avoid scale factors greater than 8× (&80000 transform units); larger values amplify rounding errors
in the path arithmetic.

---

## Draw SWIs

### Draw_ProcessPath — SWI &40700

The master SWI; all other Draw SWIs resolve to calls to this.

**On entry:**
- R0 = pointer to input path buffer
- R1 = fill style (see Fill Style section)
- R2 = pointer to transformation matrix, or 0 for identity
- R3 = flatness, or 0 for default
- R4 = line thickness (user units), or 0 for hairline
- R5 = pointer to cap/join specification, or 0
- R6 = pointer to dash pattern, or 0 for solid
- R7 = output destination (see R7 values below)

**On exit:**
- R0 = result (depends on R7; see below)

**R7 values:**

| R7 value          | Effect                                                                           |
|-------------------|----------------------------------------------------------------------------------|
| 0                 | Output back to input path buffer (only valid if size does not change, e.g. transform-only) |
| 1                 | Fill path to VDU (normal)                                                        |
| 2                 | Fill path to VDU, subpath by subpath (uses less RMA)                             |
| 3                 | Return required output buffer size in R0 (bytes needed)                          |
| &80000000+ptr     | Write bounding box to buffer at ptr: four words — low x, low y, high x, high y (in transformed internal Draw units) |
| ptr               | Output to path buffer at ptr; R0 returns pointer to new element-0 terminator     |

**Processing order** (controlled by fill style bits 27–30 and R6):
1. Close open subpaths (bit 27)
2. Flatten Bezier curves (bit 28, uses R3)
3. Apply dash pattern (if R6 ≠ 0; requires flattened path)
4. Thicken path / add caps and joins (bit 29, uses R4 and R5; requires flattened path)
5. Re-flatten round caps/joins (bit 30, uses R3)
6. Transform to screen coordinates (R2)
7. Output as specified by R7

---

### Draw_Fill — SWI &40702

Fills the interior of a closed path and renders to the VDU. Equivalent to PostScript `fill`.

**On entry:**
- R0 = pointer to input path
- R1 = fill style, or 0 for default (&30)
- R2 = pointer to transformation matrix, or 0
- R3 = flatness, or 0

**On exit:**
- R0 corrupted; R1–R3 preserved

Actions performed: close open subpaths → flatten → transform → fill to VDU.

**C usage:**
```c
#include "swis.h"
_swix(Draw_Fill, _INR(0,3), path_ptr, 0, matrix_ptr, 0);
```

---

### Draw_Stroke — SWI &40704

Strokes (outlines) a path with a given thickness and renders to the VDU. Equivalent to PostScript
`stroke`.

**On entry:**
- R0 = pointer to input path
- R1 = fill style, or 0 for default (see note)
- R2 = pointer to transformation matrix, or 0
- R3 = flatness, or 0
- R4 = line thickness (user units), or 0 for hairline
- R5 = pointer to cap/join specification
- R6 = pointer to dash pattern, or 0

**On exit:**
- R0 corrupted; R1–R6 preserved

Default fill style: if R4 ≠ 0 then &30; if R4 = 0 then &18.

If bit 31 of fill style is set, the entire path is plotted in one pass (no double-plotting on
overlapping subpaths, but uses more RMA). Most printer drivers ignore bit 31.

**C usage:**
```c
_swix(Draw_Stroke, _INR(0,6), path_ptr, 0, matrix_ptr, 0, thickness, caps_ptr, 0);
```

---

### Draw_StrokePath — SWI &40706

Like `Draw_Stroke` but writes the result to an output path buffer instead of the VDU. Useful for
pre-processing a path that will be stroked multiple times.

**On entry:**
- R0 = pointer to input path
- R1 = pointer to output path buffer, or 0 to calculate required size
- R2 = pointer to transformation matrix, or 0
- R3 = flatness, or 0
- R4 = line thickness, or 0
- R5 = pointer to cap/join specification
- R6 = pointer to dash pattern, or 0

**On exit:**
- If R1 = 0: R0 = required buffer size in bytes
- Otherwise: R0 = pointer to new element-0 terminator in output buffer

---

### Draw_FlattenPath — SWI &40708

Converts all Bezier curve elements in a path into sequences of line elements, to within the
specified flatness tolerance. Useful when you intend to stroke the same path many times.

**On entry:**
- R0 = pointer to input path
- R1 = pointer to output path buffer, or 0 to calculate required size
- R2 = flatness, or 0 for default

**On exit:**
- If R1 = 0: R0 = required buffer size
- Otherwise: R0 = pointer to new element-0 terminator

---

### Draw_TransformPath — SWI &4070A

Applies a transformation matrix to all coordinate values in a path, writing to an output buffer.
Useful when you want to transform before dashing/thickening to avoid magnifying rounding errors.

**On entry:**
- R0 = pointer to input path
- R1 = pointer to output path, or 0 to overwrite input path in place
- R2 = pointer to transformation matrix, or 0 for identity
- R3 = 0 (reserved)

**On exit:**
- R0 = pointer to new element-0 terminator (or preserved if R1 = 0 on entry and the input was overwritten)

---

## Data Structures

### Path Buffer

A path is a contiguous sequence of variable-length *path elements* in memory. Each element begins
with a word whose low byte is the element type (0–8). The remaining three bytes of the first word
are reserved (written as zero by the Draw module).

**Output buffer sentinel:** Before passing a buffer as an output path, place a type-0 element at
the buffer start with the buffer's total byte capacity in the second word:

```c
buffer[0] = 0;          /* element type 0 */
buffer[1] = buf_bytes;  /* available space */
```

After a Draw call, R0 points to the new type-0 terminator, allowing you to append further elements.

#### Path Element Types

| Type | Size (words) | Parameters       | Description                                               |
|------|-------------|------------------|-----------------------------------------------------------|
| 0    | 2           | n (space check)  | End of path. n = remaining buffer space in bytes.        |
| 1    | 2           | ptr              | Continuation: ptr is address of next element (linked path)|
| 2    | 3           | x, y             | Move to (x,y); starts a new subpath; affects winding      |
| 3    | 3           | x, y             | Move to (x,y); internal use (doesn't affect winding)      |
| 4    | 1           | —                | Close subpath with a gap (no line drawn to start point)   |
| 5    | 1           | —                | Close subpath with a line to the start point              |
| 6    | 7           | x1,y1,x2,y2,x3,y3 | Bezier curve to (x3,y3); (x1,y1) and (x2,y2) are control points |
| 7    | 3           | x, y             | Gap to (x,y) without starting a new subpath (dash use)   |
| 8    | 3           | x, y             | Line to (x,y)                                             |

**Rules:**
- Elements 2 or 3 must precede any line/curve elements (start a subpath).
- Elements 6, 7, 8 require an active (open) subpath.
- Elements 4 or 5 close the subpath; a new subpath requires another element 2 or 3.
- Dashing, thickening, and filling require the path to be flattened (no type-6 elements).

**Example path — filled rectangle (user units):**
```c
int32_t path[] = {
    2, 0, 0,          /* move to (0,0) */
    8, 1000, 0,       /* line to (1000,0) */
    8, 1000, 500,     /* line to (1000,500) */
    8, 0, 500,        /* line to (0,500) */
    5,                /* close with line */
    0, 0              /* end of path */
};
```

---

### Transformation Matrix

A 24-byte (6-word) block:

| Offset | Name | Type           | Purpose                                      |
|--------|------|----------------|----------------------------------------------|
| 0      | a    | Transform units | x-scale / cos(θ) for rotation                |
| 4      | b    | Transform units | sin(θ) for rotation                          |
| 8      | c    | Transform units | −sin(θ) for rotation                         |
| 12     | d    | Transform units | y-scale / cos(θ) for rotation                |
| 16     | e    | Internal Draw units | x-translation                           |
| 20     | f    | Internal Draw units | y-translation                           |

Transform formula: `x' = a·x + c·y + e`, `y' = b·x + d·y + f`

**Scale factor 1.0 = &00010000** in transform units (16-bit fraction).

**Translation:** multiply OS unit offset by 256 to get internal Draw units.

**Identity matrix (1:1, no translation):**
```c
int32_t identity[] = { 0x10000, 0, 0, 0x10000, 0, 0 };
```

**1:1 rendering at screen position (px, py) in OS units:**
```c
int32_t matrix[] = { 0x10000, 0, 0, 0x10000, px * 256, py * 256 };
```

**2× magnification at origin:**
```c
int32_t matrix[] = { 0x20000, 0, 0, 0x20000, 0, 0 };
```

---

### Fill Style

A 32-bit word controlling which pixels around a path boundary are plotted:

| Bits  | Meaning                                                   |
|-------|-----------------------------------------------------------|
| 0–1   | Winding rule: 0=non-zero, 1=negative, 2=even-odd, 3=positive |
| 2     | 1 = plot non-boundary exterior pixels                     |
| 3     | 1 = plot boundary exterior pixels                         |
| 4     | 1 = plot boundary interior pixels                         |
| 5     | 1 = plot non-boundary interior pixels                     |
| 6–26  | Reserved, must be zero                                    |
| 27    | 1 = close open subpaths before processing                 |
| 28    | 1 = flatten path                                          |
| 29    | 1 = thicken path (stroke)                                 |
| 30    | 1 = re-flatten after thickening                           |
| 31    | 1 = floating point output (not implemented)               |

**Common presets:**

| Value | Meaning                                              |
|-------|------------------------------------------------------|
| &30   | Default fill: bits 4+5 set — fill all interior pixels, non-zero winding |
| &18   | Default hairline stroke: bits 3+4 set — boundary pixels only |
| 0     | Special: use default for the SWI being called        |

**Winding rules:**
- **Non-zero (0):** Counts signed crossings of a ray. Zero count = exterior. Compatible with printer drivers.
- **Even-odd (2):** Counts all crossings. Odd = interior. Compatible with printer drivers.
- **Positive (3):** Interior if more anti-clockwise crossings than clockwise.
- **Negative (1):** Interior if more clockwise crossings than anti-clockwise.
- Positive/negative rules are not supported by printer drivers.

---

### Cap and Join Specification

A 16-byte (4-word) block pointed to by R5:

| Offset | Byte | Field                   | Values                                              |
|--------|------|-------------------------|-----------------------------------------------------|
| 0      | 0    | Join style              | 0=mitred, 1=round, 2=bevelled                       |
| 0      | 1    | Leading (start) cap     | 0=butt, 1=round, 2=projecting square, 3=triangular  |
| 0      | 2    | Trailing (end) cap      | 0=butt, 1=round, 2=projecting square, 3=triangular  |
| 0      | 3    | Reserved                | Must be 0                                           |
| 4      | 0–1  | Mitre limit (fraction)  | Used when join style = 0 (mitred)                   |
| 4      | 2–3  | Mitre limit (integer)   | Signed, positive; expressed in line-width multiples |
| 8      | 0–1  | Leading cap width       | In units of 1/256 of line width (&0100 = 1× width)  |
| 8      | 2–3  | Leading cap length      | Same units                                          |
| 12     | 0–1  | Trailing cap width      | Same units                                          |
| 12     | 2–3  | Trailing cap length     | Same units                                          |

Width/length fields at offsets 8 and 12 are only used when the corresponding cap is triangular
(style 3). The mitre limit at offset 4 is only used when join style is mitred (0).

**Typical square-ended, mitred joins:**
```c
int32_t caps[] = {
    0x00000000,   /* butt caps, mitred joins */
    0x000A0000,   /* mitre limit = 10.0 */
    0x00000000,   /* leading cap size (unused) */
    0x00000000    /* trailing cap size (unused) */
};
```

**Printer driver limitations:** Most printer drivers do not support:
- Differing leading and trailing caps
- Triangular caps
- Round joins at large scales

---

### Dash Pattern

A variable-length block pointed to by R6:

| Offset    | Field                          | Description                              |
|-----------|--------------------------------|------------------------------------------|
| 0         | Start offset                   | Distance into pattern at path start (user units) |
| 4         | n (number of elements)         | Number of on/off distance values          |
| 8 + 4×i   | Element i (i = 0 … n−1)        | Alternating on/off distances (user units) |

The pattern starts "on" (drawing). If n is odd, the on/off sense inverts on each pattern cycle.
Pass 0 in R6 for solid (undashed) lines.

**Example — 10 on, 5 off, starting at beginning:**
```c
int32_t dash[] = { 0, 2, 10, 5 };
```

---

### Line Thickness

Specified in user units (R4 in `Draw_Stroke` / `Draw_ProcessPath`):

- **0**: Hairline — one pixel wide regardless of transformation scale.
- **n**: Line drawn n/2 user units on each side of the path centreline (total width = n).

When R4 is non-zero, a cap/join specification (R5) must be provided.

---

## Typical Usage Patterns

### Draw a filled shape

```c
#include "swis.h"
#include <stdint.h>

/* Path: triangle with vertices at user coordinates */
static int32_t tri_path[] = {
    2, 0, 0,          /* move to (0,0) */
    8, 500, 1000,     /* line to (500,1000) */
    8, 1000, 0,       /* line to (1000,0) */
    5,                /* close with line */
    0, 0              /* end */
};

/* Identity matrix: 1 OS unit = 1 user unit, origin at (100,100) OS units */
static int32_t matrix[] = { 0x10000, 0, 0, 0x10000, 100*256, 100*256 };

void draw_triangle(void)
{
    _swix(Draw_Fill, _INR(0,3), tri_path, 0, matrix, 0);
}
```

### Stroke a path with round caps

```c
static int32_t line_path[] = {
    2, 0, 0,
    8, 2000, 0,
    0, 0
};

static int32_t caps[] = {
    0x00000101,   /* round joins (0), round leading cap (1), round trailing cap (1) */
    0x000A0000,   /* mitre limit = 10 (unused for round joins) */
    0x00000000,
    0x00000000
};

static int32_t matrix[] = { 0x10000, 0, 0, 0x10000, 50*256, 200*256 };

void draw_line(void)
{
    /* thickness = 20 user units */
    _swix(Draw_Stroke, _INR(0,6), line_path, 0, matrix, 0, 20, caps, 0);
}
```

### Calculate output buffer size for a stroked path

```c
int32_t required_bytes;
_swix(Draw_StrokePath, _INR(0,6)|_OUT(0),
      path_ptr, 0, matrix, 0, thickness, caps, dash,
      &required_bytes);
/* allocate required_bytes, then call again with real buffer */
```

---

## Printer Driver Considerations

When printing, a printer driver intercepts `Draw_Fill` and `Draw_Stroke`. The following features
are not supported by most printer drivers and must be handled before calling:

- `AND`/`OR`/`XOR` colour operations (only overwrite is safe)
- Fill style bits 2–5 (boundary/exterior pixel control) — only &30 and &18 work reliably
- Positive and negative winding rules (use non-zero or even-odd)
- Differing leading and trailing line caps
- Triangular caps
- `Draw_ProcessPath` with R7 = 1 or 2

To work around limitations: use `Draw_ProcessPath` with R7 = output-buffer to pre-process the
path into a simple filled shape, then use `Draw_Fill` to render that shape.
