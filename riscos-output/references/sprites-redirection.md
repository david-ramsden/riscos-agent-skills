# RISC OS Sprites: Output Redirection

Output redirection causes all VDU graphics and text output to be written into a sprite (image or mask) rather than the screen. This is the primary mechanism for off-screen rendering.

---

## Concepts

When output is redirected:
- All VDU operations (text, graphics primitives, `OS_Plot`, font rendering via `Font_Paint`) write into the sprite rather than the screen.
- The current VDU context (colours, windows, cursor positions, ECF patterns, etc.) is saved to a **save area** provided by the caller and restored when output is switched back.
- Redirection can target either the **image** pixels or the **mask** pixels of a sprite.
- R2 = 0 restores output to the screen.

**Important restrictions while output is redirected:**
- Do not call `Wimp_Poll` — the Wimp may attempt to redraw windows onto the sprite.
- Left-hand wastage is removed from the sprite on entry (see `SpriteOp 54`).
- Sprite control block pointers obtained before switching may be stale after the switch.

---

## OS_SpriteOp 60 — Switch Output to Sprite Image

Redirects VDU output to the image data of a sprite.

**Entry:**
- R0 = 60 + area flags
- R1 = sprite area pointer
- R2 = sprite pointer or name; **0 = restore output to screen**
- R3 = save area:
  - 0 = no save area (previous context is lost; do not use if you need to restore)
  - 1 = use the OS system save area
  - any other value = pointer to caller-supplied save area (first word must be 0 on first use)

**Exit:**
- R0 preserved
- R1 = previous sprite area pointer
- R2 = previous sprite pointer (or 0 if was screen)
- R3 = previous save area pointer

**Pattern for save and restore:**

```asm
; Switch to sprite
MOV  r0, #60 + 512          ; reason 60, sprite pointer form
LDR  r1, sprite_area
LDR  r2, sprite_ptr
ADR  r3, save_area           ; save_area first word must be 0
SWI  OS_SpriteOp
STR  r1, old_area            ; save previous context
STR  r2, old_sprite
STR  r3, old_save

; ... render into sprite ...

; Restore screen output
MOV  r0, #60 + 512
LDR  r1, old_area
MOV  r2, #0                  ; 0 = restore to screen
LDR  r3, old_save
SWI  OS_SpriteOp
```

---

## OS_SpriteOp 61 — Switch Output to Sprite Mask

Redirects VDU output to the **mask** of a sprite. Mask pixels are written as solid (all bits 1) or transparent (all bits 0); intermediate colour values are not meaningful in the mask.

**Entry:**
- R0 = 61 + area flags
- R1 = sprite area pointer
- R2 = sprite pointer or name; **0 = restore output to screen**
- R3 = save area (same as SpriteOp 60)

**Exit:**
- R0 preserved
- R1 = previous sprite area pointer
- R2 = previous sprite pointer (or 0)
- R3 = previous save area pointer

**Notes:**
- The sprite must already have a mask (created with `SpriteOp 29`) before switching to it.
- Colour restrictions apply: only colour 0 (transparent) and colour 1 or the foreground colour (solid) produce correct mask pixels.
- To create a mask programmatically: switch output to mask, flood-fill or draw the desired shape, then switch back.

---

## OS_SpriteOp 62 — Read Save Area Size

Returns the number of bytes required for a save area for a given sprite (or the screen). The size is constant for a given RISC OS version and sprite depth; query it once and allocate accordingly.

**Entry:**
- R0 = 62 + area flags
- R1 = sprite area pointer
- R2 = sprite pointer or name; **0 = query save area size for screen**

**Exit:**
- R0–R2 preserved
- R3 = required save area size in bytes

**Notes:** The save area must be word-aligned. Allocate at least this many bytes and zero the first word before the first call to SpriteOp 60 or 61.

---

## Save Area Contents

The save area stores the complete VDU context including:

- ECF (Extended Colour Fill) patterns
- Current foreground and background colours and GCOL actions
- Graphics and text windows (clip rectangles)
- Graphics and text cursor positions
- Graphics origin
- VDU status flags and queue
- Character size metrics
- Spool file handle

The format is internal to the OS and varies between RISC OS versions. Do not read or write it directly; treat the save area as an opaque blob.

---

## Service_SwitchingOutputToSprite — Service Call &72

Broadcast when output is switched to or from a sprite. Modules that cache screen state (e.g. graphics accelerators) should use this to invalidate their caches.

**Entry:**
- R1 = &72
- R2–R5 = the R0–R3 values passed to the `OS_SpriteOp` call

**Notes:** This service call is not claimed; all modules receive it.

---

## Typical Off-screen Rendering Workflow

```
1.  Allocate sprite area (e.g. 64K)
2.  SpriteOp 9   — initialise area
3.  SpriteOp 15  — create sprite (width, height, mode)
4.  SpriteOp 29  — add mask (if transparency needed)
5.  SpriteOp 62  — query save area size
6.  Allocate save area, zero first word
7.  SpriteOp 60  — switch output to sprite image
8.  ... draw content using VDU, OS_Plot, Font_Paint, etc. ...
9.  SpriteOp 60  — restore output to screen (R2=0)
10. SpriteOp 52  — plot sprite onto screen with ColourTrans table
```
