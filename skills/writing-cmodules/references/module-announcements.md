# Module Announcement Pattern

This document describes the standard pattern for RISC OS "manager" modules to announce their presence and impending removal to other modules in the system.

## Overview

Manager modules provide services to other modules. To ensure that clients can discover the manager when it starts, and clean up when it is removed, the manager should:
1.  **Announce Startup:** After successful initialisation, the module should use a callback to announce it has started.
2.  **Announce Death:** Immediately upon entering the finalisation handler, the module should announce it is dying, *before* it begins cleaning up any resources.

## Implementation Details

### 1. CMHG Configuration

The module requires an initialisation routine, a finalisation routine, and a generic veneer for the startup callback.

```cmhg
initialisation-code: Mod_Init
finalisation-code:   Mod_Final

generic-veneers: StartupCallback_Entry/StartupCallback_Handler
```

### 2. Service Call Definitions

Service calls should be defined in a header file (e.g., `h/ModName`).

```c
#define Service_MyModuleStarted (0x810C0)
#define Service_MyModuleDying   (0x810C1)
```

### 3. The Announcement Function

A common `announce` function handles both starting and dying announcements.

```c
void announce(int started)
{
    if (started)
    {
        /* Usually includes an API version in R0 for the Started announcement */
        _swix(OS_ServiceCall, _INR(0, 1),
                              API_VERSION, Service_MyModuleStarted);
    }
    else
    {
        /* Dying announcement allows clients to deregister before resources are gone */
        _swix(OS_ServiceCall, _IN(1),
                              Service_MyModuleDying);
    }
}
```

### 4. Initialisation (`Mod_Init`)

Initialisation performs setup and then schedules a callback to announce that the module is ready.

```c
_kernel_oserror *Mod_Init(const char *tail, int podule_base, void *pw)
{
    /* 1. Initialise internal systems */
    if (my_systems_init())
        return err_InitFailed;

    /* 2. Schedule the 'Started' announcement */
    _swix(OS_AddCallBack, _INR(0, 1), StartupCallback_Entry, pw);

    return NULL;
}
```

### 5. Startup Callback Handler

The callback handler simply triggers the announcement.

```c
_kernel_oserror *StartupCallback_Handler(_kernel_swi_regs *r, void *pw)
{
    announce(1);
    return NULL;
}
```

### 6. Finalisation (`Mod_Final`)

Finalisation must remove any pending startup callback and announce the module is dying *before* tearing down systems.

```c
_kernel_oserror *Mod_Final(int fatal, int podule_base, void *pw)
{
    /* 1. Remove the pending callback (ignoring errors) 
     *    This ensures we don't announce 'Started' if we're killed immediately.
     */
    _swix(OS_RemoveCallBack, _INR(0, 1), StartupCallback_Entry, pw);

    /* 2. Announce we are dying BEFORE shutting down systems.
     *    This allows clients to call our SWIs to deregister/cleanup.
     */
    announce(0);

    /* 3. Shut down internal systems */
    my_systems_final();

    return NULL;
}
```

## Why this pattern?

*   **Async Startup:** Announcing via callback ensures the module is fully active and capable of responding to SWIs by the time clients receive the `Started` service call.
*   **Safe Shutdown:** Announcing `Dying` before cleanup allows clients to perform their own teardown (e.g., deregistering resources) while the manager's API is still functional.
*   **Race Condition Prevention:** Removing the callback in `Mod_Final` prevents the module from announcing it has "started" if it was initialised and then immediately finalised.
