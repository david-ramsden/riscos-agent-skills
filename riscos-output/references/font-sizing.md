# Font Manager: Font Sizing and Metrics

SWIs for obtaining font handles, querying sizes, measuring characters and strings, and converting between coordinate systems.

---

## Coordinate Systems

| Unit | Definition | Conversion |
|------|-----------|------------|
| **Millipoints** | 1/72000 inch | Internal unit; minimises rounding errors |
| **OS units** | 1/180 inch | Screen coordinate unit |
| **Points × 16** | Used in `Font_FindFont` | 1 point = 16 units here |

Default conversion: **400 millipoints per OS unit** (both axes). This can be read or changed with `Font_ReadScaleFactor` / `Font_SetScaleFactor`.

---

## Font Handles

### Font_FindFont — SWI &40081

Obtains a font handle for the named font at a given size. Increments the usage count; every call must be matched by a call to `Font_LoseFont`.

**Entry:**
- R1 = pointer to font identifier string (null- or CR-terminated)
- R2 = x point size × 16 (e.g. 192 for 12pt)
- R3 = y point size × 16
- R4 = x resolution in DPI (0 = default 90 DPI; -1 = current screen DPI)
- R5 = y resolution in DPI (0 = default; -1 = current)

**Exit:**
- R0 = font handle
- R4 = actual x DPI used
- R5 = actual y DPI used

**Font identifier qualifiers (appended to identifier string):**

| Qualifier | Meaning |
|-----------|---------|
| `\Fid\f` | Specify encoding by font identifier |
| `\Eid\e` | Specify encoding by encoding identifier |
| `\M(matrix)` | Apply 2×2 transformation matrix |

**Notes:**
- Sets the returned handle as the current font.
- RISC OS 2 ignores qualifiers.
- If an exact bitmap size is unavailable the nearest cached size is substituted.

---

### Font_LoseFont — SWI &40082

Releases a font handle obtained from `Font_FindFont`. Decrements the usage count; the font data may remain in cache.

**Entry:** R0 = font handle

**Exit:** R0 preserved

---

### Font_ReadDefn — SWI &40083

Reads the definition details of a font handle.

**Entry:**
- R0 = font handle
- R1 = pointer to buffer for identifier string (0 = query buffer size)
- R3 = &4C4C5546 (ASCII 'FULL') to request extended information; else 0

**Exit:**
- R2 = x point size × 16
- R3 = y point size × 16
- R4 = x DPI
- R5 = y DPI
- R6 = age (number of cache accesses since this font was last used)
- R7 = usage count (number of unreleased `Font_FindFont` calls)

---

## Bounding Boxes

### Font_ReadInfo — SWI &40084

Returns the overall bounding box of the font (the box that encloses every character).

**Entry:** R0 = font handle

**Exit:**
- R1 = min x (OS units, inclusive)
- R2 = min y (OS units, inclusive)
- R3 = max x (OS units, exclusive)
- R4 = max y (OS units, exclusive)

**Notes:** Prefer `Font_CharBBox` for individual characters; `Font_ReadInfo` is provided for compatibility.

---

### Font_CharBBox — SWI &4008E

Returns the bounding box of a single character.

**Entry:**
- R0 = font handle
- R1 = character code (ASCII)
- R2 = flags: bit 4 clear → millipoints; bit 4 set → OS units

**Exit:**
- R0 = preserved
- R1 = min x (inclusive)
- R2 = min y (inclusive)
- R3 = max x (exclusive)
- R4 = max y (exclusive)

**Notes:**
- Use millipoints for line-spacing calculations (avoids rounding accumulation).
- Use OS units when positioning on screen.

---

### Font_StringBBox — SWI &40097

Returns the bounding box of a string without rendering it. Coordinates are in millipoints relative to the string's origin.

**Entry:** R1 = pointer to string (null- or CR-terminated)

**Exit:**
- R1 = min x (millipoints, inclusive)
- R2 = min y (millipoints, inclusive)
- R3 = max x (millipoints, exclusive)
- R4 = max y (millipoints, exclusive)

**Notes:**
- Cannot be used to determine the pixel footprint on screen due to rounding; use `Font_ScanString` with bit 18 set for that.
- Uses the current font.

