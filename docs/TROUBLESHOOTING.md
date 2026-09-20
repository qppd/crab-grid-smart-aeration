# Troubleshooting Guide

**Project:** Solar Automated LoRa-Integrated Mud Crab Aeration Network with Crab Grid Observation

Symptom-first fault isolation. Work top to bottom within a section; stop and cut power at the breaker for anything wet near 220 V AC.

## 1. Power and Electrical

| Symptom | Likely cause | Fix |
|---------|--------------|-----|
| RCD trips repeatedly | Moisture in a connector or damaged insulation | Dry out, re-terminate with IP67/IP68 parts, no submerged joints; electrician to verify |
| 12 V rail sags when pumps start | Undersized conductor or weak PSU | Measure inrush ([`TESTING.md`](TESTING.md) PWR-03); upsize conductor/fuse coordination |
| Node reboots when both air pumps start | Voltage dip below brownout | Add branch fuses/decoupling; confirm PSU 30 A headroom |
| ESP32 regulator hot from 12 V VIN | Onboard LDO dissipation | Feed the 5 V pin from a small 12 V→5 V buck instead |

## 2. Boot and Flashing

| Symptom | Likely cause | Fix |
|---------|--------------|-----|
| Boot loop or no boot | GPIO 12 pulled HIGH at reset (strap fault) | Keep GPIO 12 unconnected per the pin map ([`WIRING.md`](WIRING.md) §3) |
| "Connecting..." during flash | Board not in download mode | Hold **BOOT** while pressing RESET; check COM port and CH340/CP210x driver |
| Wrong COM port / garbled serial | Driver or baud mismatch | Reinstall driver; set monitor to 115200 |
| Board detected but upload fails | Wrong board profile | Select **ESP32 Dev Module**, 4 MB flash |

## 3. LoRa Link

| Symptom | Likely cause | Fix |
|---------|--------------|-----|
| No packets at all | Antenna missing; freq/SF/BW/CR/sync mismatch; DIO1/BUSY miswired | Verify antenna first, then compare `config.h` on both nodes, then wiring 18/19/23/5/14/26/17 |
| CRC errors / garbage | Mismatched sync word or bandwidth | Make radio settings identical ([`FIRMWARE.md`](FIRMWARE.md) §4) |
| Intermittent reception at distance | Marginal link budget | Log RSSI/SNR (LNK-02); raise mounting height, keep LOS, stay within the NTC EIRP limit |
| Downlink never ACKed | Float busy or frame expired | Check expiry window and that commands arrive right after an uplink |

## 4. Sensors

| Symptom | Likely cause | Fix |
|---------|--------------|-----|
| pH drifts over days | Electrode aging or dry storage | Recalibrate (2-point); store in KCl solution only ([`CALIBRATION.md`](CALIBRATION.md) §2) |
| pH reading clipped near one rail | Divider wrong or 5 V board into 3.3 V ADC | Verify the 10 kΩ/18 kΩ divider or move to ADS1115 |
| EC wildly out of range | Probe not in solution during boot; calibration lost | Recalibrate with 1.413/12.88 mS/cm standards; remember 12.88 mS/cm is not 35 ppt |
| DO stuck or reads low | Membrane fouled, bubbles under cap, no flow | Service membrane/electrolyte; provide flow across the membrane |
| DS18B20 reads −127 | Wiring fault or missing 4.7 kΩ pull-up | Check the bus and pull-up on GPIO 16 |
| Analog values change when WiFi toggles | Analog was moved to an ADC2 pin | Keep analog on 34/35/36 (ADC1) only |

## 5. Actuators

| Symptom | Likely cause | Fix |
|---------|--------------|-----|
| Relay never clicks | GPIO not driven LOW (low-level trigger) or missing ground | Check pin map 25/27/33/13 and common ground |
| Pumps chatter at boot | Missing external pull-ups on relay inputs | Fit 10 kΩ pull-ups to 3.3 V ([`WIRING.md`](WIRING.md) §4) |
| Air pump runs, no bubbles | Air line flooded or stone clogged | Drain the line (drip loop), clean or replace the stone; re-measure branch flow (ACT-02) |
| Water header no flow | Valve closed or line air-locked | Open the per-cage valve; prime the line |
| Both air pumps latched ON | Alternation logic fault | Check the N+1 duty code; verify with ACT-04 |

## 6. Camera Node

| Symptom | Likely cause | Fix |
|---------|--------------|-----|
| Random reboots / brownout message | Insufficient 5 V current | Feed ≥ 2 A from the LM2596S module; shorten the USB lead |
| Phone cannot join | AP not up or wrong SSID | Look for `crabcam-<id>`, reflash if absent; note the camera is WiFi-only by design |
| Stream stalls | Weak phone-to-AP link at range | Stay near the float during pond visits; this is a documented limitation |

## 7. Dashboard / Firebase

| Symptom | Likely cause | Fix |
|---------|--------------|-----|
| Permission denied in the UI | Not signed in, or RTDB rules too tight | Sign in with the farmer account; keep the Auth-only rules from [`SETUP.md`](SETUP.md) §5 |
| Bridge not writing to RTDB | Wrong database URL or credentials | Re-check the bridge `config.h` and the service-account values |
| Vercel build fails on private key | `FIREBASE_PRIVATE_KEY` newlines lost | Re-enter the key with `\n` escapes intact |
| Dashboard shows stale data | Bridge offline or heartbeat loss | The bridge should already be alarming (90 s watchdog); restore bridge power/link |

## 8. Escalation and Safety

1. Anything wet near 220 V: **cut the breaker**, do not re-energize until the electrician clears it.
2. Keep the bench log from [`TESTING.md`](TESTING.md) §9 — fault patterns across dates are diagnostic gold.
3. If the DO probe is out of service, switch to scheduled aeration as the documented fallback until recalibration ([`CALIBRATION.md`](CALIBRATION.md) §4).
