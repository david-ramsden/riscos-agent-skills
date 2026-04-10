# Font Manager: Font Rendering

SWIs and techniques for painting strings on screen, positioning the text caret, redirecting output to Draw files, and pre-rendering bitmaps.

---

## Font_Paint — SWI &40086

Renders a string to the screen (or to a Draw-file buffer if `Font_SwitchOutputToBuffer` is active).

**Entry:**
- R0 = font handle (used only when R2 bit 8 is set; otherwise current font is used)
- R1 = pointer to string (null- or CR-terminated)
- R2 = plot type flags (see below)
- R3 = x coordinate
- R4 = y coordinate
- R5 = pointer to coordinate block (if R2 bit 5 set)
- R6 = pointer to transformation matrix (if R2 bit 6 set)
- R7 = string length in bytes (if R2 bit 7 set)

**Exit:** R1–R7 preserved

**R2 plot type flags:**

| Bit | Meaning |
|-----|---------|
| 0 | Use graphics cursor for justification spacing |
| 1 | Plot rubout (background) box |
| 2–3 | Reserved (set to 0) |
| 4 | Coordinates in OS units (if clear: millipoints) |
| 5 | R5 points to justification/rubout coordinate block |
| 6 | R6 points to 2×2 transformation matrix |
| 7 | R7 contains explicit string length |
| 8 | R0 contains font handle (override current) |
| 9 | Apply kerning |
| 10 | Right-to-left rendering |

**Coordinate block at R5 (when bit 5 set):**

| Offset | Content |
|--------|---------|
| 0 | Space character x justification (millipoints) |
| 4 | Space character y justification |
| 8 | Letter x justification |
| 12 | Letter y justification |
| 16 | Rubout box x0 |
| 20 | Rubout box y0 |
| 24 | Rubout box x1 |
| 28 | Rubout box y1 |

---

## Embedded Control Sequences

Control characters within the string passed to `Font_Paint` (or scanned by `Font_ScanString`) modify rendering behaviour:

| Code | Dec | Action |
|------|-----|--------|
| &09 | 9 | Move x by next two signed bytes (in OS units) |
| &0B | 11 | Move y by next two signed bytes (in OS units) |
| &11 | 17 | Set foreground/background colour (next two bytes = logical colours) |
| &12 | 18 | Set colour range offset (next byte = offset, then logical colour pair) |
| &13 | 19 | Set colour using ColourTrans (next byte = offset, then palette entries) |
| &15 | 21 | Comment / skip n bytes (next byte = count) |
| &19 | 25 | Underline: next byte = position, following byte = thickness |
| &1A | 26 | Change font: next word = new font handle |
| &1B | 27 | Set transformation matrix inline |
| &1C | 28 | Reset transformation matrix to identity |

---

## Caret

### Font_Caret — SWI &40087

Paints a text caret (vertical bar with serifs) at a specified position.

**Entry:**
- R0 = caret colour (XORed onto screen)
- R1 = caret height in OS units
- R2 = flags: bit 4 clear → R3/R4 in millipoints; bit 4 set → OS units
- R3 = x coordinate
- R4 = y coordinate

**Exit:** R0–R4 preserved

---

## Caret / Hit-test Positioning

### Font_FindCaret — SWI &4008D

Finds the nearest character position in a string to given coordinates. Deprecated — use `Font_ScanString` with bit 17 set instead (except on RISC OS 2).

**Entry:**
- R1 = pointer to string
- R2 = x offset (millipoints)
- R3 = y offset (millipoints)

**Exit:**
- R1 = pointer to character at caret position
- R2 = x offset at that character
- R3 = y offset at that character
- R4 = number of printable characters before caret
- R5 = byte index into string

---

### Font_FindCaretJ — SWI &40096

As `Font_FindCaret` but for a justified string. Deprecated — use `Font_ScanString` instead (except on RISC OS 2).

**Entry:**
- R1 = pointer to string
- R2 = x offset (millipoints)
- R3 = y offset (millipoints)
- R4 = x justification offset
- R5 = y justification offset

**Exit:** Same as `Font_FindCaret`

---

## Font Selection

### Font_SetFont — SWI &4008A

Sets the current font for subsequent `Font_Paint` and measurement operations.

**Entry:** R0 = font handle

**Exit:** R0 preserved

**Notes:** The current font can also be overridden inline during `Font_Paint` using control sequence &1A (change font) or by setting R2 bit 8 and providing a handle in R0.

---

### Font_CurrentFont — SWI &4008B

Queries the currently active font and colour settings.

**Entry:** (none)

**Exit:**
- R0 = current font handle
- R1 = background logical colour
- R2 = foreground logical colour
- R3 = foreground offset (anti-aliasing levels, −14 to +14)

---

### Font_FutureFont — SWI &4008C

Returns the font and colour state that will result after scanning a string (i.e. the state `Font_ScanString` would leave behind).

**Entry:** (none)

**Exit:** Same registers as `Font_CurrentFont`

**Use:** Call after `Font_StringWidth` to determine the final font state without committing to it.

---

## Redirecting Output to a Draw File

### Font_SwitchOutputToBuffer — SWI &4009E

Redirects `Font_Paint` output into a Draw-file path object buffer instead of the screen. Allows font outlines to be captured as scalable Draw paths.

**Entry:**
- R0 = flags (0 when R1 > 0)
  - bit 0: update pointer only (do not render)
  - bit 1: apply hinting to outlines
  - bit 4: error if character has no outline (bitmap-only font)
- R1 = pointer to buffer (must start with a null word and a free-space count word)
       8 = count mode (return required space without filling)
       0 = disable redirection
      −1 = query current state

**Exit:**
- R0 = previous flags
- R1 = previous buffer pointer (incremented by space consumed)

**Notes:** Not available in RISC OS 2.

---

## Pre-rendering

### Font_MakeBitmap — SWI &40099

Pre-renders and caches a font at a specific size so that first use is fast.

**Entry:**
- R1 = font handle or pointer to font identifier string
- R2 = x size × 16
- R3 = y size × 16
- R4 = x DPI
- R5 = y DPI
- R6 = flags
  - bit 0: create outline cache file (f9999x9999); if clear: bitmap (b9999x9999)
  - bit 1: horizontal sub-pixel rendering
  - bit 2: vertical sub-pixel rendering
  - bit 3: delete the cached file instead of creating it

**Exit:** (none)
