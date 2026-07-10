# Examples

Copy-paste starting points for both transports and for command transmission.
Replace the `platform_dependant_rs422_*` / `platform_dependant_canfd_*` stubs with the end platform's HAL.

---

## Minimal Example

=== "Transport A - RS422 byte-stream parser"

    Feed received bytes into the parser; it calls `on_frame` once per valid frame.

    ```c
    #include "oscp_imu.h"
    #include <stdio.h>

    static void on_frame(const oscp_frame_t *frame, void *ctx) {
        (void)ctx;
        if (frame->type == OSCP_FRAME_EULER) {
            const oscp_euler_t *e = &oscp_euler(frame);
            printf("Roll %.2f  Pitch %.2f  Yaw %.2f\n", e->roll, e->pitch, e->yaw);
        }
    }

    int main(void) {
        oscp_parser_t parser;
        oscp_parser_init(&parser, on_frame, NULL);

        uint8_t buf[256];
        size_t  n;
        while ((n = uart_read(buf, sizeof(buf))) > 0) {
            oscp_parser_feed_buf(&parser, buf, n);
        }
        return 0;
    }
    ```

    On a bare-metal target, feed single bytes from the UART ISR instead:

    ```c
    void USARTx_IRQHandler(void) {
        oscp_parser_feed(&g_parser, (uint8_t)USARTx->RDR);
    }
    ```
    
=== "Transport B - CAN-FD message decoder in a single call"

    The controller hands a whole message; decode it in one call. DLC padding is
    handled - `len` may be larger than the frame's own length.

    ```c
    #include "oscp_imu.h"

    /* FDCAN RX callback */
    void on_can_message(const uint8_t *data, size_t len) {
        oscp_frame_t frame;
        if (oscp_frame_decode(data, len, &frame) != OSCP_OK) {
            return;  /* dropped */
        }

        if (frame.type == OSCP_FRAME_RAW) {
            const oscp_raw_t *r = &oscp_raw(&frame);
            /* de-pack into aligned locals before any float math */
            const float gx = r->gyro_x, gy = r->gyro_y, gz = r->gyro_z;
            const float ax = r->accel_x, ay = r->accel_y, az = r->accel_z;
            ingest_imu(gx, gy, gz, ax, ay, az);
        }
    }
    ```

---

## Handling every frame type

Branch on `frame->type` and read the matching accessor:

```c
static void on_frame(const oscp_frame_t *frame, void *ctx) {
    (void)ctx;
    switch (frame->type) {
        case OSCP_FRAME_RAW: {
            const oscp_raw_t *r = &oscp_raw(frame);
            /* ... */
            break;
        }
        case OSCP_FRAME_QUATERNION: {
            const oscp_quat_t *q = &oscp_quat(frame);
            const float w = q->w, x = q->x, y = q->y, z = q->z;
            /* ... */
            break;
        }
        case OSCP_FRAME_GNSS: {
            const oscp_gnss_t *g = &oscp_gnss(frame);
            const float lat = g->latitude, lon = g->longitude;
            /* ... */
            break;
        }
        case OSCP_FRAME_STARTUP: {
            const oscp_startup_t *s = &oscp_startup(frame);
            char mark[11];
            memcpy(mark, s->mark_number, sizeof(s->mark_number));
            mark[10] = '\0';   /* mark_number is NOT null-terminated */
            printf("Unit %s #%u  fw %u.%u.%u\n", mark, s->unit_number,
                   s->sw_major_ver, s->sw_minor_ver, s->sw_patch_ver);
            break;
        }
        /* command responses (RS422): */
        case OSCP_FRAME_CMD_SUCCESS:  puts("CMD OK");        break;
        case OSCP_FRAME_CMD_FAILED:   puts("CMD FAILED");    break;
        case OSCP_FRAME_CMD_UNKNOWN:  puts("CMD UNKNOWN");   break;
        case OSCP_FRAME_CMD_NOT_IMPL: puts("CMD NOT IMPL");  break;
        default: break;
    }
}
```

---

## Sending commands

Commands are encoded into a caller-owned buffer; then caller transmits bytes.

=== "Transport A - RS422 byte-stream parser"

    ```c
    uint8_t cmd[32];
    size_t  len = 0;

    /* RS422: COBS-encoded, 0x00-delimited */
    if (oscp_cmd_config(cmd, sizeof(cmd), &len, OSCP_TRANSPORT_RS422) == OSCP_OK) {
        /* Transmit as-is */
        platform_dependant_rs422_write(cmd, len);
        /* Good reliability habit to add a slight delay between commands */
        platform_dependant_delay_ms(10);
    }
    ```

