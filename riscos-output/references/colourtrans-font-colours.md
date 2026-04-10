# ColourTrans: Font Colours

SWIs for finding and setting the best available anti-aliased colour range for a given foreground/background pair. Anti-aliased fonts require a graduated range of intermediate colours between the background and the text foreground; these SWIs find the closest such range available in the current screen palette.

---

## ColourTrans_ReturnFontColours — SWI &4074E

Determines the best anti-aliasing colour range for a font, returning the logical colour numbers without actually setting them.

**Entry:**
- R0 = font handle (0 = use current font)
- R1 = background palette entry (`&BBGGRR00`)
- R2 = foreground (text) palette entry (`&BBGGRR00`)
- R3 = maximum colour offset requested (0–14; higher values allow more anti-alias levels)

**Exit:**
- R0 = preserved (font handle)
- R1 = background logical colour number
- R2 = foreground logical colour number
- R3 = maximum sensible offset actually available

**Notes:**
- The returned logical colour numbers are the two extremes of the chosen anti-alias ramp.
- R3 on exit may be less than R3 on entry if the palette cannot support the requested number of intermediate levels.
- Use this SWI to query what will be set before committing, or when you need the colour numbers for other purposes.

---

## ColourTrans_SetFontColours — SWI &4074F

Finds the best available anti-aliased colour range and sets the Font Manager to use it for subsequent font rendering.

**Entry:**
- R0 = font handle (0 = use current font)
- R1 = background palette entry (`&BBGGRR00`)
- R2 = foreground (text) palette entry (`&BBGGRR00`)
- R3 = maximum colour offset requested (0–14)

**Exit:**
- R0 = preserved (font handle)
- R1 = background logical colour number
- R2 = foreground logical colour number
- R3 = maximum sensible offset actually available

**Notes:**
- Internally calls `Font_SetFontColours` after determining the optimal colour range.
- After calling this SWI you **must** call `ColourTrans_InvalidateCache` if you subsequently change the palette via other means (VDU 19, `Wimp_SetPalette`, etc.), otherwise ColourTrans may use stale cached values.
- Typical call before rendering text: set R3 = 14 to request the maximum anti-alias quality; ColourTrans will reduce this to what the current palette allows.

---

## ColourTrans_InvalidateCache — SWI &40750

Flushes ColourTrans's internal colour cache. Must be called after any external palette modification that was not made through ColourTrans itself.

**Entry:** (none)

**Exit:** (none)

**When to call:**
- After `Font_SetFontColours` called directly
- After `Wimp_SetPalette`
- After `VDU 19` palette changes
- *Not* required after a mode change (the module detects those automatically)

---

## Anti-Alias Colour Offset

The *colour offset* (R3) controls how many intermediate shades are used between background and foreground:

| R3 value | Anti-alias levels |
|----------|-------------------|
| 0 | 2 (foreground + background only — no anti-aliasing) |
| 7 | 8 levels |
| 14 | 15 levels (maximum quality) |

On a 16-colour screen the maximum sensible offset is typically 7; on a 256-colour screen up to 14 is possible depending on the palette layout.

---

## Palette Entry Format

`&BBGGRR00` — blue (bits 31:24), green (bits 23:16), red (bits 15:8), low byte zero.

---

## Related SWIs

| SWI | Purpose |
|-----|---------|
| `ColourTrans_SetTextColour` (&40761) | Set plain (non-anti-aliased) text colour |
| `ColourTrans_InvalidateCache` (&40750) | Flush cache after external palette changes |
