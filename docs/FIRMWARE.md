# Firmware Guide

**Project:** Solar Automated LoRa-Integrated Mud Crab Aeration Network with Crab Grid Observation

Design reference for the three ESP32 firmware targets. The controlling principles: **fail-safe to AUTO with aeration ON**, **WiFi stays off on the float**, and the system must keep crabs alive with zero internet, zero cloud, and zero operator.

## 1. Targets and Layout

```
firmware/
├── main_controller/    # Float node: sensors, control loop, LoRa
├── house_bridge/       # Bridge node: LoRa <-> Firebase relay + alarms
├── camera_node/        # ESP32-CAM-MB: WiFi AP stream only
└── shared/
    ├── pins.h          # Pin map (verified in WIRING.md §3)
    ├── packets.h       # Frame layouts and CRC-8
    └── config.h        # Device ID, radio, thresholds, timing
```

Libraries: `RadioLib` (SX1262), `OneWire` + `DallasTemperature`, `DFRobot_EC`, `DFRobot_pH`; the bridge adds the Mobizt `Firebase Arduino Client` library for Realtime Database writes.

## 2. Control Rules (enforced in firmware)

| Rule | Behavior |
|------|----------|
| Boot, reset, or link loss | Mode = **AUTO**, aeration **ON** |
| MANUAL timeout | Auto-reverts to AUTO after **5 minutes** without a fresh command (`MANUAL_REVERT_MS`, configurable 1–30 min) |
| DO emergency | DO below 3 ppm overrides any mode, forces aeration ON, raises the local alarm, and sends an **immediate emergency uplink** |
| Heartbeat | One uplink at least every **60 s**; mode and pump state echoed in every frame |
| Command validation | Device ID + sequence + expiry + CRC-8 all checked; stale or corrupt frames are NAKed |
| OFF | All actuators off; exists for servicing only |
| Boot-safe outputs | Relay GPIOs driven HIGH (relays off) **before** pin init; external pull-ups hold relays off through reset ([`WIRING.md`](WIRING.md) §4) |

## 3. Pin Map (summary)

The full conflict verification is in [`WIRING.md`](WIRING.md) §3–4. Quick map: LoRa SPI 18/19/23, NSS 5, RST 14, DIO1 26, BUSY 17; 1-Wire 16; pH 34, EC 35, DO 36 (all ADC1); relays 25/27/33/13; selector 4 (A) and 21 (B); mode LED 22, link LED 2; spare 32 and 39; leave 0/12/15 and 1/3 unconnected.

## 4. Radio Configuration (identical on every node)

| Setting | Value |
|---------|-------|
| Frequency | 915.0 MHz (PH NTC licence-free SRD band — confirm measured EIRP and type-approval) |
| Spreading factor | SF9 (SF7–12 supported) |
| Bandwidth / coding rate | 125 kHz / 4-5 |
| Preamble | 8 |
| Sync word | 0x3444 (private; never LoRaWAN public) |
| CRC | On |
| TX power | Within the NTC EIRP limit |

## 5. Packet Protocol

All frames end in **CRC-8 (poly 0x07)** computed over the preceding bytes. Multi-byte values are little-endian, scaled ×100.

### 5.1 Uplink — float to bridge (17 bytes)

| Byte | Field | Notes |
|------|-------|-------|
| 0 | Device ID | 1 = float |
| 1–2 | Sequence number | uint16, increments per frame |
| 3 | Flags | Bits 0–1 mode (0 AUTO, 1 MANUAL, 2 OFF); bit 2 emergency (DO event); bit 3 sensor fault |
| 4 | Pump bitmap | Bit 0 air1, bit 1 air2, bit 2 bilge1, bit 3 bilge2 |
| 5–6 | DO (ppm ×100) | uint16 |
| 7–8 | pH (×100) | uint16 |
| 9–10 | EC (mS/cm ×100) | uint16 |
| 11–12 | Temperature 1 (°C ×100) | int16 |
| 13–14 | Temperature 2 (°C ×100) | int16 |
| 15 | Status/reserved | Uplink interval, uptime bucket |
| 16 | CRC-8 | Over bytes 0–15 |

### 5.2 Downlink — bridge to float (8 bytes)

