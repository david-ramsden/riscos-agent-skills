# RISC OS Mode Numbers

All predefined screen modes 0–46 (mode 32 is not defined). Each mode is identified by a number 0–127; bit 7 of the mode byte selects the shadow screen bank.

---

## Monitor Types

| Type | Standard | Notes |
|------|----------|-------|
| 0 | 50 Hz TV / PAL | Colour or monochrome |
| 1 | Multi-frequency | Supports 50, 56, 60, 64 Hz |
| 2 | 64 Hz hi-res mono | High-resolution monochrome only |
| 3 | 60 Hz VGA | Standard VGA monitor |
| 4 | Super-VGA | RISC OS 3+ |
| 5 | LCD | RISC OS 3+ |

Monitor type appears in the `Monitors` column below as a comma-separated list of type numbers.

---

## Complete Mode Table

| Mode | Text cols×rows | Pixels | OS units | Colours | Memory | Hz | MB/s | Monitors |
|------|---------------|--------|----------|---------|--------|----|------|----------|
| 0 | 80×32 | 640×256 | 1280×1024 | 2 | 20K | 50 | 1.0 | 0,1,3,4,5 |
| 1 | 40×32 | 320×256 | 1280×1024 | 4 | 20K | 50 | 1.0 | 0,1,3,4,5 |
| 2 | 20×32 | 160×256 | 1280×1024 | 16 | 40K | 50 | 2.0 | 0,1,3,4,5 |
| 3 | 80×25 | text only | — | 2 | 40K | 50 | 2.0 | 0,1,3,4,5 |
| 4 | 40×32 | 320×256 | 1280×1024 | 2 | 20K | 50 | 1.0 | 0,1,3,4,5 |
| 5 | 20×32 | 160×256 | 1280×1024 | 4 | 20K | 50 | 1.0 | 0,1,3,4,5 |
| 6 | 40×25 | text only | — | 2 | 20K | 50 | 1.0 | 0,1,3,4,5 |
| 7 | 40×25 | teletext | — | 16 | 80K | 50 | 4.0 | 0,1,3,4,5 |
| 8 | 80×32 | 640×256 | 1280×1024 | 4 | 40K | 50 | 2.0 | 0,1,3,4,5 |
| 9 | 40×32 | 320×256 | 1280×1024 | 16 | 40K | 50 | 2.0 | 0,1,3,4,5 |
| 10 | 20×32 | 160×256 | 1280×1024 | 256 | 80K | 50 | 4.0 | 0,1,3,4,5 |
| 11 | 80×25 | 640×250 | 1280×1000 | 4 | 40K | 50 | 2.0 | 0,1,3,4,5 |
| **12** | **80×32** | **640×256** | **1280×1024** | **16** | **80K** | **50** | **4.0** | **0,1,3,4,5** |
| **13** | **40×32** | **320×256** | **1280×1024** | **256** | **80K** | **50** | **4.0** | **0,1,3,4,5** |
| 14 | 80×25 | 640×250 | 1280×1000 | 16 | 80K | 50 | 3.9 | 0,1,3,4,5 |
| **15** | **80×32** | **640×256** | **1280×1024** | **256** | **160K** | **50** | **8.0** | **0,1,3,4,5** |
| 16 | 132×32 | 1056×256 | 2112×1024 | 16 | 132K | 50 | 6.6 | 0,1 |
| 17 | 132×25 | 1056×250 | 2112×1000 | 16 | 132K | 50 | 6.5 | 0,1 |
| 18 | 80×64 | 640×512 | 1280×1024 | 2 | 40K | 50 | 2.0 | 1 |
| 19 | 80×64 | 640×512 | 1280×1024 | 4 | 80K | 50 | 4.0 | 1 |
| 20 | 80×64 | 640×512 | 1280×1024 | 16 | 160K | 50 | 8.0 | 1 |
| 21 | 80×64 | 640×512 | 1280×1024 | 256 | 320K | 50 | 16.0 | 1 |
| 22 | 96×36 | 768×288 | 768×576 | 16 | 108K | 50 | 5.4 | 0,1 |
| 23 | 144×56 | 1152×896 | 2304×1792 | 2 | 126K | 64 | 8.1 | 2 |
| 24 | 132×32 | 1056×256 | 2112×1024 | 256 | 264K | 50 | 13.2 | 0,1 |
| 25 | 80×60 | 640×480 | 1280×960 | 2 | 38K | 60 | 2.3 | 1,3,4,5 |
| 26 | 80×60 | 640×480 | 1280×960 | 4 | 75K | 60 | 4.5 | 1,3,4,5 |
| **27** | **80×60** | **640×480** | **1280×960** | **16** | **150K** | **60** | **9.0** | **1,3,4,5** |
| **28** | **80×60** | **640×480** | **1280×960** | **256** | **300K** | **60** | **18.0** | **1,3,4,5** |
| 29 | 100×75 | 800×600 | 1600×1200 | 2 | 59K | 56 | 3.3 | 1,4 |
| 30 | 100×75 | 800×600 | 1600×1200 | 4 | 117K | 56 | 6.6 | 1,4 |
| **31** | **100×75** | **800×600** | **1600×1200** | **16** | **234K** | **56** | **13.2** | **1,4** |
| **32** | *(not defined)* | | | | | | | |
| 33 | 96×36 | 768×288 | 1536×1152 | 2 | 27K | 50 | 1.4 | 0,1 |
| 34 | 96×36 | 768×288 | 1536×1152 | 4 | 54K | 50 | 2.7 | 0,1 |
| 35 | 96×36 | 768×288 | 1536×1152 | 16 | 108K | 50 | 5.4 | 0,1 |
| 36 | 96×36 | 768×288 | 1536×1152 | 256 | 216K | 50 | 10.8 | 0,1 |
| 37 | 112×44 | 896×352 | 1792×1408 | 2 | 39K | 60 | 2.3 | 1 |
| 38 | 112×44 | 896×352 | 1792×1408 | 4 | 77K | 60 | 4.6 | 1 |
| 39 | 112×44 | 896×352 | 1792×1408 | 16 | 154K | 60 | 9.2 | 1 |
| 40 | 112×44 | 896×352 | 1792×1408 | 256 | 308K | 60 | 18.5 | 1 |
| 41 | 80×44 | 640×352 | 1280×1408 | 2 | 28K | 60 | 1.7 | 1,3,4,5 |
| 42 | 80×44 | 640×352 | 1280×1408 | 4 | 55K | 60 | 3.3 | 1,3,4,5 |
| 43 | 80×44 | 640×352 | 1280×1408 | 16 | 110K | 60 | 6.6 | 1,3,4,5 |
| 44 | 80×25 | 640×200 | 1280×800 | 2 | 16K | 60 | 0.9 | 1,3,4,5 |
| 45 | 80×25 | 640×200 | 1280×800 | 4 | 31K | 60 | 1.9 | 1,3,4,5 |
| 46 | 80×25 | 640×200 | 1280×800 | 16 | 63K | 60 | 3.8 | 1,3,4,5 |

