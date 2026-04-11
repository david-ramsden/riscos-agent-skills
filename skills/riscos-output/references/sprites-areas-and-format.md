# RISC OS Sprites: Areas and Data Format

Reference for sprite area layout, sprite control block structure, mode words, palette format, and pixel layout.

---

## Sprite Areas

A sprite area is a contiguous block of memory containing a header followed by one or more sprite control blocks. Three kinds exist:

| Kind | Description |
|------|-------------|
| System area | Built-in OS workspace; publicly accessible. Avoid in commercial applications. |
| Wimp pool | Shared pool managed by the Wimp; used for Wimp sprites (e.g. window furniture). |
| User area | Private block allocated by the application; referenced by its address. |

### Sprite Area Header (16 bytes)

| Offset | Size | Content |
|--------|------|---------|
| 0 | word | Total size of sprite area in bytes (set by caller before initialising) |
| 4 | word | Number of sprites in area |
| 8 | word | Byte offset to first sprite from start of area |
| 12 | word | Byte offset to first free word (= end of last sprite) |

When initialising a user area with `OS_SpriteOp 9`, the caller must pre-fill offset 0 (total size) and offset 8 (first sprite offset, typically 16). The OS fills offsets 4 and 12.

---

## Sprite Control Block

Each sprite immediately follows the previous one (or the area header). All offsets within the sprite are relative to the start of that sprite's control block.

| Offset | Size | Content |
|--------|------|---------|
| 0 | word | Byte offset to next sprite from start of this sprite (0 if last) |
| 4 | 12 bytes | Sprite name: up to 12 characters, null-padded, case-insensitive |
| 16 | word | Width of sprite in words − 1 |
| 20 | word | Height in scan lines − 1 |
| 24 | word | First bit used in left-most word of each row |
| 28 | word | Last bit used in right-most word of each row |
| 32 | word | Byte offset to sprite image data (from start of control block) |
| 36 | word | Byte offset to transparency mask (from start of control block); equals offset 32 if no mask |
| 40 | word | Mode: screen mode number (old format) or sprite mode word (new format) |
| 44+ | — | Palette data (optional; see below) |

**Width calculation:**

```
pixel_width = ((width_in_words + 1) * 32 - first_bit - (31 - last_bit)) / bits_per_pixel
```

---

## Sprite Mode Word (RISC OS 3.5+)

Used at offset 40 of the control block instead of a mode number. Distinguishable from a mode number by bit 0 being set.

| Bits | Content |
|------|---------|
| 0 | Always 1 (identifies this as a mode word, not a mode number) |
| 1–13 | Horizontal DPI (typically 180, 90, 45, 22) |
| 14–26 | Vertical DPI (same typical values) |
| 27–30 | Sprite type (see table below) |
| 31 | 1 = alpha channel mask; 0 = binary (1-bit) mask |

### Sprite Types

| Type | Bits/pixel | Colours | Notes |
|------|-----------|---------|-------|
| 0 | — | — | Old-format (backward compatible; mode number, not mode word) |
| 1 | 1 | 2 | |
| 2 | 2 | 4 | |
| 3 | 4 | 16 | |
| 4 | 8 | 256 | |
| 5 | 16 | 32K | |
| 6 | 32 | 16M | |

**Mask format for new-type sprites:** Always 1 bit per pixel, regardless of image depth. Each mask row is word-aligned; layout is identical to a 1bpp sprite image.

---

## Pixel Formats

### 16bpp (type 5)

| Bits | Component |
|------|-----------|
| 0–4 | Red (5 bits) |
| 5–9 | Green (5 bits) |
| 10–14 | Blue (5 bits) |
| 15 | Reserved (0) |

### 32bpp (type 6)

| Bits | Component |
|------|-----------|
| 0–7 | Red |
| 8–15 | Green |
| 16–23 | Blue |
| 24–31 | Reserved (0); or alpha when sprite mode word bit 31 is set |

---

## Palette Format

Palette data occupies offset 44+ in the control block (between the control block header and the image data). It is optional; its presence is indicated by the image offset at offset 32 being greater than 44.

Each palette entry is **two words** (8 bytes):

| Word | Content |
|------|---------|
| 0 | First physical colour (`&BBGGRR00`) |
| 1 | Second physical colour (`&BBGGRR00`) — typically the same as word 0 (used for flashing colours on VIDC1) |

**Entry count by depth:**

| Depth | Default entries | With 256-entry flag (R3 bit 31 set in SpriteOp 37) |
|-------|----------------|-----------------------------------------------------|
| 1bpp | 2 | — |
| 2bpp | 4 | — |
| 4bpp | 16 | — |
| 8bpp | 16 or 64 | 256 |

Palettes in new-type sprites require RISC OS 3.6 or later (RISC OS 3.5 did not support them).

---

## R0 Encoding for OS_SpriteOp

Every `OS_SpriteOp` call encodes both the reason code and the interpretation of R1/R2 in R0:

```
R0 = reason_code + area_and_pointer_flags
```

| Addition | Bits 8–9 | R1 | R2 |
|----------|---------|----|----|
| +0 | 00 | Not used | Pointer to sprite name (system area) |
| +256 | 01 | Pointer to user sprite area | Pointer to sprite name |
| +512 | 10 | Pointer to user sprite area | Pointer to sprite control block |

**Example:** To call reason 28 (Put Sprite) using a sprite control block pointer in a user area:
```
R0 = 28 + 512 = 540
```

---

## Plot Action Values

Used in R5 for rendering calls (reason codes 28, 34, 52, 56):

| Value | Action |
|-------|--------|
| 0 | Overwrite with sprite colour |
| 1 | OR with screen colour |
| 2 | AND with screen colour |
| 3 | XOR with screen colour |
| 4 | Invert screen colour |
| 5 | No change to screen |
| 6 | AND NOT with screen colour |
| 7 | OR NOT with screen colour |
| +8 | Add 8 to any action to use the transparency mask |

---

## Scale Factors Block

Used by reason codes 50, 51, 52, 53, 55, 56. A pointer to this 4-word block is passed in R6 (0 = no scaling):

| Offset | Content |
|--------|---------|
| 0 | x multiplier |
| 4 | y multiplier |
| 8 | x divisor |
| 12 | y divisor |

Scaled pixel size = original size × multiplier / divisor.
