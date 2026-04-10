# ColourTrans: Colour Conversion

SWIs for converting colours between RGB and other colour spaces (CIE XYZ, HSV, CMYK), and for device-to-standard colour mapping via calibration tables.

All floating-point values are passed as **16.16 fixed-point** integers unless stated otherwise (i.e. a value of 1.0 is represented as &10000).

---

## RGB ↔ CIE XYZ

### ColourTrans_ConvertRGBToCIE — SWI &40755

Converts an RGB colour to CIE XYZ tristimulus values.

**Entry:**
- R0 = red (16.16 fixed-point, range 0–1)
- R1 = green (16.16 fixed-point, range 0–1)
- R2 = blue (16.16 fixed-point, range 0–1)

**Exit:**
- R0 = CIE X
- R1 = CIE Y
- R2 = CIE Z

---

### ColourTrans_ConvertCIEToRGB — SWI &40756

Converts CIE XYZ tristimulus values back to RGB.

**Entry:**
- R0 = CIE X (16.16 fixed-point)
- R1 = CIE Y
- R2 = CIE Z

**Exit:**
- R0 = red (16.16 fixed-point, range 0–1)
- R1 = green
- R2 = blue

---

## RGB ↔ HSV

### ColourTrans_ConvertRGBToHSV — SWI &40758

Converts an RGB colour to hue, saturation, and value.

**Entry:**
- R0 = red (16.16 fixed-point, 0–1)
- R1 = green
- R2 = blue

**Exit:**
- R0 = hue in degrees (16.16 fixed-point, 0–360)
- R1 = saturation (16.16 fixed-point, 0–1)
- R2 = value (16.16 fixed-point, 0–1)

---

### ColourTrans_ConvertHSVToRGB — SWI &40759

Converts HSV back to RGB.

**Entry:**
- R0 = hue in degrees (16.16 fixed-point, 0–360)
- R1 = saturation (16.16 fixed-point, 0–1)
- R2 = value (16.16 fixed-point, 0–1)

**Exit:**
- R0 = red (16.16 fixed-point, 0–1)
- R1 = green
- R2 = blue

---

## RGB ↔ CMYK

### ColourTrans_ConvertRGBToCMYK — SWI &4075A

Converts an RGB colour to the CMYK model used in print separations.

**Entry:**
- R0 = red (16.16 fixed-point, 0–1)
- R1 = green
- R2 = blue

**Exit:**
- R0 = cyan (16.16 fixed-point, 0–1)
- R1 = magenta
- R2 = yellow
- R3 = key (black)

---

### ColourTrans_ConvertCMYKToRGB — SWI &4075B

Converts CMYK back to RGB.

**Entry:**
- R0 = cyan (16.16 fixed-point, 0–1)
- R1 = magenta
- R2 = yellow
- R3 = key (black)

**Exit:**
- R0 = red (16.16 fixed-point, 0–1)
- R1 = green
- R2 = blue

---

## Device Colour Mapping (Calibration)

These SWIs use a calibration table to map between device-specific colour values and standard (calibrated) colour values, allowing consistent colour reproduction across different hardware.

### ColourTrans_ConvertDeviceColour — SWI &40753

Maps a single device colour to its standard equivalent via a calibration table.

**Entry:**
- R1 = device colour (`&BBGGRR00`)
- R3 = 0 (use current screen calibration) or pointer to a calibration table

**Exit:**
- R2 = standard colour (`&BBGGRR00`)

---

### ColourTrans_ConvertDevicePalette — SWI &40754

Batch-converts an array of device palette entries to standard colours.

**Entry:**
- R0 = number of colours to convert
- R1 = pointer to source device palette (array of `&BBGGRR00` words)
- R2 = pointer to destination buffer (receives standard `&BBGGRR00` words)
- R3 = 0 (current calibration) or pointer to a calibration table

**Exit:** R0–R3 preserved

---

## Calibration Management

### ColourTrans_SetCalibration — SWI &40751

Loads a new calibration table for the screen, enabling colour-matched output.

**Entry:** R0 = pointer to calibration table

**Exit:** (nothing)

**Notes:** Not available in RISC OS 2.

---

### ColourTrans_ReadCalibration — SWI &40752

Reads the current screen calibration table.

**Entry:** R0 = pointer to buffer (0 = query required buffer size)

**Exit:** R0 preserved; R1 = table size in bytes (when R0 was 0 on entry)

---

### ColourTrans_WriteCalibrationToFile — SWI &40757

Writes the current calibration as `*ColourTransMap` / `*ColourTransMapSize` star commands to an open file, for use in boot configuration.

**Entry:**
- R0 = flags (bit 0 set: force write even if calibration is identity)
- R1 = file handle (from OS_Find)

**Exit:** R0 corrupted

---

## Fixed-Point Format

| Value | 16.16 representation |
|-------|----------------------|
| 0.0 | &00000000 |
| 0.5 | &00008000 |
| 1.0 | &00010000 |
| 360.0 | &01680000 |
