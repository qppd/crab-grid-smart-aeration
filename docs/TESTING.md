# Testing and Validation Plan

**Project:** Solar Automated LoRa-Integrated Mud Crab Aeration Network with Crab Grid Observation

A gate-based test plan: no gate advances until the previous one passes. **Never test pumps or release the float in a pond with crabs stocked**, and never power a radio node without its antenna.

## 1. Test Gates

| Gate | Scope | Exit criteria |
|------|-------|---------------|
| G1 — Bench (12 V supply) | Rails, sensors, relays, LoRa link, control logic | All PWR/SEN/ACT/LNK/CTL tests pass |
| G2 — Integrated bench | Full enclosure + pumps in buckets + bridge + dashboard | All NET tests pass |
| G3 — Field pilot (no crabs) | 7-day soak in the pond | No unexplained downtime; calibration holds |
| G4 — Stocked deployment | Thesis demo scenarios | First-aid scenarios repeatable |

## 2. Power (PWR)

| ID | Procedure | Pass criteria |
|----|-----------|---------------|
| PWR-01 | Measure 12 V, 5 V, 3.3 V rails under pump-start load | 12 V ±5%, 5 V ±5%, 3.3 V ±5%; no brownout reset |
| PWR-02 | Press the shore-end RCD test button | RCD trips; reset restores supply |
| PWR-03 | Measure each pump's running and inrush current | Within relay 10 A and branch fuse ratings; record values for fuse sizing |
| PWR-04 | Kill AC mid-run | On restoration, node boots to AUTO with aeration ON |

## 3. Sensors (SEN)

| ID | Procedure | Pass criteria |
|----|-----------|---------------|
| SEN-01 | DS18B20 pair vs reference thermometer | Within ±0.5 °C; devices found on the shared bus |
| SEN-02 | pH two-point calibration then read buffers 4.01 / 6.86 | Reads ±0.05 pH after calibration |
| SEN-03 | EC two-point with 1.413 and 12.88 mS/cm standards | Within ±2% of standard; 12.88 mS/cm is a conductivity standard, **not** 35 ppt seawater |
| SEN-04 | DO single-point (water-saturated air) then vs handheld meter | Within ±0.3 ppm of the handheld reading |
| SEN-05 | Read ADC1 pins with node WiFi off, then (on a spare node) WiFi on | Readings unchanged — proves the ADC1-only analog design |
| SEN-06 | Unplug each probe in turn | Sensor-fault bit set in the uplink within one heartbeat |

## 4. Actuators (ACT)

| ID | Procedure | Pass criteria |
|----|-----------|---------------|
| ACT-01 | Command each relay channel from MANUAL | Relay clicks; correct pump runs |
| ACT-02 | Measure delivered airflow per cage branch (jug + stopwatch or flow meter) | 0.6–0.8 L/min per cage after regulation (4.8–6.4 L/min total for 320 L) |
| ACT-03 | Run bilge pump against a valved cage outlet | Flow at the outlet; valve shuts fully |
| ACT-04 | Verify N+1 alternation across air pumps | Duty swaps without both pumps latching on |
| ACT-05 | Reset the node with pumps connected | No relay chatter at boot (pull-ups effective) |

## 5. LoRa Link (LNK)

| ID | Procedure | Pass criteria |
|----|-----------|---------------|
| LNK-01 | Bench P2P exchange both directions | Uplink and downlink verified by CRC; ACK returned |
| LNK-02 | Log RSSI/SNR at the pond edge | Stable telemetry; record the baseline for range checks |
| LNK-03 | Walk test to the house (target 1–3 km LOS) | Packets still arrive or bridge alarms are understood as the coverage limit |
| LNK-04 | Send an expired / wrong-CRC command | NAK returned; mode unchanged |

## 6. Control Logic (CTL)

| ID | Procedure | Pass criteria |
|----|-----------|---------------|
| CTL-01 | Boot the node fresh | AUTO selected; aeration relay ON |
| CTL-02 | Rotate the selector through AUTO/MANUAL/OFF | Decode matches [`WIRING.md`](WIRING.md) §6 table at every position |
| CTL-03 | Enter MANUAL and wait 5 minutes | Auto-revert to AUTO without any command |
| CTL-04 | Simulate DO below 3 ppm (feed the ADC a low value or use the test hook) | Aeration forced ON in any mode; emergency uplink sent immediately; alarm raised |
| CTL-05 | Command OFF via dashboard, then pull the bridge offline | After revert/link loss the float returns to AUTO with aeration ON |
| CTL-06 | Send a MANUAL aeration-OFF command while DO reads below 3 ppm | Command rejected (NAK) — interlock held |

## 7. Network End-to-End (NET)

| ID | Procedure | Pass criteria |
|----|-----------|---------------|
| NET-01 | Uplink → Firebase → dashboard | New values visible within one heartbeat (≤ 60 s) |
| NET-02 | Dashboard command → downlink → pump | Command lands on the next uplink cycle; ACK shown in the UI |
| NET-03 | Unplug the bridge | Heartbeat-loss alarm after 90 s; recovers cleanly |
| NET-04 | Compare uplink echo vs dashboard state | Mode and pump states always match the device's belief |
| NET-05 | Threshold edit on the dashboard | New value clamped to range; firmware applies it on the next downlink |

## 8. Field Scenarios (thesis demo script)

1. **Rain dilution:** add freshwater to a test bucket loop — salinity falls below `SAL_MIN`; the bilge/flush logic runs; salinity recovers in an isolated or closed-loop test volume (open-water correction is not claimed).
2. **DO depletion:** stop aeration in a test bucket until DO trends toward 3 ppm — the emergency path forces aeration and sends the emergency uplink.
3. **Link loss:** switch the bridge off — heartbeat alarm at the house; float stays in AUTO and keeps aerating.

## 9. Test Log Template

| Date | Test ID | Build/commit | Result | Notes |
|------|---------|--------------|--------|-------|
| | | | pass/fail | |
