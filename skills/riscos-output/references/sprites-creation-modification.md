# RISC OS Sprites: Creation and Modification

SWIs for creating sprites, capturing screen content into sprites, manipulating pixels, masks, rows and columns, and managing palettes.

---

## Creating Sprites

### OS_SpriteOp 15 — Create Sprite

Creates a blank sprite of a given size within a sprite area. Pixel data is uninitialised.

**Entry:**
- R0 = 15 + area flags (256 or 512; see [Areas and Format](sprites-areas-and-format.md))
- R1 = sprite area pointer
- R2 = pointer to sprite name string
- R3 = palette flag: 0 = no palette; 1 = include palette
- R4 = width in pixels
- R5 = height in pixels
- R6 = mode: mode number, sprite mode word, or mode selector pointer

**Exit:** R0–R6 preserved

**Notes:**
- Accepts the full range of mode specifiers (mode number, sprite mode word, selector) since RISC OS 3.5.
- The sprite area must have sufficient free space.

---

### OS_SpriteOp 14 — Get Sprite (screen capture)

Captures the current graphics window defined by the old and new graphics cursor positions into a named sprite.

**Entry:**
- R0 = 14 + area flags
- R1 = sprite area pointer
- R2 = pointer to sprite name string
- R3 = palette flag: 0 = no palette; 1 = include palette

**Exit:**
- R0/R1 preserved
- R2 = pointer to created sprite control block (user area only)
- R3 preserved

**Notes:** Pixels outside the captured area are filled with the background colour.

---

### OS_SpriteOp 16 — Get Sprite from User Coordinates

Captures a screen rectangle specified by explicit OS-unit coordinates into a named sprite.

**Entry:**
- R0 = 16 + area flags
- R1 = sprite area pointer
- R2 = pointer to sprite name string
- R3 = palette flag
- R4 = left x (OS units, inclusive)
- R5 = bottom y (OS units, inclusive)
- R6 = right x (OS units, inclusive)
- R7 = top y (OS units, inclusive)

**Exit:**
- R0/R1 preserved
- R2 = pointer to sprite control block (user area only)
- R3–R7 preserved

---

### OS_SpriteOp 2 — Screen Save

Saves the current graphics window to a sprite file containing a single sprite named `screendump`.

**Entry:**
- R0 = 2
- R2 = pointer to filename string
- R3 = palette flag

**Exit:** R0/R2/R3 preserved

---

## Pixel Modification

### OS_SpriteOp 41 — Read Pixel Colour

**Entry:**
- R0 = 41 + area flags
- R1 = sprite area pointer
- R2 = sprite pointer or name
- R3 = x coordinate (pixels from left, 0-based)
- R4 = y coordinate (pixels from bottom, 0-based)

**Exit:** R0–R4 preserved; R5 = colour number; R6 = tint value

---

### OS_SpriteOp 42 — Write Pixel Colour

**Entry:**
- R0 = 42 + area flags
- R1 = sprite area pointer
- R2 = sprite pointer or name
- R3 = x coordinate (pixels)
- R4 = y coordinate (pixels)
- R5 = colour number
- R6 = tint value

**Exit:** R0–R6 preserved

---

### OS_SpriteOp 43 — Read Pixel Mask

**Entry:**
- R0 = 43 + area flags
- R1 = sprite area pointer
- R2 = sprite pointer or name
- R3 = x coordinate
- R4 = y coordinate

**Exit:** R0–R4 preserved; R5 = 0 (transparent) or 1 (solid)

---

### OS_SpriteOp 44 — Write Pixel Mask

**Entry:**
- R0 = 44 + area flags
- R1 = sprite area pointer
- R2 = sprite pointer or name
- R3 = x coordinate
- R4 = y coordinate
- R5 = 0 (transparent) or 1 (solid)

**Exit:** R0–R5 preserved

---

## Mask Management

### OS_SpriteOp 29 — Create Mask

Adds a transparency mask to a sprite. All mask pixels are initialised to solid (1).

**Entry:**
- R0 = 29 + area flags
- R1 = sprite area pointer
- R2 = sprite pointer or name

**Exit:** R0–R2 preserved

---

### OS_SpriteOp 30 — Remove Mask

Removes the transparency mask from a sprite, freeing its memory.