Modes 0–17 and 22–24 run at 50 Hz on TV-type monitors. Modes 18–21 require a multi-frequency monitor. Modes with fewer than 352 vertical pixels display centred with blank scan lines (letterbox) on VGA/SVGA monitors at 70 Hz effective rate.

---

## Commonly Used Modes

### Mode 12 — 640×256, 16 colours, 50 Hz
The standard RISC OS desktop mode on early hardware. Square-ish pixels (1280×1024 OS units); 4bpp; 80 text columns.

### Mode 13 — 320×256, 256 colours, 50 Hz
Low-resolution 8bpp mode; wide pixels (each pixel = 4 OS units wide). Popular for games and paint programs.

### Mode 15 — 640×256, 256 colours, 50 Hz
High-colour 8bpp equivalent of mode 12. Requires 160K screen memory; the highest-colour 50 Hz mode available on all monitors.

### Mode 27 — 640×480, 16 colours, 60 Hz
Standard VGA resolution, 4bpp. First mode requiring a VGA or better monitor. Approximately square pixels (1280×960 OS units).

### Mode 28 — 640×480, 256 colours, 60 Hz
VGA resolution at 8bpp. Requires 300K screen memory. The most common mode for VGA-era RISC OS desktop use.

### Mode 31 — 800×600, 16 colours, 56 Hz
SVGA resolution, 4bpp. Requires a Super-VGA monitor (type 4). 234K screen memory.

### Mode 32 — *Not defined*
Mode 32 has no definition in the standard RISC OS mode table. Applications requiring resolutions beyond mode 31 should use a **mode selector** block rather than a mode number.

---

## Notes on 256-Colour Modes

In 256-colour modes using VIDC1 hardware, only 64 base colours can be selected directly; four levels of tinting expand those to 256 shades. VIDC20-based systems provide a full 256-entry palette without this restriction.

## Shadow Modes

Setting bit 7 of the mode byte selects the shadow screen bank (e.g. mode 12 + 128 = mode 140). The shadow bank is a second screen buffer; switching between banks provides double-buffering without re-drawing.
