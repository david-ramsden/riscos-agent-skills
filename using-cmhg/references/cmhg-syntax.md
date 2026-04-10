# CMHG / CMunge File Syntax Guide

This file is a reference for agents writing or editing `.cmhg` description files processed
by CMunge. CMunge generates assembly veneers that bridge the RISC OS kernel environment to C.

---

## General Syntax Rules

- **Comments**: Start with `;` and extend to end of the physical line.
- **Directives**: A keyword (letters and hyphens) followed by `:`, then zero or more arguments.
  Keywords are case-insensitive.
- **Line continuation**: A physical line ending with `,` continues onto the next line.
- **Numbers**: Decimal by default; `&` prefix for RISC OS hex, `0x` for C hex.
  Parentheses are stripped for C header compatibility, e.g. `(&800 + 1)` is valid (CMunge only;
  only simple addition is supported).
- **Strings**: Enclosed in `"`. Adjacent strings are concatenated even across lines.
- **Arguments**: May be separated by commas or spaces. Commas are required when splitting lists
  across continuation lines.
- **64-bit mode**: CMunge defines `__riscos64` for the preprocessor when `-64bit` is used.
- **OSLib mode**: When CMunge is invoked with `-zoslib`, all occurrences of `_kernel_swi_regs`
  become `os_register_block` and `_kernel_oserror` becomes `os_error` in the generated header.

---

## Identification Directives

- **`title-string: <name>`**: The module's internal name as shown by `*Modules`. Should ideally
  be less than 16 characters and match the name in `help-string`.
- **`help-string: <name> <version> <comment>`**: Descriptive string for `*Help Modules`.
  Version should be in `d.dd` form (e.g. `1.00`).
  Example: `help-string: MyModule 1.00 (01 Jan 2026) © Me`
- **`date-string: <text>`**: (CMunge) Optional build date string.

---

## Module Behaviour Flags

- **`module-is-runnable:`**: Module has a `main()` function called in user mode when run:
  `int main(int argc, char *argv[]);`
- **`module-is-not-reentrant:`**: Prevents the module being re-entered if it is already running.
- **`module-is-c-plus-plus:`**: (CMunge) Sets up C++ global constructors via `______main` during
  initialisation. Equivalent to calling `______main()` at the start of `initialisation-code`.
- **`module-is-initialised-early:`**: (CMunge) Requests early initialisation for ROM modules.

---

## Core Handler Directives

### `initialisation-code: <C_func>`
Called when the module loads or reinitialises.
```c
_kernel_oserror *init(char *tail, int podule_base, void *pw);
```
- `tail`: arguments the module was invoked with.
- `podule_base`: podule base address, or 0.
- `pw`: module private word (use only for OS calls such as setting up veneer claims).
- Return `NULL` for success, error pointer to fail. Finalisation is not called if this fails.

### `finalisation-code: <C_func>`
Called just before the module is killed (before the C library is shut down).
```c
_kernel_oserror *final(int fatal, int podule, void *pw);
```
- `fatal`: value of R10 on entry.
- Return `NULL` to permit death, error pointer to refuse. An abort here leaves a dead module.

### `service-call-handler: <C_func> [<service_number>...]`
Called for matching service calls. Listing numbers causes the veneer to filter; omitting them
passes all service calls (not recommended).
```c
void service(int service_number, _kernel_swi_regs *r, void *pw);
```
Use `-zbase` to have CMunge define `extern int Image__RO_Base` in the header when you need a
pointer to the module base.

---

## SWI Directives

### `swi-chunk-base-number: <number>`
Base SWI number for the module's block of 64 SWIs. Do not set bit 17 (the X-bit).

### `swi-handler-code: <C_func>`
Central dispatcher for all SWIs in the chunk.
```c
_kernel_oserror *swi_handler(int number, _kernel_swi_regs *r, void *pw);
```
- `number`: offset within the block (0–63).
- Update `r->r[0]`–`r->r[9]` to change return register values.