=== "Transport B - CAN-FD message decoder in a single call"

    ```c
    uint8_t cmd[32];
    size_t  len = 0;

    /* CAN-FD: raw ASCII payload; you set the command identifier and DLC */
    if (oscp_cmd_enable_oft(cmd, sizeof(cmd), &len, OSCP_FRAME_SEL_RAW, OSCP_TRANSPORT_CANFD) == OSCP_OK) {
        /* Require 0x6FC CAN Command ID and DLC padding */
        platform_dependant_canfd_write(0x6FC, cmd, len);
        /* Good reliability habit to add a slight delay between commands */
        platform_dependant_delay_ms(10);
    }
    ```

---

### A complete configuration sequence (RS422)

```c
static void send_cmd(const uint8_t *b, size_t n) { 
    platform_dependant_rs422_write(b, n); 
    platform_dependant_delay_ms(10);
}

void configure_unit(void) {
    uint8_t cmd[32];
    size_t  len;

    /* 1. enter CONFIG from OPERATING */
    oscp_cmd_config(cmd, sizeof(cmd), &len, OSCP_TRANSPORT_RS422);
    send_cmd(cmd, len);

    /* 2. apply settings */
    oscp_cmd_om (cmd, sizeof(cmd), &len, OSCP_OM_MEDIUM, OSCP_TRANSPORT_RS422); 
    send_cmd(cmd, len);

    oscp_cmd_drg(cmd, sizeof(cmd), &len, OSCP_GYRO_DR_125DPS, OSCP_TRANSPORT_RS422);
    send_cmd(cmd, len);

    oscp_cmd_dra(cmd, sizeof(cmd), &len, OSCP_ACCEL_DR_2G, OSCP_TRANSPORT_RS422);
    send_cmd(cmd, len);

    oscp_cmd_enable_oft(cmd, sizeof(cmd), &len, OSCP_FRAME_SEL_EULER, OSCP_TRANSPORT_RS422);
    send_cmd(cmd, len);

    /* 3. return to OPERATING */
    oscp_cmd_exit(cmd, sizeof(cmd), &len, OSCP_TRANSPORT_RS422);
    send_cmd(cmd, len);
}
```

---

## Providing Context (RS422)

The `ctx` pointer handed to `oscp_parser_init` is passed back to the callback
unchanged on every invocation. It can be used to reach an application state
without globals - passing a pointer to a struct, then recovered inside the
callback. This is also what keeps multiple parser instances independent.

```c
#include "oscp_imu.h"
#include <stdio.h>

/* Application state shared with the callback through ctx. */
typedef struct {
    float    roll, pitch, yaw;   /* latest attitude */
    uint32_t euler_frames;       /* how many Euler frames we've seen */
} imu_app_t;

static void on_frame(const oscp_frame_t *frame, void *ctx) {
    imu_app_t *app = (imu_app_t *)ctx;

    if (frame->type == OSCP_FRAME_EULER) {
        const oscp_euler_t *e = &oscp_euler(frame);
        app->roll  = e->roll;
        app->pitch = e->pitch;
        app->yaw   = e->yaw;
        app->euler_frames++;
    }
}

int main(void) {
    imu_app_t app = {0};

    oscp_parser_t parser;
    oscp_parser_init(&parser, on_frame, &app); /* &app reaches every callback */

    uint8_t buf[256];
    size_t  n;
    while ((n = uart_read(buf, sizeof(buf))) > 0) {
        oscp_parser_feed_buf(&parser, buf, n);
    }

    printf("Got %u Euler frames; last yaw = %.2f\n", app.euler_frames, app.yaw);
    return 0;
}
```

!!! warning "`ctx` must outlive the parser"
    The parser stores the ctx pointer; it does not copy what it points to.
    Make sure the pointed-to object stays alive for as long as the parser is fed. 
    A stack local like `app` above is fine because it lives for all of
    `main`, but don't hand it the address of something that goes out of scope.

---

## Monitoring link health

The parser tallies every dropped frame. Poll the stats periodically to watch
line quality:

```c
const oscp_stats_t *s = oscp_parser_stats(&parser);
printf("ok=%u  framing=%u  crc=%u  cobs=%u  overflow=%u\n",
       s->frames_ok, s->framing_errors, s->crc_errors,
       s->cobs_errors, s->overflows);

oscp_parser_stats_reset(&parser);   /* zero the counters for the next window */
```

Rising `crc_errors` or `cobs_errors` usually points at a noisy line or a baud
mismatch; `overflows` means bytes are arriving faster than they're drained.
