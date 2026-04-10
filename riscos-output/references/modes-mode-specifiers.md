# RISC OS Mode Specifiers

A *mode specifier* is the single value or block used wherever a mode must be identified — in `OS_ScreenMode`, `OS_CheckModeValid`, `ColourTrans_*ForMode`, sprite headers, and so on. Four distinct forms exist.

---

## 1. Mode Numbers (integers 0–127)

The simplest form: a plain integer 0–127 selects a predefined mode from the standard mode table. Bit 7 selects the shadow screen bank.

```
mode_number = 15        ; mode 15 (640×256, 256 colours)
shadow_mode = 12 + 128  ; mode 12 in shadow bank
```

**Availability:** All RISC OS versions.

**Limitations:** Only the ~46 predefined modes are available; no way to request arbitrary resolutions or frame rates.

---

## 2. Mode Selector Blocks

A word-aligned data structure in memory that fully specifies a mode. Allows arbitrary resolutions, colour depths, and frame rates not available as numbered modes.

**Identification:** Bit 0 of the first word is set (value 1); bits 1–7 are the format number (0 for current format).

### Structure

| Offset | Size | Content |
|--------|------|---------|
| 0 | word | Flags: bit 0 must be 1; bits 1–7 = format (0); bits 8–31 reserved (0) |
| 4 | word | X resolution in pixels |
| 8 | word | Y resolution in pixels |
| 12 | word | Pixel depth code (see table below) |
| 16 | word | Frame rate in Hz; −1 = use highest available |
| 20+ | — | Mode variable pairs: [variable_number, value] (see [Mode Information](modes-mode-information.md)) |
| n | word | Terminator: −1 |

### Pixel Depth Codes

| Code | Bits/pixel | Colours |
|------|-----------|---------|
| 0 | 1 | 2 |
| 1 | 2 | 4 |
| 2 | 4 | 16 |
| 3 | 8 | 256 |
| 4 | 16 | 32768 (15-bit) |
| 5 | 32 | 16M (24-bit in 32-bit word) |

### Example — 1024×768 at 8bpp, 60 Hz

```asm
mode_selector
    DCD 1           ; flags: format 0, bit 0 set
    DCD 1024        ; X resolution
    DCD 768         ; Y resolution
    DCD 3           ; 8bpp (256 colours)
    DCD 60          ; 60 Hz
    DCD -1          ; terminator (no extra mode variables)
```

### Mode Variable Pairs

Optional pairs at offset 20+ override specific mode variables. Each pair is two words: [variable_index, value]. Terminate the list with a single word −1. Useful overrides:

| Variable | Index | Example use |
|----------|-------|-------------|
| `XEigFactor` | 4 | Set OS units per pixel ratio in x |
| `YEigFactor` | 5 | Set OS units per pixel ratio in y |

**Availability:** RISC OS 3.5+. The mode selector pointer must be word-aligned.

---

## 3. Sprite Mode Words

A 32-bit word describing a sprite's pixel format and DPI. Used in sprite control blocks (offset 40) and with `ColourTrans` operations. Unlike mode selectors, sprite mode words carry no resolution information.

**Identification:** Bit 0 is set, and bits 27–30 are non-zero.

### Bit Layout

| Bits | Content |
|------|---------|
| 0 | Always 1 (mode word flag) |
| 1–13 | Horizontal DPI |
| 14–26 | Vertical DPI |
| 27–30 | Sprite type (see table) |
| 31 | 1 = alpha channel; 0 = binary mask |

### Sprite Types

| Type | Bits/pixel | Colours/format |
|------|-----------|----------------|
| 1 | 1 | 2 colours |
| 2 | 2 | 4 colours |
| 3 | 4 | 16 colours |
| 4 | 8 | 256 colours |
| 5 | 16 | 32K colours |
| 6 | 32 | 16M colours (24-bit in 32-bit) |

### Typical DPI values: 180, 90, 45, 22

### Example — 90×90 DPI, 8bpp, binary mask

```
mode_word = (4 << 27) | (90 << 14) | (90 << 1) | 1
          = &08B40B41
```

**Availability:** RISC OS 3.5+; sprite palettes supported from 3.6+.

---

## 4. Mode Strings

A human-readable text string specifying a mode by its properties. Parsed by `OS_ScreenMode` (reason code 0) and `OS_CheckModeValid`.

**Identification:** A pointer to a string that begins with a letter or digit (not word-aligned as a selector).

### Parameters (space- or comma-separated)

| Parameter | Meaning |
|-----------|---------|
| `X####` | X resolution in pixels |
| `Y####` | Y resolution in pixels |
| `C###` | Colours: 2, 4, 16, 64, 256, 32T, 32K, 16M |
| `G###` | Greys: 4, 16, 256 |
| `T###` | Teletext mode with *n* colours |
| `EX#` | X EIG factor (0–3) |
| `EY#` | Y EIG factor (0–3) |
| `F##` | Frame rate in Hz |
| `TX#` | Teletext character width in pixels |
| `TY#` | Teletext character height in pixels |

### Colour Specifiers

| Token | Meaning |
|-------|---------|
| `C2` | 2 colours (1bpp) |
| `C4` | 4 colours (2bpp) |
| `C16` | 16 colours (4bpp) |
| `C64` | 64 colours (VIDC1 tinted 8bpp) |
| `C256` | 256 colours (8bpp) |
| `C32T` | 32-bit true colour |
| `C32K` | 32768 colours (16bpp) |
| `C16M` | 16M colours (24bpp in 32-bit) |

### Examples

```
"X640 Y480 C256 F60"        ; 640×480, 256 colours, 60 Hz
"X800 Y600 C16"             ; 800×600, 16 colours, best frame rate
"X1024 Y768 C16M F75"       ; 1024×768, 16M colours, 75 Hz
```

**Availability:** RISC OS 3.5+.

---

## Identifying Specifier Type

Given a value *v*:

| Condition | Type |
|-----------|------|
| v < 256 | Mode number |
| v is word-aligned pointer, word at v has bit 0 set | Mode selector block |
| v is word-aligned pointer, word at v has bits 27–30 non-zero and bit 0 set | Sprite mode word |
| v is pointer to string | Mode string |

The OS distinguishes mode selectors from sprite mode words by checking whether bits 1–7 (the format field) are in the valid range for a mode selector.

---

## 16bpp and 32bpp Pixel Formats

**16bpp (type 5 / depth 4):**

| Bits | Component |
|------|-----------|
| 0–4 | Red (5 bits) |
| 5–9 | Green (5 bits) |
| 10–14 | Blue (5 bits) |
| 15 | Reserved |

**32bpp (type 6 / depth 5):**

| Bits | Component |
|------|-----------|
| 0–7 | Red |
| 8–15 | Green |
| 16–23 | Blue |
| 24–31 | Reserved (or alpha if sprite mode word bit 31 set) |