---

## String Measurement

### Font_StringWidth — SWI &40085

Measures the width of a string in the current font, optionally stopping at a split character.

**Entry:**
- R1 = pointer to string
- R2 = maximum x offset (millipoints; stop when reached)
- R3 = maximum y offset (millipoints)
- R4 = split character code (-1 = none)
- R5 = maximum number of characters to scan (-1 = whole string)

**Exit:**
- R1 = pointer to termination character
- R2 = x offset at termination (millipoints)
- R3 = y offset at termination (millipoints)
- R4 = number of split characters encountered
- R5 = index of termination character

**Notes:** Deprecated in favour of `Font_ScanString`. Retained for RISC OS 2 compatibility.

---

### Font_ScanString — SWI &400A1

Comprehensive string analyser: measures width, finds caret position, returns bounding box, or counts split characters. Does not render.

**Entry:**
- R0 = font handle (used only if R2 bit 8 is set)
- R1 = pointer to string
- R2 = flags (see below)
- R3 = x coordinate or maximum x offset (millipoints)
- R4 = y coordinate or maximum y offset (millipoints)
- R5 = pointer to bounding box buffer (if R2 bit 18 set)
- R6 = pointer to matrix buffer (if R2 bit 19 set)
- R7 = string length in bytes (if R2 bit 7 set)

**Exit:**
- R1 = pointer to termination character (or split character if bit 20 set)
- R3 = x offset at termination (millipoints)
- R4 = y offset at termination (millipoints)
- R7 = number of split characters found (if bit 20 set)

**Key R2 flags:**

| Bit | Meaning |
|-----|---------|
| 7 | R7 contains explicit string length |
| 8 | R0 contains font handle (override current) |
| 9 | Apply kerning |
| 10 | Right-to-left text |
| 17 | Return caret position nearest to (R3, R4); else return length |
| 18 | Return bounding box in buffer at R5 |
| 19 | Return final transformation matrix in buffer at R6 |
| 20 | Return split character count (word-wrap mode) |

**Notes:** Preferred replacement for `Font_StringWidth`, `Font_FindCaret`, `Font_FindCaretJ`. Not available in RISC OS 2.

---

## Coordinate Conversion

### Font_ConverttoOS — SWI &40088

Converts millipoints to OS units.

**Entry:** R1 = x (millipoints); R2 = y (millipoints)

**Exit:** R1 = x (OS units); R2 = y (OS units)

---

### Font_Converttopoints — SWI &40089

Converts OS units to millipoints.

**Entry:** R1 = x (OS units); R2 = y (OS units)

**Exit:** R0 corrupted; R1 = x (millipoints); R2 = y (millipoints)

---

### Font_ReadScaleFactor — SWI &4008F

Reads the current millipoints-per-OS-unit scale factors.

**Entry:** (none)

**Exit:** R1 = x scale; R2 = y scale (default both = 400)

---

### Font_SetScaleFactor — SWI &40090

Sets the millipoints-per-OS-unit scale factors.

**Entry:** R1 = x scale; R2 = y scale

**Exit:** R1/R2 preserved

**Warning:** Do not use under the Wimp desktop. If you must change this, save and restore the previous values.

---

## Full Metrics

### Font_ReadFontMetrics — SWI &4009F

Reads comprehensive typographic metrics from the font's IntMetrics file, including bounding boxes, character widths, and kerning pairs.

**Entry:**
- R0 = font handle
- R1 = buffer for bounding boxes (0 = query size)
- R2 = buffer for x widths (0 = query size)
- R3 = buffer for y widths (0 = query size)
- R4 = buffer for miscellaneous metrics (0 = query size)
- R5 = buffer for kern pair table (0 = query size)
- R6 = 0 (reserved)
- R7 = 0 (reserved)

**Exit:**
- R0 = flags
  - bit 1 set: no x kern offsets present
  - bit 2 set: no y kern offsets present
  - bit 3 set: more than 255 kern pairs
- R1–R5 = sizes written (or required sizes if buffers were 0)

**Notes:**
- Not available on transformed (matricised) fonts.
- Not available in RISC OS 2.
- Call with all buffer registers = 0 first to determine sizes, then allocate and call again.
