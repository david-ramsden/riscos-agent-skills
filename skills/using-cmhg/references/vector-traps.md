# Vector Traps in CMunge

Vector traps are a CMunge-specific veneer type that gives a module the ability to inspect,
modify, or replace vector calls whilst still being able to invoke the remainder of the vector
chain and observe the result. They are more powerful than `vector-handlers`, which can only
claim or pass on a call and cannot call further down the chain themselves.

---

## When to Use Vector Traps

Use `vector-traps` (rather than `vector-handlers`) when your handler needs to:

- **Pre-process** the call: inspect or modify input registers before passing the call on.
- **Post-process** the result: examine or alter the output registers returned by the rest of
  the chain.
- **Do both**: wrap the call — mutate registers going in and again on the way back.

If you only need to intercept certain calls and either claim or pass them through untouched,
`vector-handlers` is simpler and sufficient.

---

## CMHG Directive

```
vector-traps: EntryName/HandlerName
```

- `EntryName` — the assembly entry-point symbol (passed to `OS_Claim` / `OS_Release`).
- `HandlerName` — the C function that CMunge calls on each vector invocation.
- If `/HandlerName` is omitted, the handler is assumed to be `EntryName_handler`.
- Multiple pairs may appear on one directive, separated by spaces or commas.

### Example (from this project's `cmhg/modhead`)

```
vector-traps: VectorTrap_Entry/VectorTrap_Handler
```

---

## Generated Header Symbols

CMunge generates the following in the C header (e.g. `h/modhead`):

```c
/* Function type for calling the rest of the vector chain */
typedef _kernel_oserror *(*vectortrap_f)(_kernel_swi_regs *regs, void *trappw);

/* Assembly entry point — pass to OS_Claim / OS_Release, never call directly */
extern void VectorTrap_Entry(void);

/* Optimised pass-on sentinel — do NOT use to pass on a call you have examined */
#define VECTORTRAP_PASSON ((_kernel_oserror *)1)
```

The handler function prototype that you must implement:

```c
_kernel_oserror *VectorTrap_Handler(_kernel_swi_regs *r, void *pw,
                                    vectortrap_f trap, void *trappw);
```

---

## Handler Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `r` | `_kernel_swi_regs *` | Register block on entry to the vector. Modify this to change what downstream claimants see. |
| `pw` | `void *` | The module's private word, passed in at `OS_Claim` time. |
| `trap` | `vectortrap_f` | Pointer to the CMunge-generated function that calls the rest of the vector chain. |
| `trappw` | `void *` | Opaque private value required by `trap`; pass it straight through. |

---

## Return Values

| Return value | Effect |
|---|---|
| `NULL` | Claim the vector — the call does not propagate further. |
| `VECTORTRAP_PASSON` | Pass the call on unchanged without invoking `trap`. Use **only** when you have not inspected or modified the call at all. This is an optimisation; for any case where you have examined registers, call `trap` instead. |
| `trap(r, trappw)` | Propagate the call (with whatever register modifications you made) and return the chain's result to the caller. |
| error pointer | Claim the vector and return V set with R0 pointing to this error block. |

---

## Calling the Rest of the Chain

```c
_kernel_oserror *err = trap(modified_regs, trappw);
```

- Pass a `_kernel_swi_regs` block to `trap`. This may be the same `r` pointer or a local copy.
- After `trap` returns, `modified_regs` contains the output registers from the rest of the chain.
- If `trap` returns non-`NULL`, the V flag was set downstream; the return value is the error.
- You may further modify the registers before returning `NULL` or an error, allowing post-processing of
  the result.

---

## Claiming and Releasing the Vector

The entry point is used exactly like any other vector claimant. Claim in
`initialisation-code` and release in `finalisation-code`:

```c
_kernel_oserror *Mod_Init(const char *tail, int podule_base, void *pw)
{
    _swix(OS_Claim, _INR(0,2), WrchV, VectorTrap_Entry, pw);
    return NULL;
}

_kernel_oserror *Mod_Final(int fatal, int podule_base, void *pw)
{
    _swix(OS_Release, _INR(0,2), WrchV, VectorTrap_Entry, pw);
    return NULL;
}
```

