# RISC OS Mode Information

SWIs and OS calls for querying properties of the current or a named screen mode.

---

## OS_ReadModeVariable — SWI &35

Reads a single variable describing the properties of a given mode.

**Entry:**
- R0 = mode specifier (mode number, selector pointer, etc.; −1 = current mode)
- R1 = variable number (see table below)

**Exit:**
- R0 = mode specifier (preserved or normalised)
- R1 = variable number (preserved)
- R2 = variable value
- C flag set: variable or mode is invalid

**Notes:**
- Can be called for any valid mode, not just the current one.
- Passing R0 = −1 queries the mode currently in use on screen.
- The C flag is the error indicator; no OS error is generated for an invalid variable.

### Mode Variable Numbers

| Number | Name | Meaning |
|--------|------|---------|
| 0 | `ModeFlags` | Mode characteristic flags (see below) |
| 1 | `ScrRCol` | Number of text columns − 1 |
| 2 | `ScrBCol` | Number of text rows − 1 |
| 3 | `NColour` | Number of colours − 1 (see table below) |
| 4 | `XEigFactor` | Horizontal EIG factor (OS units per pixel = 2^n) |
| 5 | `YEigFactor` | Vertical EIG factor |
| 6 | `LineLength` | Screen row width in bytes |
| 7 | `ScreenSize` | Total screen memory in bytes |
| 8 | `YShftFactor` | log₂(LineLength / 2) — used for fast row addressing |
| 9 | `Log2BPP` | log₂(bits per pixel) |
| 10 | `Log2BPC` | log₂(bits per character cell) |
| 11 | `XWindLimit` | Right-hand pixel column number (pixels − 1) |
| 12 | `YWindLimit` | Top pixel row number (pixels − 1) |

### ModeFlags (variable 0) Bit Meanings

| Bit | Meaning when set |
|-----|-----------------|
| 0 | Non-graphic mode (text or teletext only) |
| 1 | Teletext mode |
| 2 | Gap mode (blank lines between text rows) |
| 3 | Hardware scrolling available |
| 4 | Colour output (if clear: greyscale/monochrome) |
| 5 | Hi-res texture supported |

### NColour Values (variable 3)

| NColour | Colours | bpp |
|---------|---------|-----|
| 1 | 2 | 1 |
| 3 | 4 | 2 |
| 15 | 16 | 4 |
| 63 | 64 | 8 (VIDC1 tinted) |
| 255 | 256 | 8 |
| 65535 | 65536 | 16 |
| &FFFFFFFF | 16M | 32 |

### EIG Factor Meanings (variables 4 and 5)

| EIG value | OS units per pixel | Example |
|-----------|-------------------|---------|
| 0 | 1 | 1:1 square pixels |
| 1 | 2 | Wide pixels (320×256 modes) |
| 2 | 4 | Very wide pixels |
| 3 | 8 | — |

For mode 12 (640×256): XEigFactor=1, YEigFactor=2 → 2 OS units/pixel × 4 OS units/pixel = 1280×1024 OS unit screen.

### Log2BPP Values (variable 9)

| Log2BPP | Bits per pixel |
|---------|---------------|
| 0 | 1 |
| 1 | 2 |
| 2 | 4 |
| 3 | 8 |
| 4 | 16 |
| 5 | 32 |

---

## OS_ReadVduVariables — SWI &31

Reads multiple VDU system variables describing the **current** screen state in one call. Unlike `OS_ReadModeVariable`, this reads live state (cursor position, window bounds, etc.) not just static mode properties.

**Entry:**
- R0 = pointer to a word array of variable numbers, terminated by −1
- R1 = pointer to output word buffer (must be at least as large as input array)

**Exit:** R0/R1 preserved; output buffer filled with corresponding values

**Notes:**
- VDU variables include both mode-derived values (same as `OS_ReadModeVariable`) and current state variables (cursor coordinates, graphics origin, clipping window, etc.).
- Variable numbers 0–12 correspond to the same mode variables as `OS_ReadModeVariable`.
- Additional variables (128+) provide current cursor, window, and colour state.

---

## OS_Byte 135 — Read Character at Cursor / Screen Mode

Returns both the character code at the current text cursor position and the current screen mode number.

**Entry:** R0 = 135

**Exit:**
- R1 = character code at text cursor position (0 if no character)
- R2 = current screen mode number

**Notes:** Only returns a mode *number* (0–127); if the current mode was selected via a mode selector, R2 may not reflect the full mode specification. Use `OS_ScreenMode 1` for the full specifier.

---

## Mode Properties Quick-Calculation Examples

Given `OS_ReadModeVariable` results, common derived values:

**Pixel width in OS units:**
```
pixel_os_width = 1 << XEigFactor
```

**Screen width in OS units:**
```
screen_os_width = (XWindLimit + 1) << XEigFactor
```

**Bytes per pixel:**
```
bytes_per_pixel = (1 << Log2BPP) / 8    ; (0 for sub-byte depths)
```

**Row address of pixel (x, y):**
```
address = screen_base + (y * LineLength) + (x >> (3 - Log2BPP))
```
(For sub-byte depths the bit offset within the byte also depends on `Log2BPP`.)

---

## Common Mode Variable Values for Key Modes

| Property | Mode 12 | Mode 13 | Mode 15 | Mode 27 | Mode 28 | Mode 31 |
|----------|---------|---------|---------|---------|---------|---------|
| Pixels | 640×256 | 320×256 | 640×256 | 640×480 | 640×480 | 800×600 |
| NColour | 15 | 255 | 255 | 15 | 255 | 15 |
| Log2BPP | 2 | 3 | 3 | 2 | 3 | 2 |
| XEigFactor | 1 | 2 | 1 | 1 | 1 | 1 |
| YEigFactor | 2 | 2 | 2 | 1 | 1 | 1 |
| OS units | 1280×1024 | 1280×1024 | 1280×1024 | 1280×960 | 1280×960 | 1600×1200 |
| LineLength | 320 | 320 | 640 | 320 | 640 | 400 |
| ScreenSize | 81920 | 81920 | 163840 | 153600 | 307200 | 240000 |