### `swi-decoding-table: <prefix>, <name1>[/<handler1>], <name2>[/<handler2>]...`
Defines the SWI name table. The first entry is the SWI prefix. Each subsequent name may have
an optional `/HandlerFunc` to dispatch that SWI to a specific C function instead of the central
handler. The per-SWI handler has the same prototype as `swi-handler-code`.

### `swi-decoding-code: <func>` or `swi-decoding-code: <name2num>/<num2name>`

**1-function form** — `void decoder(int r[4], void *pw)`:
- If `r[0] < 0`: name-to-number. `r[1]` points to a **control-terminated** (not null-terminated)
  string. Set `r[0]` to the SWI offset if recognised; leave negative if not.
- If `r[0] >= 0`: number-to-name. `r[0]` is the SWI offset; `r[1]` points to the output buffer;
  `r[2]` is the current write offset; `r[3]` points to the byte beyond the buffer end.
  Write the name (without any terminator) from `r[2]` and update `r[2]` with bytes written.

**2-function form**:
```c
int name2num(const char *string, void *pw);      /* control-terminated input */
int num2name(int n, char *buf, int off, int end, void *pw);
```
- `name2num`: return the SWI offset (0–63), or negative if unrecognised.
- `num2name`: write the unterminated name into `buf` starting at `off`; return updated `off`.
  If no name exists for `n`, return `off` unchanged.

---

## Command Table

### `command-keyword-table: <default_handler> <commands>...`

The default handler may be `-` if every command has its own `handler:` or `no-handler:`.
At least one command description must be provided.

```c
_kernel_oserror *command(const char *arg_string, int argc, int number, void *pw);
```
- `number`: zero-based index of the command within the table.

Commands are written as `Name( <options> )`. Options (all optional):

| Option | Description |
|--------|-------------|
| `min-args: <n>` | Minimum argument count (0–255, default 0). |
| `max-args: <n>` | Maximum argument count (0–255, default 0). |
| `gstrans-map: <bits>` | Bitmap of arguments to GSTranslate (default 0). |
| `fs-command:` | Marks as a filing system command. |
| `international:` | Help/syntax messages are internationalisation tokens. |
| `add-syntax:` | Appends `invalid-syntax` to the help text. |
| `configure:` | Command is a `*Configure`/`*Status` command. |
| `status:` | Alias for `configure:`. |
| `help:` | (Unsupported in current CMunge.) |
| `invalid-syntax: <string>` | Syntax error message or token. |
| `help-text: <string>` | Help message or token. |
| `handler: <C_func>` | (CMunge) Specific C function for this command. |
| `no-handler:` | (CMunge) Help-only command; no code handler. |

### Special `arg_string` values (defined in generated header)

| Invocation | `arg_string` value | Action |
|---|---|---|
| `*Help` | `help_PRINT_BUFFER` (buffer pointer) | Fill buffer with zero-terminated text; return initial `arg_string`. Return `NULL` to print nothing. |
| `*Configure` (show syntax) | `arg_CONFIGURE_SYNTAX` (cast of 0) | Print syntax; return `NULL`. |
| `*Status` | `arg_STATUS` (cast of 1) | Print current status; return `NULL`. |
| `*Configure <value>` | actual argument string | Apply configuration; return `NULL`, error pointer, or one of: |

Configure special return values (defined in generated header):

| Constant | Meaning |
|----------|---------|
| `configure_BAD_OPTION` | Option not recognised. |
| `configure_NUMBER_NEEDED` | Numeric argument required but not given. |
| `configure_TOO_LARGE` | Value too large. |
| `configure_TOO_MANY_PARAMS` | Too many parameters supplied. |

---

## Veneer Directives

Veneers create assembly entry points that bridge RISC OS vectors/events to C functions.

**Format**: `EntryName[/HandlerName][(<params>)]`
- If `/HandlerName` is omitted, the handler is `EntryName_handler`.
- Multiple entries may be listed space- or comma-separated on the same directive.
- The generated header provides `extern void EntryName(void)` — pass this address to
  `OS_Claim` etc. **Never call a veneer entry point directly.**