- The third argument to `OS_Claim` / `OS_Release` is the private word; CMunge passes this
  to the handler as `pw`.
- The vector number comes from `riscos/Vectors.h` (e.g. `WrchV`, `ByteV`, `RDCHV`).

---

## Worked Example: Capitalising After a Vowel (WrchV)

This project demonstrates a vector trap on `WrchV` that capitalises the character following
any lowercase vowel. The trap pre-processes the output character by modifying R0 before
passing the call on, leaving the rest of the chain (including the screen driver) unaware
that anything changed.

### `cmhg/modhead` (directive only)

```
vector-traps: VectorTrap_Entry/VectorTrap_Handler
```

### `c/module`

```c
#include <ctype.h>
#include <stdbool.h>
#include "kernel.h"
#include "modhead.h"
#include "swis.h"
#include "riscos/Vectors.h"

static bool upper_next = false;

_kernel_oserror *VectorTrap_Handler(_kernel_swi_regs *r, void *pw,
                                    vectortrap_f trap, void *trappw)
{
    _kernel_oserror *err;
    _kernel_swi_regs newregs;
    char c = r->r[0];

    if (upper_next)
    {
        /* Modify registers before passing on to the rest of the chain */
        upper_next = false;
        newregs = *r;
        newregs.r[0] = toupper(c);
        err = trap(&newregs, trappw);
        return err;
    }

    /* No modification — check whether the next character should be uppercased */
    if (c == 'a' || c == 'e' || c == 'i' || c == 'o' || c == 'u')
        upper_next = true;

    return VECTORTRAP_PASSON;
}
```

**Key points from this example:**

- When `upper_next` is `true`, a **copy** of the register block is made (`newregs = *r`)
  and R0 is altered before calling `trap`. The original `r` is not modified, keeping the
  pre-call state clean.
- `VECTORTRAP_PASSON` is returned only on the path where the call is completely unexamined
  beyond reading `c` to decide whether to act; the pass-on path does not alter anything.
- The return value of `trap(...)` is passed straight back, propagating any error from the
  downstream chain to the original caller.

---

## Common Patterns

### Pre-process only (alter inputs, don't inspect result)

```c
_kernel_oserror *MyHandler(_kernel_swi_regs *r, void *pw,
                           vectortrap_f trap, void *trappw)
{
    _kernel_swi_regs modified = *r;
    modified.r[1] = transform(r->r[1]);
    return trap(&modified, trappw);
}
```

If you were doing this, you would be better off just using the `vector-handlers` entry point.

### Post-process only (inspect or alter results)

```c
_kernel_oserror *MyHandler(_kernel_swi_regs *r, void *pw,
                           vectortrap_f trap, void *trappw)
{
    _kernel_oserror *err = trap(r, trappw);
    if (err == NULL)
        r->r[0] = adjust(r->r[0]);   /* r is updated in-place by trap */
    return err;
}
```

### Wrap (pre- and post-process)

```c
_kernel_oserror *MyHandler(_kernel_swi_regs *r, void *pw,
                           vectortrap_f trap, void *trappw)
{
    _kernel_swi_regs modified;
    _kernel_oserror *err;
    modified = *r;
    modified.r[0] = pre_process(r->r[0]);
    err = trap(&modified, trappw);
    if (err == NULL)
        r->r[0] = post_process(modified.r[0]);
    return err;
}
```

Note that all variable declarations appear at the top of the function block, as required
by C89.

---

## Differences from `vector-handlers`

| Feature | `vector-handlers` | `vector-traps` |
|---|---|---|
| CMunge only | No (also CMHG) | Yes |
| Can claim the call | Yes | Yes |
| Can pass the call on | Yes (without calling further down) | Yes (via `trap`) |
| Can inspect chain result | No | Yes |
| Can modify registers for chain | Yes (but will be returned to caller) | Yes |
| Handler return type | `int` | `_kernel_oserror *` |
| Can return an error | Only with `error-capable:` | Always |
