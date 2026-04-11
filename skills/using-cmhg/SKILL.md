---
name: using-cmhg
description: Describes the CMHG file format and CMunge usage. Use when CMHG files are required for RISC OS modules.
license: MIT
---

# CMunge Skill Summary

## Purpose
CMunge is an enhanced, free alternative to the Acorn/Castle C Module Header Generator (CMHG). It generates ARM assembly "veneers" that allow RISC OS relocatable modules to be written in C or C++. It supports 26-bit, 32-bit, and 64-bit ARM architectures.

## Command Line Usage
`CMunge [options] <infile>`

Key options:
- `-o <file>`: Output object (AOF) file.
- `-s <file>`: Output assembly file.
- `-d <file>`: Output C header file.
- `-26bit`: Target legacy 26-bit ARM (default).
- `-32bit`: Target 32-bit compatible ARM.
- `-64bit`: Target 64-bit (AArch64) ARM for GCC.
- `-p` / `-px`: Pre-process input file (standard / extended).
- `-throwback`: Enable error reporting to editors.
- `-zoslib` / `-zoslibpath`: Use OSLib header styles.

## CMHG/CMunge Directives Summary
- `title-string`: Internal module name.
- `help-string`: Descriptive name and version.
- `initialisation-code`: C function called on load.
- `finalisation-code`: C function called on unload.
- `module-is-initialised-early`: (CMunge) Requests early initialization for ROM.
- `service-call-handler`: C function for service calls.
- `swi-chunk-base-number`: Base SWI number.
- `swi-handler-code`: Central SWI handler C function.
- `swi-decoding-table`: Lists SWI names and optional specific handlers.
- `command-keyword-table`: Defines `*Commands` and their handlers.
- `generic-veneers`: General purpose C wrappers.
- `vector-handlers`: Wrappers for claiming RISC OS vectors.
- `event-handler`: Wrappers for RISC OS events.
- `error-base` (CMunge): Base error number for the module.
- `error-identifiers` (CMunge): Automatically generates error definitions.
- `vector-traps` (CMunge): Advanced vector chaining handlers — see [references/vector-traps.md](references/vector-traps.md).
- `library-initialisation-code` / `library-enter-code`: Override C library entry symbols.


## References

For more information on the CMHG file format, see [references/cmhg-syntax.md](references/cmhg-syntax.md).

For detailed information on `vector-traps` including a worked example, see [references/vector-traps.md](references/vector-traps.md).
