# RISC OS Sprites: Management

SWIs for loading, saving, merging, enumerating, copying, renaming and deleting sprites and sprite files.

---

## Sprite Area Initialisation

### OS_SpriteOp 9 — Initialise Sprite Area

Prepares a user sprite area for use. The caller must have pre-filled the area header before calling.

**Entry:**
- R0 = 9
- R1 = pointer to sprite area

**Exit:** R0/R1 preserved

**Pre-initialisation required by caller:**

```asm
; Before calling SpriteOp 9:
STR  r_size, [area, #0]   ; total area size in bytes
MOV  tmp, #16
STR  tmp,   [area, #8]    ; offset to first sprite = 16 (after header)
SWI  OS_SpriteOp          ; R0=9, R1=area ptr
```

The OS then sets the sprite count (offset 4) to 0 and the free-word pointer (offset 12).

---

### OS_SpriteOp 8 — Read Area Control Block

Queries the current state of a sprite area without modifying it.

**Entry:**
- R0 = 8
- R1 = sprite area pointer

**Exit:**
- R0/R1 preserved
- R2 = total area size in bytes
- R3 = number of sprites in area
- R4 = offset to first sprite
- R5 = offset to first free word

---

## Loading and Saving

### OS_SpriteOp 10 — Load Sprite File

Loads all sprites from a file, **overwriting** any sprites already in the area.

**Entry:**
- R0 = 10 + area flags
- R1 = sprite area pointer
- R2 = pointer to filename string

**Exit:** R0–R2 preserved

**Notes:** The area must be initialised first. The file's header word (area size) is not loaded; the OS uses the area's own header.

---

### OS_SpriteOp 11 — Merge Sprite File

Merges sprites from a file into the area. Existing sprites with the same name as an incoming sprite are replaced. The area must have sufficient free space to hold both old and new data simultaneously during the merge.

**Entry:**
- R0 = 11 + area flags
- R1 = sprite area pointer
- R2 = pointer to filename string

**Exit:** R0–R2 preserved

---

### OS_SpriteOp 12 — Save Sprite File

Saves all sprites in the area to a file. The area header word (offset 0, total size) is **not** written; the file begins at offset 4.

**Entry:**
- R0 = 12 + area flags
- R1 = sprite area pointer
- R2 = pointer to filename string

**Exit:** R0–R2 preserved

---

### OS_SpriteOp 3 — Screen Load

Loads a sprite file and plots it directly to the screen at the bottom-left of the graphics window. Changes the screen mode if the sprite's mode differs from the current one.

**Entry:**
- R0 = 3
- R2 = pointer to filename string

**Exit:** R0/R2 preserved

---

## Querying Sprites

### OS_SpriteOp 40 — Read Sprite Information

Returns the dimensions, mask status, and mode of a sprite.

**Entry:**
- R0 = 40 + area flags
- R1 = sprite area pointer
- R2 = sprite pointer or name

**Exit:**
- R0–R2 preserved
- R3 = width in pixels
- R4 = height in pixels
- R5 = mask presence: 0 = no mask; 1 = mask present
- R6 = mode (mode number or sprite mode word)

---

### OS_SpriteOp 13 — Return Sprite Name

Returns the name of a sprite by its ordinal position (1-indexed) in the area.

**Entry:**
- R0 = 13 + area flags
- R1 = sprite area pointer
- R2 = pointer to output buffer
- R3 = maximum name length
- R4 = sprite number (1 = first sprite)

**Exit:**
- R0–R2 preserved
- R3 = actual name length (excluding null terminator)
- R4 preserved

**Notes:** Returns a null-terminated string. Use this iteratively (R4 = 1, 2, 3, …) to enumerate all sprites. Stop when R4 > sprite count (from `SpriteOp 8`).

---

## Copying and Renaming

### OS_SpriteOp 27 — Copy Sprite

Duplicates a sprite within the same area under a new name. Requires free space for the copy.

**Entry:**
- R0 = 27 + area flags
- R1 = sprite area pointer
- R2 = source sprite pointer or name
- R3 = pointer to new name string

**Exit:** R0–R3 preserved. Error if new name already exists.

---

### OS_SpriteOp 26 — Rename Sprite

Renames a sprite in-place (no memory movement).

**Entry:**
- R0 = 26 + area flags
- R1 = sprite area pointer
- R2 = sprite pointer or name
- R3 = pointer to new name string

**Exit:** R0–R3 preserved. Error if new name already exists.

---

## Deleting Sprites

### OS_SpriteOp 25 — Delete Sprite

Removes a sprite from the area and compacts the free space.

**Entry:**
- R0 = 25 + area flags
- R1 = sprite area pointer
- R2 = sprite pointer or name

**Exit:** R0–R2 preserved

**Warning:** Any sprite control block pointers obtained before this call may be invalidated by the compaction, since later sprites are moved down in memory.

---

## Star Commands

| Command | Equivalent to |
|---------|--------------|
| `*SLoad file` | SpriteOp 10 on system area |
| `*SMerge file` | SpriteOp 11 on system area |
| `*SSave file` | SpriteOp 12 on system area |
| `*SGet name` | SpriteOp 14 on system area |
| `*SCopy src dst` | SpriteOp 27 on system area |
| `*SRename old new` | SpriteOp 26 on system area |
| `*SDelete name…` | SpriteOp 25 on system area |
| `*SNew` | Clear all system sprites |
| `*SList` | List all system sprites |
| `*SInfo` | Show system sprite workspace status |
| `*ScreenSave file` | SpriteOp 2 |
| `*ScreenLoad file` | SpriteOp 3 |
| `*Configure SpriteSize nK` | Set system sprite area size |

---

## Sprite Naming Rules

- Maximum 12 characters.
- Names are case-insensitive for lookup purposes.
- The name is stored null-padded to 12 bytes in the control block.
- Duplicate names within an area are not permitted.
