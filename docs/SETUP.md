# Setup Guide

**Project:** Solar Automated LoRa-Integrated Mud Crab Aeration Network with Crab Grid Observation

This guide takes a builder from an empty desk to a powered, flashing, cloud-connected system. Follow it in order: software first (free), then hardware per the phased budget in [`BOM.md`](BOM.md), then first power-up on the bench.

## 1. Builder Pack Map

| File | Use it when |
|------|-------------|
| [`SETUP.md`](SETUP.md) (this file) | Installing tools, creating Firebase/Vercel accounts, first power-up |
| [`HARDWARE.md`](HARDWARE.md) | Component specifications and validation rationale |
| [`WIRING.md`](WIRING.md) | Assembling the float, wiring, and the verified ESP32 pin map |
| [`FIRMWARE.md`](FIRMWARE.md) | Firmware architecture, pin map, and packet protocol |
| [`TESTING.md`](TESTING.md) | Running the test plan and recording results |
| [`CALIBRATION.md`](CALIBRATION.md) | Calibrating pH, EC, DO, and temperature probes |
| [`TROUBLESHOOTING.md`](TROUBLESHOOTING.md) | Diagnosing power, boot, LoRa, sensor, and dashboard faults |
| [`APP.md`](APP.md) | Dashboard webapp features, schema, and safety interlocks |

## 2. Prerequisites

- Basic electronics skill: soldering, crimping, multimeter use.
- Tools: multimeter, soldering iron, wire strippers, crimpers, heat gun, screwdrivers, cable ties.
- **Safety gate:** all 220 V AC work (shore-end RCD, breaker, outdoor run) is a licensed-electrician job under the Philippine Electrical Code. Builders work only on the 12 V DC side.

## 3. Hardware Ordering

Order by phase so cash flow matches the project timeline (prices and links in [`BOM.md`](BOM.md)):

| Phase | Contents | Rough cost |
|-------|----------|-----------|
| 1 — Bench | 2 × ESP32 + E22/antenna, DS18B20, relay module, 1 bilge pump, selector | ₱2,906 |
| 2 — Dashboard | House bridge parts, EC sensor, refractometer, EC standards | ₱7,127–7,727 |
| 3 — Power | AC drop, RCD + breaker, 12 V 30 A PSU, LM2596S module, fuses, glands | ₱3,127–4,969 |
| 4 — Full set | DO, pH, camera, boxes, floats, frame, pipe grid, air pumps, salinity-test set | ₱34,850–46,400 |

## 4. Software Installation

1. Install **Arduino IDE 2.x** or **VS Code + PlatformIO** (PlatformIO recommended for multi-node builds).
2. Add the ESP32 board package. In Arduino IDE, File > Preferences > Additional Board Manager URLs:
   `https://raw.githubusercontent.com/espressif/arduino-esp32/gh-pages/package_esp32_index.json`
   then Boards Manager > install **esp32 by Espressif**.
3. Install the USB-serial driver for your devkit (**CH340** or **CP210x**) and confirm a COM port appears.
4. Install libraries (Library Manager or PlatformIO registry):

| Library | Used by | Purpose |
|---------|---------|---------|
| `RadioLib` | float node, bridge | SX1262 driver for the E22-900M22S |
| `OneWire` + `DallasTemperature` | float node | DS18B20 probes |
| `DFRobot_EC`, `DFRobot_pH` | float node | EC and pH conversion with calibration |
| `Firebase Arduino Client Library for ESP8266/ESP32` (Mobizt) | house bridge | RTDB REST/realtime writes |

5. Install **Node.js LTS** and **Git** for the dashboard work.

## 5. Firebase Project

1. Create a project at console.firebase.google.com (Analytics off is fine).
2. Build > **Realtime Database** > Create database > start in locked mode.
3. Build > **Authentication** > enable **Email/Password** and create the farmer account(s).
4. Deploy database rules (Auth-only; the bridge connects server-side with its own credentials):

```json
{
  "rules": {
    ".read": "auth != null",
    ".write": "auth != null"
  }
}
```

5. Project settings > Service accounts > **Generate new private key** — keep the JSON file out of the repository; it feeds the dashboard env vars below.

## 6. Dashboard Deployment

1. Push the repo to GitHub, then import it in **Vercel** (Hobby tier) or deploy via Firebase Hosting.
2. Set the environment variables exactly as named:

| Env var | Value |
|---------|-------|
| `FIREBASE_DATABASE_URL` | `https://<project>-default-rtdb.firebaseio.com` |
| `FIREBASE_PROJECT_ID` | Firebase project ID |
| `FIREBASE_CLIENT_EMAIL` | Service-account email |
| `FIREBASE_PRIVATE_KEY` | Private key from the JSON (keep the `\n` escapes) |

3. Deploy and open the site; confirm the app boots with "waiting for device data" and no permission errors.

## 7. Node Configuration

Edit `firmware/<node>/config.h` before flashing (see [`FIRMWARE.md`](FIRMWARE.md) §9):

- `DEVICE_ID`: float node = 1, house bridge = 2.
- Radio settings must be **identical on every node**: 915.0 MHz, SF9, BW 125 kHz, CR 4/5, sync word 0x3444, CRC on.
- WiFi stays **off** on the float node; only the house bridge joins the network.
- The camera node is configured separately ([`FIRMWARE.md`](FIRMWARE.md) §10): AP mode, no LoRa.

## 8. First Power-Up Checklist (bench only)

1. **Attach the 915 MHz antenna before powering any radio node.** Transmitting without an antenna destroys the PA.
2. Feed the float node from a current-limited 12 V bench supply first — never from mains on the first spin.
3. Verify rails with a multimeter: 12 V ±5% at the PSU output, 5 V ±5% at the LM2596S/buck, 3.3 V ±5% at the ESP32 3V3 pin.
4. Confirm the relay module shows all channels OFF at boot (external pull-ups fitted — see [`WIRING.md`](WIRING.md) §5).
5. Flash the float node; watch the serial log for sensor init and the first LoRa uplink.
6. Flash the bridge; confirm the uplink lands in Firebase and appears on the dashboard.
7. Press the RCD test button monthly once the AC side is commissioned by the electrician.

## 9. Next Steps

- Run the bench test matrix in [`TESTING.md`](TESTING.md).
- Calibrate every probe with [`CALIBRATION.md`](CALIBRATION.md) before trusting any reading.
- Keep [`TROUBLESHOOTING.md`](TROUBLESHOOTING.md) next to the bench.
