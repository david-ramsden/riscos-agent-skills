# ColourTrans: Sprite Rendering with Pixel Translation Tables

Pixel translation tables map each pixel value in a source sprite to the best available colour in the destination mode/palette. They are passed to `OS_SpriteOp` (or equivalent) when plotting sprites to ensure correct colour reproduction across different screen modes.

---

## ColourTrans_SelectTable — SWI &40740

Creates a pixel translation table that maps source pixel values to destination pixel values.

**Entry:**
- R0 = source mode
  - -1 → current screen mode
  - ≥ 256 → pointer to a sprite
- R1 = source palette
  - -1 → current screen palette
  - ≥ 256 → sprite name pointer (if R0 is a sprite area) or sprite pointer
- R2 = destination mode (-1 = current screen mode)
- R3 = destination palette (-1 = current palette; 0 = default palette for mode)
- R4 = pointer to buffer to receive table, or 0 to query the required buffer size
- R5 = flags (see below)
- R6 = transfer function workspace pointer (only if R5 bit 2 set)
- R7 = transfer function pointer (only if R5 bit 2 set)

**Exit:**
- R0–R3 = preserved
- R4 = required buffer size in bytes (only when R4 was 0 on entry)

**R5 Flags:**

| Bit | Meaning |
|-----|---------|
| 0 | R0 is a sprite pointer (not a sprite area pointer) |
| 1 | Use default palette for source even if palette is present |
| 2 | Apply transfer function to each colour (R6/R7 must be set) |
| 24–31 | Output format: 0 = pixel values, 1 = GCOLs, 2 = colour numbers, 3 = palette entries |

**Typical usage pattern:**
1. Call with R4=0 to obtain the table size.
2. Allocate a buffer of that size.
3. Call again with R4 pointing to the buffer.
4. Pass the buffer to `OS_SpriteOp 52` (Put Sprite Scaled) or similar.

---

## ColourTrans_SelectGCOLTable — SWI &40741

Generates a list of GCOL values corresponding to each entry in the source palette, translated for the destination mode. Primarily useful for non-256-colour work where a simple GCOL list is sufficient.

**Entry:**
- R0 = source mode (-1 = current; ≥256 = sprite area pointer)
- R1 = source palette (-1 = current; ≥256 = sprite name or pointer)
- R2 = destination mode (-1 = current)
- R3 = destination palette (-1 = current; 0 = default)
- R4 = pointer to output buffer
- R5 = flags
  - bit 0: R0/R1 identify a sprite by pointer rather than area+name
  - bit 1: use default source palette even if the sprite has its own

**Exit:** R0–R5 preserved

**Notes:**
Unlike `SelectTable`, this always produces GCOLs and does not support querying the buffer size first — the caller must pre-allocate enough space (one byte per source colour entry).

---

## ColourTrans_GenerateTable — SWI &40763

Enhanced replacement for `SelectTable`, available from RISC OS 3.10 onwards. The calling convention is identical to `SelectTable` but the flags word (R5) is validated more strictly and additional format options may be available.

**Entry:** Same as `ColourTrans_SelectTable`

**Exit:** Same as `ColourTrans_SelectTable`

**Notes:**
Prefer `GenerateTable` over `SelectTable` in new code targeting RISC OS 3.10+. The behaviour for invalid flag combinations is defined (error returned) rather than undefined.

---

## Relationship to OS_SpriteOp

After building a translation table with any of the above SWIs, pass it in R7 when calling `OS_SpriteOp` reason codes that accept a translation table, for example:

| OS_SpriteOp reason | Operation |
|--------------------|-----------|
| 52 (&34) | Put Sprite Scaled (with table in R7) |
| 53 (&35) | Create Mask from Colour (with table in R7) |

A null pointer (0) in place of the table pointer tells the Sprite operations to use a default 1:1 mapping.

---

## Related SWIs

| SWI | Purpose |
|-----|---------|
| `ColourTrans_ReadPalette` (&4075C) | Read palette for a mode or sprite |
| `ColourTrans_ReturnGCOLForMode` (&40745) | Closest GCOL for a given mode/palette |
