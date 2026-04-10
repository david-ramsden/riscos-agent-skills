# ColourTrans: Graphics Colours

SWIs for selecting and setting graphics foreground/background colours (GCOLs) to the closest or most contrasting available value for the current or a specified mode and palette.

---

## ColourTrans_ReturnGCOL — SWI &40742

Returns the GCOL closest to the requested palette entry for the current mode and palette, without setting it.

**Entry:**
- R0 = palette entry (`&BBGGRR00`)

**Exit:**
- R0 = closest GCOL

**Notes:**
Shorthand for `ColourTrans_ReturnGCOLForMode` with R1=-1 (current mode) and R2=-1 (current palette).

---

## ColourTrans_SetGCOL — SWI &40743

Sets the graphics GCOL to the closest match for the requested palette entry.

**Entry:**
- R0 = palette entry (`&BBGGRR00`)
- R3 = flags
  - bit 7 clear → foreground
  - bit 7 set → background
  - bit 8 set → use ECF patterns (extended colour fill)
- R4 = GCOL action (0 = store, 1 = OR, 2 = AND, 3 = XOR, 4 = invert, etc.)

**Exit:**
- R0 = GCOL chosen
- R2 = log₂(bits-per-pixel) for current mode
- R3 = entry R3 AND &80

---

## ColourTrans_ReturnGCOLForMode — SWI &40745

Returns the closest GCOL for a specified mode and palette combination, without setting it.

**Entry:**
- R0 = palette entry (`&BBGGRR00`)
- R1 = destination mode number (-1 = current mode)
- R2 = palette pointer (-1 = current palette; 0 = default palette)

**Exit:**
- R0 = closest GCOL

---

## ColourTrans_ReturnOppGCOL — SWI &40747

Returns the GCOL furthest (most contrasting) from the requested palette entry in the current mode.

**Entry:**
- R0 = palette entry (`&BBGGRR00`)

**Exit:**
- R0 = furthest GCOL

---

## ColourTrans_SetOppGCOL — SWI &40748

Sets the graphics GCOL to the colour furthest from the requested palette entry.

**Entry:**
- R0 = palette entry (`&BBGGRR00`)
- R3 = 0 (foreground) or 128 (background)
- R4 = GCOL action

**Exit:**
- R0 = GCOL chosen
- R2 = log₂(bits-per-pixel)
- R3 = entry R3 AND &80

---

## ColourTrans_ReturnOppGCOLForMode — SWI &4074A

Returns the most contrasting GCOL for a specified mode and palette, without setting it.

**Entry:**
- R0 = palette entry (`&BBGGRR00`)
- R1 = destination mode (-1 = current)
- R2 = palette pointer (-1 = current; 0 = default)

**Exit:**
- R0 = furthest GCOL

---

## Colour Numbers (256-colour modes)

In 256-colour modes the colour number and GCOL differ. Use these SWIs when a colour number rather than a GCOL is needed.

### ColourTrans_ReturnColourNumber — SWI &40744

**Entry:** R0 = palette entry
**Exit:** R0 = closest colour number

### ColourTrans_ReturnColourNumberForMode — SWI &40746

**Entry:** R0 = palette entry; R1 = mode (-1=current); R2 = palette (-1=current, 0=default)
**Exit:** R0 = closest colour number

### ColourTrans_ReturnOppColourNumber — SWI &40749

**Entry:** R0 = palette entry
**Exit:** R0 = furthest colour number

### ColourTrans_ReturnOppColourNumberForMode — SWI &4074B

**Entry:** R0 = palette entry; R1 = mode; R2 = palette
**Exit:** R0 = furthest colour number

---

## GCOL ↔ Colour Number Conversion (256-colour modes only)

### ColourTrans_GCOLToColourNumber — SWI &4074C

**Entry:** R0 = GCOL
**Exit:** R0 = colour number

### ColourTrans_ColourNumberToGCOL — SWI &4074D

**Entry:** R0 = colour number
**Exit:** R0 = GCOL

---

## ColourTrans_SetColour — SWI &4075E

Applies a known GCOL number directly to the graphics/text foreground or background without any colour matching.

**Entry:**
- R0 = GCOL number
- R3 = flags
  - bit 7 set → background
  - bit 9 set → text colour (rather than graphics)
- R4 = GCOL action

**Exit:** All registers preserved

---

## Palette Entry Format

`&BBGGRR00` — blue (bits 31:24), green (bits 23:16), red (bits 15:8), low byte zero.
