# RISC OS Mode Selection

How to request a screen mode change, validate a mode before selecting it, and enumerate available modes.

---

## VDU 22 — Change Screen Mode

The simplest way to change mode. Sends a single-byte mode number to the VDU system.

```
VDU 22, mode_number
```

Or from assembler:
```
MOV R0, #22
SWI OS_WriteC
MOV R0, #mode_number
SWI OS_WriteC
```

**Effect:** Changes to the requested mode, reinitialises the text cursor and graphics window to defaults, and resets the palette. Mode number must be 0–127 (add 128 for shadow bank). Cannot accept mode selector blocks; use `OS_ScreenMode 0` for that.

---

## OS_ScreenMode — SWI &65

The primary interface for mode selection, enquiry, and enumeration. Behaviour is selected by the reason code in R0.

---

### OS_ScreenMode 0 — Select Screen Mode

Selects a new screen mode. Accepts any mode specifier type.

**Entry:**
- R0 = 0
- R1 = mode specifier (see [Mode Specifiers](modes-mode-specifiers.md))
  - Integer 0–127: mode number
  - Bit 7 set in low byte: shadow bank
  - Pointer to mode selector block (word-aligned, bit 0 of first word set)
  - Pointer to mode string (not word-aligned, or string begins with a digit/letter)

**Exit:** All registers preserved

**Notes:**
- Triggers `Service_ModeChanging` before the change and `Service_ModeChanged` after.
- If the mode is unavailable for the current monitor, an error is returned.
- Under the Wimp desktop, use `Wimp_SetMode` instead; direct mode changes bypass the desktop's mode management.

---

### OS_ScreenMode 1 — Return Current Mode Specifier

**Entry:** R0 = 1

**Exit:** R1 = current mode specifier
- Returns the mode number if selected by number
- Returns a pointer to the mode selector if selected by block

---

### OS_ScreenMode 2 — Enumerate Available Modes

Returns descriptions of modes supported by the current monitor and hardware. Acts as a front-end to `Service_EnumerateScreenModes`, automatically supplying monitor type, bandwidth, and VRAM parameters.

**Entry:**
- R0 = 2
- R2 = number of modes to skip (0 on first call)
- R6 = pointer to output buffer (0 to query required space)
- R7 = output buffer size in bytes

**Exit:**
- R1 = mode specifier of the first returned mode
- R2 = negative count of modes placed in buffer
- R6 = pointer to next free byte in buffer
- R7 = remaining buffer space

**Each returned entry contains:**
- Entry size (word)
- Flags (word)
- X resolution in pixels (word)
- Y resolution in pixels (word)
- Pixel depth code 0–5 (word)
- Frame rate in Hz (word)
- Mode name string (variable length, null-terminated)

---

## OS_CheckModeValid — SWI &26

Tests whether a given mode specifier is valid for the current monitor and available screen memory, without actually changing mode.

**Entry:** R0 = mode specifier (mode number or pointer to mode selector)

**Exit:**
- N flag set: mode is invalid; R1 = suggested alternative mode (or -1 if no alternative)
- N flag clear: mode is valid; R1 = mode specifier (normalised)
- V flag set: error occurred

**Notes:**
- Returns a suggested alternative if the requested mode cannot be displayed on the current monitor.
- Use this before calling `OS_ScreenMode 0` to avoid unexpected mode changes.

---

## *LoadModeFile — Star Command

Loads a monitor definition file containing custom mode timing data.

**Syntax:** `*LoadModeFile <pathname>`

After loading, the monitor type is set to 7 (file-defined). Modes defined in the file are then available via `OS_ScreenMode 0` and `OS_CheckModeValid`.

---

## Mode Change Sequence

When a mode change is performed (by any method), the OS:

1. Broadcasts `Service_ModeChanging` (&46) — modules may veto the change by claiming it.
2. Reprograms the video hardware (VIDC) with timing data from `Service_ModeExtension`.
3. Clears and reinitialises the screen buffer.
4. Resets the text cursor, graphics window, and palette to mode defaults.
5. Broadcasts `Service_ModeChanged` (&47) — modules update their internal state.

---

## Wimp Considerations

Under the Wimp desktop:
- Use `Wimp_SetMode` (reason code in Wimp_Initialise) rather than calling `OS_ScreenMode` directly.
- The Wimp manages the desktop palette and redraws all windows after a mode change.
- Direct mode changes via VDU 22 or `OS_ScreenMode 0` will confuse the Wimp.
