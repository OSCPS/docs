# **OSCP IMU C Library**

`oscp-imu-c` is a portable, dependency-light C library for communicating with
OSCP MK2 IMU units. It decodes every operating frame the unit emits and encodes
the full command set, over **both** transports the family supports: streamed
**RS422** and framed **CAN-FD**.

:material-github: [https://github.com/OSCPS/oscp-imu-c](https://github.com/OSCPS/oscp-imu-c){:target="_blank"}

---

## Build & install

The library is small enough to drop straight into your build - no package
manager required. Copy **four files** into your source tree and compile them alongside your own code.

### What you need

Four files:

| File | From | Role |
|------|------|------|
| `oscp_imu.h` | repository root | Library header |
| `oscp_imu.c` | repository root | Library implementation |
| `cobs.h` | `third-party/cobs-c/` | Craig McQueen COBS dependency (header) |
| `cobs.c` | `third-party/cobs-c/` | Craig McQueen COBS dependency (implementation) |


### Add it to your project

1. **Copy** the four files into your source tree (e.g. a `third-party/oscp-imu-c/` folder).
2. **Compile** `oscp_imu.c` and `cobs.c` as part of your build.
3. **Add** the folders holding `oscp_imu.h` and `cobs.h` to your include path.
4. **Include** the one public header:

```c
#include "oscp_imu.h"
```

That's the whole installation. 

!!! note "Requirements"
    A **C** compiler (GCC, Clang or MSVC) and a **little-endian** target
    (Cortex-M, x86, ARM64). Decoding is a `memcpy` into packed structs, which
    is correct only when host and wire byte orders match. 
    
    ??? warning "Big-endian targets fail to compile - by design"
        A `#error` guard in `oscp_imu.h` stops the build on big-endian hosts. To
        support one, replace the `decode_*` functions with explicit byte-swap
        helpers. Reach out to OSCP if you need this.

---

## Why this library

<div class="grid cards" markdown>

-   __Zero dynamic allocation__

    All parser state lives in a caller-owned `oscp_parser_t`. Nothing uses
    `malloc` - safe for bare-metal and RTOS targets.

-   __Two transports, one decoder__

    A byte-stream parser for RS422 and a single stateless call for CAN-FD.
    Both produce the same `oscp_frame_t`.

-   __Full frame coverage__

    Raw, Euler, Quaternion, Rotation Matrix, GNSS, Debug 1/2 and Startup
    frames are decoded into typed structs.

-   __Complete command set__

    17 commands encoded for either transport, with enumerations for every argument.

-   __Integrity built in__

    CRC-16 validation on every frame, plus COBS framing on the RS422 path.

-   __Genuinely portable__

    Builds with GCC, Clang and MSVC. Runs on Cortex-M, x86 and ARM64.

</div>

---

## Supported hardware

| Model | Description |
|-------|-------------|
| [**MK2M2**](https://www.oscp.com/products/mk2m2){:target="_blank"} | OSCP Tactical-Grade MEMS IMU |
| [**MK2E2**](https://www.oscp.com/products/mk2e2){:target="_blank"} | OSCP Tactical-Grade MEMS + Optical Gyroscope IMU |
| [**MK2Z**](https://www.oscp.com/products/mk2z){:target="_blank"} | OSCP Navigation-Grade MEMS + Optical Gyroscope IMU |

---

## Documentation map

<div class="grid cards" markdown>

-   [__:material-go-kart-track: Integration Paths__](transports.md)

    Choose between the RS422 byte-stream parser and the CAN-FD message
    decoder.

-   [__:material-book: Frame Reference__](frames.md)

    Every operating frame, field by field, with units and bytes
    layout.

-   [__:material-progress-wrench: Command Reference__](commands.md)

    All 17 commands, their state classes, and the argument enumerations.

-   [__:material-api: API Reference__](api.md)

    Every public function, type and return code in one place.

-   [__:material-code-block-parentheses: Examples__](examples.md)

    Copy-paste starting points for RS422, CAN-FD and command transmission.

</div>

---

!!! info "Version"
    This documentation tracks **oscp-imu-c v0.2.0**.
