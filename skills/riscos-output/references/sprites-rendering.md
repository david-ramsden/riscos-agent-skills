# RISC OS Sprites: Rendering

SWIs for plotting sprites and masks on screen — at fixed positions, scaled, and with arbitrary linear transformations.

---

## Basic Plotting

### OS_SpriteOp 28 — Put Sprite

Plots a sprite at the current graphics cursor position. The bottom-left corner of the sprite is placed at the cursor.

**Entry:**
- R0 = 28 + area flags
- R1 = sprite area pointer
- R2 = sprite pointer or name
- R5 = plot action (see [Areas and Format](sprites-areas-and-format.md))

**Exit:** R0–R2, R5 preserved

---

### OS_SpriteOp 34 — Put Sprite at User Coordinates

Plots a sprite at an explicit OS-unit coordinate. No mode compatibility check is performed — pixels are copied directly.

**Entry:**
- R0 = 34 + area flags
- R1 = sprite area pointer
- R2 = sprite pointer or name
- R3 = x coordinate (OS units)
- R4 = y coordinate (OS units)
- R5 = plot action

**Exit:** R0–R5 preserved

---

### OS_SpriteOp 48 — Plot Sprite Mask

Plots only the mask of a sprite at the current graphics cursor. Mask bits that are 1 (solid) are drawn in the current background colour; 0 bits are not plotted. If the sprite has no mask, a solid rectangle is drawn.

**Entry:**
- R0 = 48 + area flags
- R1 = sprite area pointer
- R2 = sprite pointer or name

**Exit:** R0–R2 preserved

---

### OS_SpriteOp 49 — Plot Mask at User Coordinates

As SpriteOp 48 but at an explicit coordinate.

**Entry:**
- R0 = 49 + area flags; R1/R2 = area/sprite; R3 = x (OS units); R4 = y (OS units)

**Exit:** R0–R4 preserved

---

## Scaled Plotting (requires SpriteExtend)

### OS_SpriteOp 52 — Put Sprite Scaled

Plots a sprite with optional scaling and pixel colour translation. The most commonly used rendering call for mode-independent sprite display.

**Entry:**
- R0 = 52 + area flags
- R1 = sprite area pointer
- R2 = sprite pointer or name
- R3 = x coordinate (OS units)
- R4 = y coordinate (OS units)
- R5 = plot action (see below for extended flags)
- R6 = pointer to scale factors block (0 = no scaling, plot at 1:1)
- R7 = pointer to pixel translation table (0 = no translation)

**Exit:** R0–R7 preserved

**R5 extended flags for SpriteOp 52:**

| Bit | Meaning |
|-----|---------|
| 0–3 | GCOL action (0=store, 1=OR, 2=AND, 3=XOR, 4=invert, 5=leave, 6=ANDNOT, 7=ORNOT) |
| 3 | Set to use mask |
| 4 | Palette entries in translation table (not colour numbers) |
| 5 | Wide translation table format |

**Notes:**
- Pass a pixel translation table built by `ColourTrans_SelectTable` (SWI &40740) or `ColourTrans_GenerateTable` (SWI &40763) to ensure correct colours in any screen mode.
- The ECF fill pattern is not scaled with the sprite.
- Requires the SpriteExtend module.

---

### OS_SpriteOp 50 — Plot Mask Scaled

Plots the sprite's mask scaled, in the current background colour.

**Entry:**
- R0 = 50 + area flags
- R1/R2 = area/sprite
- R3 = x (OS units); R4 = y (OS units)
- R6 = pointer to scale factors block

**Exit:** R0–R6 preserved. Requires SpriteExtend.

---

### OS_SpriteOp 53 — Put Sprite Grey Scaled

Anti-aliased scaled rendering. The sprite must be 4bpp with a linear grey-scale palette. Slower than SpriteOp 52.

**Entry:**
- R0 = 53 + area flags
- R1/R2 = area/sprite
- R3 = x; R4 = y; R5 = 0; R6 = scale factors; R7 = pixel translation table

**Exit:** R0–R7 preserved. Requires SpriteExtend.

---

### OS_SpriteOp 51 — Paint Character Scaled

Renders a single character glyph scaled, using the current foreground colour and GCOL action.

**Entry:**
- R0 = 51
- R1 = character code (ASCII)
- R3 = x (OS units); R4 = y (OS units); R6 = scale factors

**Exit:** R0, R1, R3, R4, R6 preserved. Requires SpriteExtend.

---

## Transformed Plotting (requires SpriteExtend)

