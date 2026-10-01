# OSCP IMU Rust Library

`oscp-imu` is a Rust crate for communicating with OSCP MK2 IMU units. It decodes
the operating frames emitted by the unit and encodes the command set for both
supported transports: **RS422** (byte stream) and **CAN-FD** (message based).

:material-github: [https://github.com/OSCPS/oscp-imu-rs](https://github.com/OSCPS/oscp-imu-rs){:target="_blank"}

---

## Build & install

The crate is distributed as a Git dependency. It is not yet published on crates.io.

### Dependencies

| Crate | Source | Role |
|-------|--------|------|
| `oscp-imu` | `https://github.com/OSCPS/oscp-imu-rs.git` | Library crate |
| `cobs` | crates.io, resolved by Cargo | COBS encoding and decoding |
| `serde` | crates.io, resolved by Cargo, optional | `Serialize` implementations for decoded types |


### Add it to your project

1. Add the dependency to `Cargo.toml`:

    ```toml
    [dependencies]
    oscp-imu = { git = "https://github.com/OSCPS/oscp-imu-rs.git" }
    ```

2. To derive `serde::Serialize` on frames and command responses, enable the
   `serde` feature:

    ```toml
    [dependencies]
    oscp-imu = { git = "https://github.com/OSCPS/oscp-imu-rs.git", features = ["serde"] }
    ```

3. For reproducible builds, pin a commit with `rev = "<commit>"`.
4. Import items from the crate root:

```rust
use oscp_imu::{OscpParser, OscpFrame, OscpTransport};
```

!!! note "Requirements"
    Rust 1.85 or later (edition 2024). Fields are decoded with
    `from_le_bytes`, so the crate is independent of host byte order.

---

## Features

<div class="grid cards" markdown>

-   __No dynamic allocation__

    Parser state is held in a caller-owned `OscpParser` containing fixed-size
    arrays. The crate uses `core` only and does not allocate, so it is
    `no_std`-compatible.

-   __Two transports, one frame type__

    A byte-stream parser for RS422 and a stateless decode function for CAN-FD.
    Both return `OscpFrame`.

-   __Frame coverage__

    Raw, Euler, Quaternion, Rotation Matrix, GNSS, Debug 1/2 and Startup
    frames are decoded into typed structs.

-   __Command set__

    16 commands, encoded for either transport. Each argument is a typed enum.

-   __Integrity checks__

    CRC-16 on every frame. COBS framing on RS422.

-   __Error handling__

    Fallible functions return `Result`. The RS422 parser counts dropped frames
    in `OscpStats`.

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

    RS422 byte-stream parser and CAN-FD message decoder.

-   [__:material-book: Frame Reference__](frames.md)

    Operating frames, field by field, with types and units.

-   [__:material-progress-wrench: Command Reference__](commands.md)

    Commands, state classes and argument enumerations.

-   [__:material-api: API Reference__](api.md)

    Public functions, types and errors.

-   [__:material-code-block-parentheses: Examples__](examples.md)

    RS422 and CAN-FD reception, and command transmission.

</div>

---

!!! info "Revision"
    This documentation tracks **oscp-imu v0.1.0**.
