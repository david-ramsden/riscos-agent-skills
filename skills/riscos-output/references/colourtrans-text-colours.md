# ColourTrans: Text Colours

SWIs for setting the text foreground/background colour to the closest (or most contrasting) available GCOL for the current mode and palette.

---

## ColourTrans_SetTextColour — SWI &40761

Sets the text foreground or background colour to the GCOL closest to the requested palette entry.

**Entry:**
- R0 = palette entry (`&BBGGRR00` format)
- R3 = flags
  - bit 7 clear → foreground colour
  - bit 7 set → background colour

**Exit:**
- R0 = GCOL number chosen
- R3 = preserved

**Notes:**
Equivalent to `ColourTrans_SetGCOL` with the text flag applied. Handles all screen modes automatically.

---

## ColourTrans_SetOppTextColour — SWI &40762

Sets the text foreground or background colour to the GCOL *furthest* (most contrasting) from the requested palette entry.

**Entry:**
- R0 = palette entry (`&BBGGRR00` format)
- R3 = flags
  - bit 7 clear → foreground colour
  - bit 7 set → background colour

**Exit:**
- R0 = GCOL number chosen
- R3 = preserved

**Notes:**
Useful for ensuring readable text on any background by automatically selecting a high-contrast colour.

---

## Palette Entry Format

Palette entries throughout ColourTrans use the format `&BBGGRR00` — blue in bits 31:24, green in bits 23:16, red in bits 15:8, and the least-significant byte always zero.

---

## Related SWIs

| SWI | Purpose |
|-----|---------|
| `ColourTrans_SetGCOL` (&40743) | Set graphics foreground/background GCOL |
| `ColourTrans_ReturnGCOL` (&40742) | Return closest GCOL without setting it |
| `ColourTrans_SetFontColours` (&4074F) | Set anti-aliased font colours |
