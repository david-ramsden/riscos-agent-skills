# Font Manager: Font Colours

The Font Manager renders text using anti-aliasing: up to 16 intermediate logical colours are used between the background and foreground to produce smooth edges. The SWIs below control how those colours are chosen, configured, and queried.

---

## Anti-aliasing Overview

When a font is painted, the Font Manager selects a ramp of logical colours from background (offset 0) to foreground (offset *n*). The offset parameter controls how many intermediate shades are available:

| Offset | Levels |
|--------|--------|
| 0 | 2 (no anti-aliasing) |
| 7 | 8 |
| 14 | 15 (maximum) |

The actual colours used depend on the current palette. Under the desktop, use `ColourTrans_SetFontColours` (SWI &4074F) rather than `Font_SetFontColours` directly, so that the physical palette is considered.

---

## Font_SetFontColours — SWI &40092

Sets the font, background colour, foreground colour, and anti-aliasing offset for subsequent rendering.

**Entry:**
- R0 = font handle (0 = current font)
- R1 = background logical colour number
- R2 = foreground logical colour number
- R3 = foreground offset (−14 to +14; positive = anti-alias ramp above background; negative = below)

**Exit:** R0–R3 preserved

**Notes:**
- The offset defines the size of the anti-aliasing colour ramp.
- Under the Wimp desktop prefer `Wimp_SetFontColours` or `ColourTrans_SetFontColours`, which automatically find the best physical palette match.

---

## Font_SetPalette — SWI &40093

Directly configures the 16-entry anti-aliasing palette, mapping logical colour indices to physical colours.

**Entry:**
- R1 = background logical colour
- R2 = foreground logical colour
- R3 = offset
- R4 = physical background colour (`&BBGGRR00`)
- R5 = physical foreground colour (`&BBGGRR00`)
- R6 = &65757254 (ASCII 'True') to indicate 24-bit physical colours; else 0

**Exit:** R1–R6 preserved

**Warning:** Do not use under the Wimp desktop. Use `Wimp_SetFontColours` or `ColourTrans_SetFontColours` (SWI &4074F) instead, which handle palette arbitration correctly.

---

## Font_CurrentFont — SWI &4008B

Queries the currently active font handle and colour settings.

**Entry:** (none)

**Exit:**
- R0 = current font handle
- R1 = background logical colour
- R2 = foreground logical colour
- R3 = current foreground offset

---

## Font_FutureFont — SWI &4008C

Returns the font and colour state that would result after `Font_Paint` / `Font_ScanString` processes the current string (accounting for embedded colour control sequences).

**Entry:** (none)

**Exit:** Same registers as `Font_CurrentFont`

---

## Font_ReadColourTable — SWI &40098

Reads the 16-entry logical colour table currently used for anti-aliasing.

**Entry:** R1 = pointer to a 16-byte buffer

**Exit:** R1 preserved; buffer filled with 16 logical colour numbers (index 0 = background, index *n* = foreground)

---

## Anti-aliasing Thresholds

Thresholds control when the Font Manager switches between using more or fewer anti-aliasing levels depending on the rendered size. The table format is:

- Byte 0: foreground offset currently in use
- Subsequent bytes: threshold values (pixel sizes at which each offset is active)
- Terminator: &FF

### Font_ReadThresholds — SWI &40094

**Entry:** R1 = pointer to buffer

**Exit:** R1 preserved; buffer filled with threshold table

---

### Font_SetThresholds — SWI &40095

**Entry:** R1 = pointer to new threshold table (same format as above)

**Exit:** R1 preserved

**Notes:** Rarely needed; the default threshold table is appropriate for most applications.

---

## ColourTrans Integration

The recommended desktop workflow for setting font colours:

```
; R0 = font handle, R1 = background &BBGGRR00, R2 = foreground &BBGGRR00, R3 = max offset (14)
SWI ColourTrans_SetFontColours  ; &4074F
```

`ColourTrans_SetFontColours` calls `Font_SetFontColours` internally after mapping physical palette entries to the best available logical colours. After changing the palette by other means (VDU 19, Wimp_SetPalette) call `ColourTrans_InvalidateCache` (SWI &40750) to flush stale values.

---

## Inline Colour Control Sequences (within Font_Paint strings)

| Code | Action |
|------|--------|
| &11 (17) | Set foreground/background: next two bytes = logical colour numbers |
| &12 (18) | Set colour range offset: next byte = offset, next two bytes = logical colours |
| &13 (19) | Set colour via ColourTrans: next byte = offset, next two words = palette entries (`&BBGGRR00`) |