SpriteOp 55 and 56 apply an arbitrary 2×2 linear transformation (rotation, shear, arbitrary scale) to the sprite. Coordinate units are **1/256 OS units** (fixed-point). Not available in RISC OS 2.

### OS_SpriteOp 56 — Put Sprite Transformed

**Entry:**
- R0 = 56 + area flags
- R1 = sprite area pointer
- R2 = sprite pointer or name
- R3 = flags:
  - bit 0 clear: R6 is a 2×2 matrix pointer
  - bit 0 set: R6 is a pointer to four destination coordinate pairs
  - bit 1 set: R4 is a pointer to a source rectangle
- R4 = source rectangle pointer (if R3 bit 1 set) or 0 (use whole sprite)
- R5 = plot action; add 8 to apply mask
- R6 = transformation matrix pointer or destination coordinates pointer
- R7 = pixel translation table pointer (0 = none)

**Exit:** All registers preserved

**Transformation matrix format (at R6, R3 bit 0 clear):**

| Offset | Content |
|--------|---------|
| 0 | M00 — x scale (16.16 fixed-point) |
| 4 | M01 — x shear |
| 8 | M10 — y shear |
| 12 | M11 — y scale |
| 16 | x translation (1/256 OS units) |
| 20 | y translation (1/256 OS units) |

**Destination coordinates format (at R6, R3 bit 0 set):** Four coordinate pairs `(x0,y0), (x1,y1), (x2,y2), (x3,y3)` (bottom-left, bottom-right, top-right, top-left) in 1/256 OS units.

---

### OS_SpriteOp 55 — Plot Mask Transformed

As SpriteOp 56 but plots only the mask in the current background colour.

**Entry/Exit:** Identical to SpriteOp 56. Requires SpriteExtend.

---

## Hardware Pointer

### OS_SpriteOp 36 — Set Pointer Shape

Programs the hardware mouse pointer from a sprite.

**Entry:**
- R0 = 36 + area flags
- R1 = sprite area pointer
- R2 = sprite pointer or name
- R3 = control flags:
  - bits 0–3: pointer shape number (1–4)
  - bit 4 set: load pixel data
  - bit 5 set: load palette data
  - bit 6 set: set active shape to shape number in bits 0–3
- R4 = x offset of pointer hotspot (pixels from left)
- R5 = y offset of pointer hotspot (pixels from top)
- R6 = pointer to scale factors block (0 = 1:1)
- R7 = pointer to pixel translation table (0 = none)

**Exit:** R0–R3 preserved

**Notes:**
- The hardware pointer supports shapes 1–4; only colours 0, 1, and 3 are usable in high-resolution modes.
- The pointer sprite should be small (typically 32×32 or smaller).

---

## Pixel Translation Tables

When plotting sprites across different screen modes or palettes, use `ColourTrans_SelectTable` or `ColourTrans_GenerateTable` to build a translation table and pass it in R7. The table maps source pixel values to destination pixel values. Pass R4=0 on first call to get the required buffer size, then allocate and call again.

```asm
; Build a translation table for plotting a sprite
MOV  r0, sprite_area          ; source mode/sprite area
MOV  r1, sprite_ptr           ; source palette
MOV  r2, #-1                  ; destination: current mode
MOV  r3, #-1                  ; destination: current palette
MOV  r4, #0                   ; query size
MOV  r5, #0                   ; flags
SWI  ColourTrans_GenerateTable ; returns required size in R4
; allocate R4 bytes, put pointer in r4
SWI  ColourTrans_GenerateTable ; fill buffer
; now pass buffer pointer in R7 to SpriteOp 52
```

---

## VDU Plotting

Sprites can also be plotted using VDU sequences after selecting a sprite with SpriteOp 24:

| Sequence | Action |
|----------|--------|
| `VDU 23,27,0,n\|` | Select sprite n for VDU output |
| `VDU 25,232,x;y;` | Plot selected sprite (overwrite) |
| `VDU 25,233,x;y;` | Plot selected sprite (OR) |
| `VDU 25,234,x;y;` | Plot selected sprite (AND) |
| `VDU 25,235,x;y;` | Plot selected sprite (XOR) |
| `VDU 25,236,x;y;` | Plot selected sprite (invert) |
| `VDU 25,237–239,x;y;` | Further GCOL actions |

### OS_SpriteOp 24 — Select Sprite

Selects a sprite for subsequent VDU 25,232–239 plot commands.

**Entry:**
- R0 = 24 + area flags
- R1 = sprite area pointer
- R2 = sprite pointer or name

**Exit:**
- R0/R1 preserved
- R2 = pointer to sprite control block (user area only; valid until next area-rearranging operation)
