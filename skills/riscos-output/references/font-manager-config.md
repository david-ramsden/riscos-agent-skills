# Font Manager: Configuration and Management

SWIs, star commands, and service calls relating to the Font Manager cache, font discovery, path management, and cache persistence.

---

## Cache Status

### Font_CacheAddr — SWI &40080

Returns version information and cache usage statistics.

**Entry:** (none)

**Exit:**
- R0 = version × 100 (e.g. 242 for version 2.42)
- R2 = total cache size (bytes)
- R3 = cache currently used (bytes)

---

## Font Enumeration

### Font_ListFonts — SWI &40091

Enumerates available fonts or builds a Wimp menu of fonts. Call repeatedly, incrementing the counter in R2 bits 0–15 each time, until R2 returns −1.

**Entry:**
- R1 = pointer to buffer for font identifier (or menu structure)
- R2 = counter and flags word
  - bits 0–15: enumeration counter (0 on first call)
  - bits 16–31: option flags (see below)
- R3 = size of R1 buffer
- R4 = pointer to buffer for font names / menu data
- R5 = size of R4 buffer
- R6 = pointer to font identifier to tick in menu (0 = none; 1 = 'System font')

**Exit:**
- R2 = updated counter (−1 when enumeration is complete)
- R3 = bytes written to R1 buffer (or required size)
- R5 = bytes written to R4 buffer (or required size)

**Option flags (bits 16–31 of R2 on entry):**

| Bit | Meaning |
|-----|---------|
| 16 | Return font identifier in R1 buffer |
| 17 | Return localised font name in R4 buffer |
| 18 | Use CR (13) as string terminator (else null) |
| 19 | Build Wimp menu structure |
| 20 | Include 'System font' entry in enumeration |
| 21 | Tick the font specified in R6 |
| 22 | List available encodings rather than fonts |

**Notes:**
- Use `Font$Path` for RISC OS 2 compatibility.
- Fonts are found by searching `Font$Path` directories for subdirectories that contain `IntMetrics` or `Outlines` files.

---

### Font_DecodeMenu — SWI &400A0

Converts a Wimp menu selection (from a menu built by `Font_ListFonts`) back into a font identifier string, including encoding qualifiers.

**Entry:**
- R0 = flags
  - bit 0: this is an encoding sub-menu selection
- R1 = pointer to menu definition block (from `Font_ListFonts`)
- R2 = pointer to Wimp menu selection array
- R3 = pointer to output buffer (0 = query required size)
- R4 = size of output buffer

**Exit:**
- R0/R1 preserved
- R2 = pointer to next element of selection array
- R4 = required buffer size

**Output format:** `\Fidentifier\fterritory name` for a font, or `\Eidentifier\eterritory name` for an encoding.

**Notes:** Not available in RISC OS 2.

---

## Cache Tuning

### Font_ReadFontMax — SWI &4009C

Reads the current cache threshold settings.

**Entry:** (none)

**Exit:**
- R0 = FontMax (maximum cache size, KB)
- R1 = FontMax1 (threshold for discarding pixel data)
- R2 = FontMax2 (threshold for discarding bitmap data)
- R3 = FontMax3 (threshold for outline data)
- R4 = FontMax4 (threshold for accent data)
- R5 = FontMax5 (threshold for metric data)
- R6/R7 = corrupted

**Notes:** Returned values reflect the internal state and may differ from the `*Configure` settings after a cache reorganisation.

---

### Font_SetFontMax — SWI &4009B

Sets new cache threshold values and triggers a cache reorganisation.

**Entry:**
- R0 = FontMax
- R1 = FontMax1
- R2 = FontMax2
- R3 = FontMax3
- R4 = FontMax4
- R5 = FontMax5
- R6 = 0 (reserved)
- R7 = 0 (reserved)

**Exit:** (none)

---

## Cache File Management

### Font_UnCacheFile — SWI &4009A

Removes a specific font file from the cache, optionally reloading from disc.

**Entry:**
- R1 = pointer to full filename (as constructed via `Font$Path`)
- R2 = 0 to remove from cache; 1 to reload into cache

**Exit:** (none)

**Pattern for updating a font on disc:**
1. Call with R2=0 to evict the old cached data.
2. Replace the file on disc.
3. Call with R2=1 to reload the updated file.

---

### Font_ReadFontPrefix — SWI &4009D

Returns the full directory path for a font handle's source directory.

**Entry:**
- R0 = font handle
- R1 = pointer to output buffer
- R2 = buffer size in bytes

**Exit:**
- R0 = preserved
- R1 = pointer to null terminator in buffer
- R2 = remaining buffer space

**Example output:** `ADFS::4.$.!Fonts.Trinity.Medium.`

---

## Star Commands

| Command | Purpose |
|---------|---------|
| `*FontList` | Display cache size and list of currently cached fonts |
| `*FontCat` | List all fonts found on the current `Font$Path` |
| `*FontInstall dir` | Prepend *dir* to `Font$Path` so its fonts are available |
| `*FontRemove dir` | Remove *dir* from `Font$Path` |
| `*Configure FontSize n` | Set minimum cache reservation in KB |
| `*Configure FontMax n` | Set the maximum cache size threshold |
| `*SaveFontCache file` | Save the current cache state to a file |
| `*LoadFontCache file` | Restore a previously saved cache state |

---

## System Variables

| Variable | Purpose |
|----------|---------|
| `Font$Path` | Colon-separated list of directories searched for fonts |

Fonts are located by searching each directory in `Font$Path` for a subdirectory whose name is the font identifier (e.g. `Trinity.Medium`). That subdirectory must contain at least an `IntMetrics` and either `Outlines` or a bitmap file.

---

## Service Calls

### Service_FontsChanged — &6E

Broadcast when `Font$Path` changes (e.g. after `*FontInstall` or `*FontRemove`). Modules that cache font lists should re-enumerate fonts in response.

---

## Configuration Thresholds (FontMax1–5)

| Threshold | Controls eviction of |
|-----------|----------------------|
| FontMax1 | Pixel (rendered) data |
| FontMax2 | Bitmap cache files |
| FontMax3 | Outline data |
| FontMax4 | Accent composite data |
| FontMax5 | Metrics data |

When the cache size falls below a threshold, the corresponding data type is discarded first. Higher values make the cache more aggressive about keeping that data type.
