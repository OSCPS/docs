# Examples

Reception examples for both transports, and command transmission.
The `platform_dependant_rs422_*` / `platform_dependant_canfd_*` functions are placeholders for the target platform's HAL.

---

## Minimal Example

=== "Transport A - RS422 byte-stream parser"

    Received bytes are passed to the parser, which yields one `OscpFrame` per valid frame.

    ```rust
    use oscp_imu::{OscpFrame, OscpParser};

    fn main() {
        let mut parser = OscpParser::new();

        let mut buf = [0u8; 256];
        loop {
            let n = uart_read(&mut buf);
            if n == 0 {
                break;
            }
            for frame in parser.feed_buf(&buf[..n]) {
                if let OscpFrame::Euler(e) = frame {
                    println!("Roll {:.2}  Pitch {:.2}  Yaw {:.2}", e.roll, e.pitch, e.yaw);
                }
            }
        }
    }
    ```

    On a bare-metal target, bytes can be passed one at a time, e.g. from a
    queue filled by the UART receive ISR:

    ```rust
    while let Some(byte) = rx_queue.dequeue() {
        if let Some(frame) = parser.feed(byte) {
            handle_frame(frame);
        }
    }
    ```

=== "Transport B - CAN-FD message decoder in a single call"

    Each received message is decoded with one call. `data.len()` may exceed
    the frame length because of DLC padding.

    ```rust
    use oscp_imu::{oscp_frame_decode, OscpFrame};

    /* FDCAN RX callback */
    fn on_can_message(id: u32, data: &[u8]) {
        if id == 0x6FD {
            return; /* command response, not an operating frame */
        }

        let Ok(frame) = oscp_frame_decode(data) else {
            return; /* dropped */
        };

        if let OscpFrame::Raw(r) = frame {
            ingest_imu(r.gyro_x, r.gyro_y, r.gyro_z, r.accel_x, r.accel_y, r.accel_z);
        }
    }
    ```

---

## Handling every frame type

`match` on the returned frame:

```rust
use oscp_imu::{OscpFrame, OscpFrameCmd};

fn handle_frame(frame: OscpFrame) {
    match frame {
        OscpFrame::Raw(r) => {
            let (gx, gy, gz) = (r.gyro_x, r.gyro_y, r.gyro_z);
            /* ... */
        }
        OscpFrame::Quat(q) => {
            let (w, x, y, z) = (q.w, q.x, q.y, q.z);
            /* ... */
        }
        OscpFrame::Gnss(g) => {
            let (lat, lon) = (g.latitude, g.longitude);
            /* ... */
        }
        OscpFrame::Startup(s) => {
            let mark = core::str::from_utf8(&s.mark_number)
                .unwrap_or("<invalid>")
                .trim_end_matches('\0'); /* mark_number is a byte array */
            println!("Unit {} #{}  fw {}.{}.{}", mark, s.unit_number,
                     s.sw_major_ver, s.sw_minor_ver, s.sw_patch_ver);
        }
        /* command responses (RS422): */
        OscpFrame::CmdResponse(OscpFrameCmd::Success) => println!("CMD OK"),
        OscpFrame::CmdResponse(OscpFrameCmd::Failed)  => println!("CMD FAILED"),
        OscpFrame::CmdResponse(OscpFrameCmd::Unknown) => println!("CMD UNKNOWN"),
        OscpFrame::CmdResponse(OscpFrameCmd::NotImpl) => println!("CMD NOT IMPL"),
        _ => {}
    }
}
```

---

## Sending commands

Commands are encoded into a caller-owned buffer. The caller transmits the bytes.

=== "Transport A - RS422 byte-stream parser"

    ```rust
    use oscp_imu::{oscp_cmd_config, OscpTransport};

    let mut cmd = [0u8; 32];

    /* RS422: COBS-encoded, 0x00-delimited */
    if let Ok(len) = oscp_cmd_config(&mut cmd, OscpTransport::Rs422) {
        /* Transmit as-is */
        platform_dependant_rs422_write(&cmd[..len]);
        /* Delay between consecutive commands */
        platform_dependant_delay_ms(10);
    }
    ```