**Entry:**
- R0 = 30 + area flags
- R1 = sprite area pointer
- R2 = sprite pointer or name

**Exit:** R0–R2 preserved

---

## Row and Column Operations

Coordinate system: row 0 is the **bottom** row; column 0 is the **left** column.

### OS_SpriteOp 31 — Insert Row

Inserts a blank row at the given position. Existing rows at or above that position move up. New pixels are colour 0 (or transparent if the sprite has a mask).

**Entry:**
- R0 = 31 + area flags
- R1 = sprite area pointer
- R2 = sprite pointer or name
- R3 = row number (0 = bottom)

**Exit:** R0–R3 preserved

---

### OS_SpriteOp 32 — Delete Row

**Entry:**
- R0 = 32 + area flags; R1/R2 = area/sprite; R3 = row number

**Exit:** R0–R3 preserved. Error if R3 ≥ sprite height.

---

### OS_SpriteOp 45 — Insert Column

**Entry:**
- R0 = 45 + area flags; R1/R2 = area/sprite; R3 = column number (0 = left)

**Exit:** R0–R3 preserved

---

### OS_SpriteOp 46 — Delete Column

**Entry:**
- R0 = 46 + area flags; R1/R2 = area/sprite; R3 = column number

**Exit:** R0–R3 preserved

---

### OS_SpriteOp 57 — Insert/Delete Multiple Rows (SpriteExtend)

**Entry:**
- R0 = 57 + area flags
- R1/R2 = area/sprite
- R3 = starting row
- R4 = count: positive = insert that many rows; negative = delete that many rows

**Exit:** R0–R4 preserved. Requires SpriteExtend; not available in RISC OS 2.

---

### OS_SpriteOp 58 — Insert/Delete Multiple Columns (SpriteExtend)

**Entry:**
- R0 = 58 + area flags
- R1/R2 = area/sprite
- R3 = starting column
- R4 = count: positive = insert; negative = delete

**Exit:** R0–R4 preserved. Requires SpriteExtend; not available in RISC OS 2.

---

## Flipping

### OS_SpriteOp 33 — Flip about X Axis (vertical mirror)

Reverses the order of rows (top becomes bottom). The sprite appears upside-down.

**Entry:** R0 = 33 + area flags; R1/R2 = area/sprite

**Exit:** R0–R2 preserved

---

### OS_SpriteOp 47 — Flip about Y Axis (horizontal mirror)

Reverses the order of pixels within each row (left becomes right). The sprite appears back-to-front.

**Entry:** R0 = 47 + area flags; R1/R2 = area/sprite

**Exit:** R0–R2 preserved

---

## Palette Management

### OS_SpriteOp 37 — Create or Remove Palette

**Entry:**
- R0 = 37 + area flags
- R1 = sprite area pointer
- R2 = sprite pointer or name
- R3 = sub-reason:
  - −1 = read palette information
  - 0 = remove palette
  - 1 = create palette with default entries
  - Any positive value: size of palette to create
  - Bit 31 set: create with 256 entries (8bpp only)

**Exit (R3 was −1 on entry):**
- R3 = palette size in bytes
- R4 = pointer to palette data
- R5 = sprite mode word

**Notes:**
- Modifying palette data while output is switched to that sprite invalidates the display pointer.
- Palette support for new-type sprites requires RISC OS 3.6+.

---

## Geometry Tidying

### OS_SpriteOp 54 — Remove Left-hand Wastage

Word-aligns the left edge of the sprite by removing any unused bits in the first word of each row. If more than 32 bits are removed, the entire first word is dropped, reducing the sprite's word-width.

**Entry:** R0 = 54 + area flags; R1/R2 = area/sprite

**Exit:** R0–R2 preserved

---

## Append Sprite

### OS_SpriteOp 35 — Append Sprite

Merges two sprites side-by-side (horizontal) or top-to-bottom (vertical). The result is stored in sprite 1; sprite 2 is deleted. No extra memory is required.

**Entry:**
- R0 = 35 + area flags
- R1 = sprite area pointer
- R2 = sprite 1 pointer or name (receives result)
- R3 = sprite 2 pointer or name (deleted after merge)
- R4 = 0 (append horizontally) or 1 (append vertically)

**Exit:** R0–R4 preserved