| Byte | Field | Notes |
|------|-------|-------|
| 0 | Device ID | Must match |
| 1–2 | Command sequence | Echoed in the ACK |
| 3 | Mode command | 0 AUTO, 1 MANUAL, 2 OFF, 255 = no change |
| 4 | Pump override bitmap | 0xFF = no change; bits as uplink |
| 5–6 | Expiry (s) | Command invalid after this window |
| 7 | CRC-8 | Over bytes 0–6 |

### 5.3 ACK/NAK — float to bridge (4 bytes)

Byte 0 device ID; byte 1 `0x06` ACK or `0x15` NAK; byte 2 echoed command sequence (low byte); byte 3 CRC-8.

## 6. Timing

| Timer | Value | Purpose |
|-------|-------|---------|
| Heartbeat/uplink | 60 s (`HEARTBEAT_MS`) | Telemetry + command check each cycle; a future low-power build may stretch this to 5–15 min |
| Emergency uplink | Immediate | DO below 3 ppm bypasses the schedule |
| MANUAL revert | 5 min | No fresh command → AUTO |
| Command expiry | Set per command | Reject stale downlinks |
| Sensor sampling | 1 s sample / 15 s average | Smooth ADC noise before uplink |
| Heartbeat-loss alarm (bridge) | No uplink for 90 s | Local alarm + RTDB alert |

## 7. Sensor Pipeline

1. Read DS18B20 pair over 1-Wire; pH/EC/DO through the DFRobot libraries with the calibration constants from [`CALIBRATION.md`](CALIBRATION.md).
2. Apply the pH divider correction (or use ADS1115 values directly) and temperature compensation per library docs.
3. Flag a **sensor fault** when a reading is physically impossible (e.g., pH outside 0–14, DS18B20 returns −127) — the fault bit rides the next uplink.
4. Persist calibration constants in NVS/EEPROM so a reboot never loses calibration.

## 8. House Bridge Node

- Receives uplinks, verifies ID + CRC-8, writes to `devices/<id>/state` and trims `logs/<id>/<timestamp>` (schema in [`APP.md`](APP.md) §2).
- Polls `commands/<id>/queue` every uplink cycle, relays fresh commands downlink, and marks each command `acked` / `naked` / `expired`.
- Raises the **heartbeat-loss alarm** after 90 s of silence (local LED/buzzer plus an RTDB alert) — this is the single-point-of-failure watchdog for the life-support load.
- WiFi joins the home network; the float node's WiFi is never enabled.

## 9. config.h Defaults

| Key | Default | Allowed range |
|-----|---------|---------------|
| `DEVICE_ID` | 1 (float), 2 (bridge) | 1–254 |
| `RF_FREQ_MHZ` | 915.0 | fixed by NTC band |
| `RF_SF` / `RF_BW_KHZ` / `RF_CR` | 9 / 125 / 5 | identical on all nodes |
| `HEARTBEAT_MS` | 60000 | 30000–300000 |
| `MANUAL_REVERT_MS` | 300000 | 60000–1800000 |
| `DO_EMERGENCY_PPM` | 3.0 | 2.0–4.0 |
| `DO_TARGET_PPM` | 5.0 | 4.0–7.0 |
| `SAL_MIN_PPT` / `SAL_MAX_PPT` | 28 / 32 | 10–34 (min < max) |
| `PH_MIN` / `PH_MAX` | 7.5 / 8.5 | 6.5–9.0 (min < max) |
| `TEMP_MIN_C` / `TEMP_MAX_C` | 27 / 30 | 20–35 (min < max) |

Dashboard-side edits to thresholds arrive as config downlinks and are clamped to these ranges ([`APP.md`](APP.md) §6).

## 10. Camera Node (ESP32-CAM-MB)

- Uses the stock AI-Thinker OV2640 pinout; **no LoRa** — video cannot fit the LoRa link.
- Runs a WiFi **access point** (`crabcam-<id>`) joined by the farmer's phone during pond visits.
- Power: 5 V at ≥ 2 A from the LM2596S module; brownouts are the usual camera fault (see [`TROUBLESHOOTING.md`](TROUBLESHOOTING.md) §6).

## 11. Building and Flashing

1. Board: **ESP32 Dev Module**; flash 4 MB; upload 115200–921600; CPU 240 MHz.
2. Select the right COM port; hold **BOOT** if "Connecting..." never completes.
3. Flash the float node **before** wiring pumps, and confirm serial logs show sensor init and the first uplink.
4. Flash the bridge, then the camera last.
5. Version-tag every field deployment in git so a known-good firmware can be re-flashed at the pond.