=== "Transport B - CAN-FD message decoder in a single call"

    ```rust
    use oscp_imu::{oscp_cmd_enable_oft, OscpFrameSel, OscpTransport};

    let mut cmd = [0u8; 32];

    /* CAN-FD: ASCII payload; caller sets the command identifier and DLC */
    if let Ok(len) = oscp_cmd_enable_oft(&mut cmd, OscpFrameSel::Raw, OscpTransport::Canfd) {
        /* CAN command ID 0x6FC; caller applies DLC padding */
        platform_dependant_canfd_write(0x6FC, &cmd[..len]);
        /* Delay between consecutive commands */
        platform_dependant_delay_ms(10);
    }
    ```

---

### Configuration sequence (RS422)

```rust
use oscp_imu::{
    oscp_cmd_config, oscp_cmd_dra, oscp_cmd_drg, oscp_cmd_enable_oft, oscp_cmd_exit, oscp_cmd_om,
    OscpAccelDr, OscpErr, OscpFrameSel, OscpGyroDr, OscpOmSel, OscpTransport,
};

fn send_cmd(b: &[u8]) {
    platform_dependant_rs422_write(b);
    platform_dependant_delay_ms(10);
}

fn configure_unit() -> Result<(), OscpErr> {
    let mut cmd = [0u8; 32];
    let t = OscpTransport::Rs422;

    /* 1. enter CONFIG from OPERATING */
    let len = oscp_cmd_config(&mut cmd, t)?;
    send_cmd(&cmd[..len]);

    /* 2. apply settings */
    let len = oscp_cmd_om(&mut cmd, OscpOmSel::Medium, t)?;
    send_cmd(&cmd[..len]);

    let len = oscp_cmd_drg(&mut cmd, OscpGyroDr::Dps125, t)?;
    send_cmd(&cmd[..len]);

    let len = oscp_cmd_dra(&mut cmd, OscpAccelDr::G2, t)?;
    send_cmd(&cmd[..len]);

    let len = oscp_cmd_enable_oft(&mut cmd, OscpFrameSel::Euler, t)?;
    send_cmd(&cmd[..len]);

    /* 3. return to OPERATING */
    let len = oscp_cmd_exit(&mut cmd, t)?;
    send_cmd(&cmd[..len]);

    Ok(())
}
```

---

## Application state (RS422)

The parser returns frames instead of calling a callback, so there is no
context pointer. Application state is updated in the receive loop. Separate
`OscpParser` instances are independent and can be used for multiple units.

```rust
use oscp_imu::{OscpFrame, OscpParser};

/* Application state updated in the receive loop */
#[derive(Default)]
struct ImuApp {
    roll: f32,
    pitch: f32,
    yaw: f32,          /* latest attitude */
    euler_frames: u32, /* number of Euler frames received */
}

fn main() {
    let mut app = ImuApp::default();
    let mut parser = OscpParser::new();

    let mut buf = [0u8; 256];
    loop {
        let n = uart_read(&mut buf);
        if n == 0 {
            break;
        }
        for frame in parser.feed_buf(&buf[..n]) {
            if let OscpFrame::Euler(e) = frame {
                app.roll = e.roll;
                app.pitch = e.pitch;
                app.yaw = e.yaw;
                app.euler_frames += 1;
            }
        }
    }

    println!("Got {} Euler frames; last yaw = {:.2}", app.euler_frames, app.yaw);
}
```

!!! note
    Frames are returned by value and implement `Copy`. They can be stored,
    sent over a channel or moved to another thread without borrowing from the
    parser.

---

## Monitoring link health

The parser counts dropped frames by cause. Reading the counters periodically
gives an indication of link quality:

```rust
let s = parser.stats;
println!("ok={}  framing={}  crc={}  cobs={}  overflow={}",
         s.frames_ok, s.framing_errors, s.crc_errors,
         s.cobs_errors, s.overflows);

parser.stats_reset();   /* zero the counters for the next window */
```

!!! note
    `parser.stats` cannot be read while a `feed_buf` iterator exists, because
    the iterator holds a mutable borrow of the parser. Read it after the loop
    that consumes the iterator.

An increase in `crc_errors` or `cobs_errors` typically indicates line noise or
a baud rate mismatch. An increase in `overflows` indicates more than
`OSCP_FRAME_MAX_LEN` bytes received between delimiters, typically caused by a
lost delimiter byte.