- All vector claims using veneer addresses must be made in **SVC mode** (e.g. from
  initialisation code or a SWI handler).

### `generic-veneers: Entry[/Handler][(params)]...`
General callbacks, e.g. from `OS_AddCallBack`.
```c
int handler(_kernel_swi_regs *r, void *pw);
```
Parameters:

| Parameter | Description |
|-----------|-------------|
| `private-word: <reg>` | Register supplying the private word on entry (default `r12`). R12 is not preserved unless this is specified. Specify `private-word: r12` explicitly to force R12 preservation without loading from it. |
| `carry-capable:` | Handler may return `VENEER_SETCARRY` to set the carry flag on return. |

Return `NULL`, an error pointer, or `VENEER_SETCARRY` (if `carry-capable:` set).

### `vector-handlers: Entry[/Handler][(params)]...`
For claiming RISC OS vectors via `OS_Claim`.
```c
int handler(_kernel_swi_regs *r, void *pw);
```
- `r->r[0]`–`r->r[9]` on entry; update to change return values.
- Return `VECTOR_CLAIM` or `VECTOR_PASSON`.

Parameter:

| Parameter | Description |
|-----------|-------------|
| `error-capable:` | Handler may return `VECTOR_ERROR(err)` to set the V flag, claim the vector, and return `err` in R0. If `err` is `NULL`, equivalent to `VECTOR_CLAIM`. |

### `irq-handlers: Entry[/Handler]...`
Obsolete alias for `vector-handlers:`. Use `vector-handlers:` in new code.

### `event-handler: Entry[/Handler] [<event_number>...]`
Veneer for `EventV`. Only one entry/handler pair per directive (repeat the directive for
multiple handlers). Listing event numbers enables fast accept/reject filtering; strongly
recommended.
```c
int handler(_kernel_swi_regs *r, void *pw);
```
Return 0 to pass the event on, non-zero to claim it.

### `vector-traps: Entry[/Handler]...`
(CMunge) Advanced vector chaining. Provides access to the remainder of the vector chain.
```c
_kernel_oserror *handler(_kernel_swi_regs *r, void *pw, vectortrap_f trap, void *trappw);
```
Call `trap(r2, trappw)` to invoke the remainder of the chain; `r2` (same or different block)
is updated with return values.

| Return value | Effect |
|---|---|
| `NULL` | Claim the vector. |
| `VECTORTRAP_PASSON` | Pass on (equivalent to `return trap(r, trappw)` but cheaper on stack). |
| error pointer | Claim and return error. |

---

## Library Entry Overrides

- **`library-initialisation-code: <symbol>`**: Replaces `_clib_initialisemodule`. Called before
  the C environment exists, so must be written in assembly and must call
  `_clib_initialisemodule` before returning.
- **`library-enter-code: <symbol>`**: Replaces `_clib_entermodule`.

---

## Internationalisation

- **`international-help-file: <pathname>`**: Embeds a literal Messages file pathname in the
  module header. Any command `help-text` or `invalid-syntax` tagged as `international:` will
  be looked up in this file when the OS supports it.

---

## Error Directives (CMunge)

- **`error-chunk-base-number: <number>`** (alias: **`error-base:`**): Base error number for the
  module's error block. Exported as `ERROR_BASE` in the C header. Allocated in blocks of 256;
  CMunge accepts any alignment for sub-block reuse.
- **`error-identifiers: <id>([<number>,] <string>)...`**: Defines named error constants.
  Numbers are optional and auto-allocated from the base if omitted. Each identifier is
  exported in the generated header as a `_kernel_oserror *` via a macro (so errors are not
  copied to workspace).

---

## Generated Header Symbols

The following are always defined in the generated C header:

| Symbol | Value |
|--------|-------|
| `CMUNGE_VERSION` | CMunge version that produced the file. |
| `CMHG_VERSION` | `531` (closest equivalent CMHG version, for compatibility). |
| `Module_VersionString` | Full version string (CMunge); version number only (CMHG). |
| `Image__RO_Base` | Base address of the module image. Only present when `-zbase` is used. |
