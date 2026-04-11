# Font Files Reference

RISC OS fonts are stored as collections of files inside a directory named after the font (e.g. `Trinity.Medium`). The directory is located via `Font$Path`. All multi-byte values are **little-endian** unless stated otherwise.

---

## Directory Structure

```
<Font$Path>.FontFamily.Weight/
    IntMetrics          - typographic metrics (character widths, kerning, etc.)
    IntMetric0          - per-encoding metrics (encoding 0)
    IntMetric1          - per-encoding metrics (encoding 1) …
    Outlines            - scalable outline data (preferred)
    f<x>x<y>            - pre-rendered outline cache (e.g. f9999x9999)
    b<dpi>x<dpi>        - pre-rendered bitmap cache (e.g. b90x45)
    x<x>y<y>            - old-style bitmap (e.g. x90y45)
    Encoding            - encoding definition (optional)
    Messages            - localised font name strings (optional)
```

Encoding subdirectories (for multi-encoding fonts) mirror this structure inside a subdirectory named by encoding identifier.

---

## IntMetrics / IntMetric_n Files

Contains all typographic data: character widths, bounding boxes, kern pairs, and miscellaneous metrics.

### Header (52 bytes)

| Offset | Size | Content |
|--------|------|---------|
| 0 | 40 | Font name, padded with CR (13) |
| 40 | 4 | Reserved; value 16 |
| 44 | 4 | Reserved; value 16 |
| 48 | 1 | `nlo` — low byte of character count |
| 49 | 1 | Version: 0, 1, or 2 |
| 50 | 1 | Flags byte (see below) |
| 51 | 1 | `nhi` — high byte of character count |

**Flag byte bits:**

| Bit | Meaning when set |
|-----|-----------------|
| 0 | No bounding box data present |
| 1 | No x-offset (advance width) data |
| 2 | No y-offset data |
| 3 | Additional (miscellaneous) data follows the main metrics |
| 5 | Character map size word precedes the map |
| 6 | Kern character codes are 16-bit (else 8-bit) |

### Following the Header

1. **Character mapping table** — 0–256 bytes; maps character codes to internal indices. Size depends on version and flag bit 5.
2. **Bounding boxes** (if flag bit 0 clear) — 8 bytes per character: `(x0, y0, x1, y1)` as four signed 16-bit values in design units.
3. **X offsets / advance widths** (if flag bit 1 clear) — 2 bytes per character, signed 16-bit.
4. **Y offsets** (if flag bit 2 clear) — 2 bytes per character.
5. **Miscellaneous data area** (if flag bit 3 set) — pointed to by an 8-byte extended-data offset block; contains:
   - Font maximum bounding box
   - Default x/y advance offsets
   - Italic angle metric
   - Underline position and thickness
   - Typographic heights: CapHeight, XHeight, Descender, Ascender

### Kern Pair Data

Appended after the main metrics. Each entry contains:
- Left character code (1 or 2 bytes, per flag bit 6)
- Right character code (1 or 2 bytes)
- x kern amount (signed 16-bit, design units)
- y kern amount (signed 16-bit, design units)

---

## Old-style Bitmap Files (x90y45)

4-bits-per-pixel indexed character bitmaps, named by resolution (e.g. `x90y45` for 90×45 DPI).

### Index Section

Each entry is 16 bytes:

| Offset | Size | Content |
|--------|------|---------|
| 0 | 1 | Point size |
| 1 | 1 | Bits per pixel |
| 2 | 2 | x resolution (DPI) |
| 4 | 2 | y resolution (DPI) |
| 6 | 2 | Reserved |
| 8 | 4 | File offset to pixel data block |
| 12 | 4 | Size of pixel data block |

A single zero byte terminates the index.

### Pixel Data Block

- Bounding box and size specification
- 512-byte character offset table (one word per character)
- Character rows stored **bottom to top**, 4 bits per pixel

---

## New-style Bitmap Files (b*x*)

Word-aligned format supporting 1-bpp and 4-bpp, named by size and DPI (e.g. `b90x45`).

### File Header (≥ 36 bytes for version < 8)

| Offset | Size | Content |
|--------|------|---------|
| 0 | 4 | `"FONT"` magic word |
| 4 | 1 | Bits per pixel (1 or 4) |
| 5 | 1 | Version (4–8+) |
| 6 | 2 | Flags |
| 8 | 8 | Maximum bounding box: x0, y0, x1, y1 (four signed 16-bit values) |
| 16+ | — | Chunk offset array |

Following the header:
- **Table start word** — size of scaffold/table region
- **X/Y size and resolution data**
- **Character offset index** — 512 bytes (one word per character code)

### Character Data

Variable-length per character; begins with a flag byte:

| Bit(s) | Meaning |
|--------|---------|
| 0–1 | Coordinate size: 0=none, 1=8-bit, 2=12-bit |
| 2 | Data format: 0=uncompressed, 1=compressed |
| 3 | Initial colour (0 or 1) |
| 4 | Character type: 0=normal, 1=composite |

Followed optionally by bounding box, then pixel rows (bottom to top).

---

## Outline File (Outlines)

Stores resolution-independent character outlines as bezier/spline paths.

### File Header

| Offset | Size | Content |
|--------|------|---------|
| 0 | 4 | `"FONT"` magic word |
| 4 | 1 | 0 (indicates outlines, not bitmap) |
| 5 | 1 | Version (4–8+) |
| 6 | 2 | Design size (units per em) |
| 8 | 8 | Maximum bounding box |
| 16+ | — | Chunk offsets |

### Scaffold Data

The scaffold provides a set of reference lines (stems, serifs) that hints use to align outlines to the pixel grid:

- **Base character reference** — which character provides the scaffold
- **Scaffold lines** — each is: coordinate (design units), link index, width

### Character Chunks

- **Character index entries** — 32 bytes each (128 bytes with sub-pixel variants): flags, bounding box, offset into data area
- **Dependency bytes** — for composite characters (accented glyphs built from components)
- **Outline segments** — variable length; path commands encoding on-curve and off-curve (bezier control) points

---

## Encoding Files

Plain text files mapping character positions to glyph names.

```
# Optional comment
# BaseEncoding Latin1
# Alphabet 101

/.notdef          ; position 0 must always be .notdef
/A
/B
…
/.NotDef          ; null/unused positions
```

- Lines beginning with `#` are comments or metadata.
- `BaseEncoding` specifies the encoding this one derives from.
- `Alphabet` specifies the RISC OS alphabet number.
- Glyph names use PostScript naming conventions.
- Character 0 must be `.notdef`; unused slots use `.NotDef` (case-sensitive distinction).

---

## Messages Files

Standard RISC OS Messages format (token=value pairs) used to provide localised font names. Special tokens:

| Token | Meaning |
|-------|---------|
| `Encoding_xxx` | Name of encoding `xxx` (base-encoding dependent) |
| `BEncoding_xxx` | Name of base encoding `xxx` |
| `Font_xxx` | Name of fixed font `xxx` |
| `LFont_xxx` | Name of alphabet-varying font `xxx` |

---

## Cached Outline Files (f*x*)

Pre-rendered outline caches (e.g. `f9999x9999`) have the same header format as the Outlines file but contain pre-processed hint data and pre-computed path transformations for fast rendering at common sizes. Created by `Font_MakeBitmap` (SWI &40099) with R6 bit 0 set.

---

## Design Units

Outline coordinates are expressed in **design units** — an arbitrary scale whose size is recorded in the Outlines file header (design size field). Typical values are 1000 or 2048 units per em. The Font Manager scales from design units to millipoints at render time using the requested point size.
