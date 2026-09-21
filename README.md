<div align="center">

# Solar Automated LoRa-Integrated Mud Crab Aeration Network with Crab Grid Observation

A house-powered, remotely monitored smart aquaculture system for mud crab fattening in mangrove areas — combining physical crab separation, continuous water quality monitoring, automated aeration and alerts, LoRa remote control, and overhead visual security.

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Platform: ESP32](https://img.shields.io/badge/Platform-ESP32-red.svg)](https://www.espressif.com/)
[![LoRa: 915 MHz P2P](https://img.shields.io/badge/LoRa-915%20MHz%20P2P-green.svg)](#electronics--hardware)
[![Power: 220V AC from House Solar](https://img.shields.io/badge/Power-220V%20AC%20from%20House%20Solar-yellow.svg)](#power-system)
[![Status: In Development](https://img.shields.io/badge/Status-In%20Development-orange.svg)]()

**Author:** [QPPD](https://www.github.com/qppd)

</div>

---

> **Where to find what:** this README = overview, architecture and timeline · [`Components.md`](docs/Components.md) = engineering specs and validation rationale · [`docs/BOM.md`](docs/BOM.md) = prices, product links, compatibility verification and power budget · [`SETUP.md`](docs/SETUP.md), [`WIRING.md`](docs/WIRING.md), [`FIRMWARE.md`](docs/FIRMWARE.md), [`TESTING.md`](docs/TESTING.md), [`CALIBRATION.md`](docs/CALIBRATION.md), [`TROUBLESHOOTING.md`](docs/TROUBLESHOOTING.md) = builder guides · [`APP.md`](docs/APP.md) = dashboard webapp specification · [`SYSTEM-ARCHITECTURE.md`](docs/SYSTEM-ARCHITECTURE.md), [`BLOCK-DIAGRAM.md`](docs/BLOCK-DIAGRAM.md), [`FLOWCHART.md`](docs/FLOWCHART.md) = diagrams (Mermaid) · [`STACKS.md`](docs/STACKS.md) = technology stack.

---

## Table of Contents

- [About the Project](#about-the-project)
- [Core Capabilities](#core-capabilities)
- [System Architecture](#system-architecture)
- [Structure & Layout](#structure--layout)
- [Electronics & Hardware](#electronics--hardware)
- [Power System](#power-system)
- [Software & Firmware](#software--firmware)
- [BOM & Budget](#bom--budget)
- [Getting Started](#getting-started)
  - [Prerequisites](#prerequisites)
  - [Hardware Setup](#hardware-setup)
  - [Firmware Installation](#firmware-installation)
  - [Dashboard Configuration](#dashboard-configuration)
- [Project Structure](#project-structure)
- [Compatibility Notes](#compatibility-notes)
- [Roadmap](#roadmap)
- [Contributing](#contributing)
- [License](#license)
- [Contact](#contact)

---

## About the Project

Mud crab (*Scylla serrata*) fattening is a vital livelihood in Southeast Asian mangrove communities, but crabs are notoriously cannibalistic — they eat each other when crowded or molting. Traditional farming relies on manual water monitoring, which is reactive rather than proactive. In these remote mangrove areas there is no utility grid and no Wi-Fi, and whatever power exists comes from small household solar systems — which makes conventional IoT solutions impractical.

This project solves these problems with a **low-power, remotely monitored aquaculture system** that:

1. Physically protects crabs using individual floating cages
2. Continuously senses water quality with multiple environmental sensors
3. Automatically triggers aeration and alerts; salinity correction is treated as a controlled test function, not a guaranteed open-water correction
4. Transmits data over **LoRa** to a bridge node at the house (where WiFi exists) for real-time remote monitoring
5. Provides visual security through an overhead camera system

The prototype is powered from the **household's existing solar installation** (220V AC, available 24/7 from the array, the inverter and a battery bank of over 600 Ah), so it needs no panels or batteries of its own — only a protected AC drop from the house to the float plus a 12V DC supply on site.

---

## Core Capabilities

| Feature | Description |
|---------|-------------|
| **Physical Protection** | Individual floating cages (horizontal crab fattening boxes) physically separate crabs to prevent cannibalism. |
| **Water Quality Sensing** | Continuous monitoring of dissolved oxygen, pH, temperature and salinity — plus a pH/temperature-derived ammonia toxicity-risk indicator. Absolute NH₃ concentration requires a TAN measurement or laboratory test. |
| **Automated First Aid** | When conditions become dangerous, the system forces aeration, raises an alarm, and logs the event. Open-water salinity correction is not assumed; brine dosing is only valid in an isolated or closed-loop test enclosure. |
| **Manual & Automatic Control** | Runs itself in **AUTO** (firmware thresholds drive the pumps) and accepts **MANUAL** or **OFF** from the dashboard webapp, with a physical selector on the box as the fail-safe. |
| **Remote Monitoring & Control** | LoRa carries sensor data and pump commands to the house bridge node, which feeds the remote dashboard — even where there is no internet or Wi-Fi at the pond. |
| **Visual Security** | Overhead camera to monitor crab molting, deter animal predators, and spot human poachers. |

---

## System Architecture

```
                            ┌─────────────────────────────────────┐
                            │      REMOTE DASHBOARD (Next.js      │
                            │         + Firebase webapp)          │
                            └──────────────┬──────────────────────┘
                                           │ Internet / WiFi
                                           │
                            ┌──────────────┴──────────────────────┐
                            │  House Bridge Node (ESP32 + E22)    │
                            │  at the house, where WiFi exists    │
                            └──────────────┬──────────────────────┘
                                           │ LoRa (915 MHz, P2P)
                            ┌──────────────┴──────────────────────┐
                            │   ESP32 DevKit + E22 (SX1262)       │
                            │       — Main Controller Node        │
                            └──┬─────┬──────┬──────┬──────┬───────┘
                               │     │      │      │      │
                  ┌────────────┘     │      │      │      └────────────┐
                  │                  │      │      │                   │
          ┌───────┴────────┐  ┌─────┴────┐ │ ┌────┴───────┐  ┌───────┴──────┐
          │  Water Sensors │  │  Relay   │ │ │  Camera    │  │  House Solar │
          │  · EC/Salinity │  │  Module  │ │ │  ESP32-CAM │  │  AC Feed     │
          │  · pH Probe    │  │  (8-Ch)  │ │ │  OV2640    │  │  · 220V 24/7 │
          │  · DS18B20 ×2  │  └────┬─────┘ │ └────────────┘  │  · RCD 30mA  │
          │  · DO Sensor   │       │       │                  │  · 12V 30A   │
          └────────────────┘  ┌────┴─────┐ │                  │    PSU       │
                              │ Actuators│ │                  └──────────────┘
                              │ · Air    │ │
                              │   Pumps  │ │
                              │ · Bilge  │ │
                              │   Pumps  │ │
                              └──────────┘ │
                                           │
                              ┌─────────────┴───────────┐
                              │   8 × Individual Crab   │
                              │   Fattening Boxes on     │
                              │   Foam Pontoon Grid      │
                              └─────────────────────────┘
```

---

## Structure & Layout

| Specification | Detail |
|---------------|--------|
| **Pond Area** | 500 sqm (modular conceptual prototype) |
| **Capacity** | 8 individual fattening boxes on the pontoon grid, one crab per box (the anti-cannibalism constraint) |
| **Boxes & frames** | Slotted plastic boxes, shaded under the frame; frame/gantry preferably UV-stabilized PVC/FRP, or marine-grade timber sealed with marine epoxy/PU and Grade 316/nylon fasteners |
| **Floaters** | Ready-made foam-filled pontoon floats (×8–12), each with a manufacturer-rated buoyancy of at least 50 kg. Final sizing uses the measured dry above-water load and load distribution; water inside slotted cages is not counted as dead load while submerged. The PVC pipes are **conduit, never flotation**: water-filled pipe weighs ~1 kg per litre (see [Components.md](docs/Components.md)). |
| **Aeration** | 2 × RESUN MPQ-03 (MPQ-903) 12V air pumps (35W, 68 L/min open-flow, 6 outlets) + air stone per cage — regulate and measure the delivered flow. The biological target is approximately 4.8–6.4 L/min total for 320 L (1.5–2 L/min per 100 L), with N+1 redundancy. |
| **Air & water routing** | Two separate circuits inside the PVC pipe grid: an air header with per-cage drop tubes and stones, and a water header fed by the bilge pumps with a valved outlet per cage. Flexible hose runs from pump to cage are an option/backup |
| **Sensor hub** | Centred on the grid for balance and symmetric cable runs; probes at mid-depth (~20–30 cm), clear of the aerator plume. The DO probe must have gentle water movement across its membrane; one probe is a single-point risk, so use a handheld meter or scheduled aeration as a fallback. |
| **Controller & Power** | Controller centred in the setup for optimal wiring. Powered by a 220V AC drop from the household solar system (24/7) → shore-end 30 mA RCD + 2-pole breaker → weatherproof feedthroughs → 12V 30A DC supply. The AC installation and conductor/fuse sizing require a licensed electrician and Philippine Electrical Code review. |

> See [`Components.md`](docs/Components.md) for the full component specifications and [`docs/BOM.md`](docs/BOM.md) for the detailed Bill of Materials with verified product links and pricing.

---

## Electronics & Hardware

| Component | Specification | Purpose |
|-----------|--------------|---------|
| **ESP32 DevKit 38-pin + EBYTE E22-900M22S (SX1262)** | ESP32-WROOM-32 + 915 MHz LoRa (SX1262, up to +22 dBm) | Main controller node — reads all sensors, runs automation logic, communicates over point-to-point LoRa on the 915 MHz ISM band. Verify measured EIRP and NTC type-approval before deployment. |
| **House Bridge Node (ESP32 + E22-900M22S)** | Same radio stack, at the house | LoRa ⇄ WiFi bridge — pushes float telemetry to the Firebase dashboard and relays dashboard commands back down over LoRa |
| **ESP32-CAM** | OV2640, WiFi/BT | Overhead camera node for visual monitoring |
| **8-Channel Relay Module** | 12V coil, optocoupler, low-level trigger | Drives aerators and salinity pumps with spare channels |
| **DFRobot EC/Salinity Sensor** | K=10, 0–100 mS/cm | Measures conductivity/salinity. Use the manufacturer/PSS-78 conversion rather than a fixed 0.66 ppt-per-mS/cm rule; 35 ppt seawater is approximately 53 mS/cm at 25 °C. |
| **pH Electrode E-201-C** | PH-4502C analog board | Monitors pH (target: 7.5–8.5 for brackish water) |
| **DS18B20 Temperature Probes** | Waterproof, stainless steel ×2 | Digital temperature sensing (1-Wire) |
| **DFRobot Dissolved Oxygen Kit** | SEN0237-A, galvanic probe — **one unit (single measurement point)** | Measures dissolved oxygen — the #1 crab killer; mounted on the centred hub with water movement across the membrane. Carry a handheld DO meter and spare membrane/electrolyte because one probe is a single-point failure. |
| **12V DC Air Pumps (RESUN MPQ-03)** | Electromagnetic, 35W, 68 L/min open-flow, 0.068 MPa, ~3.5A, 6 outlets each ×2 | Oxygenation for all 8 cages; regulate each branch to the measured biological target (about 0.6–0.8 L/min per cage for 4.8–6.4 L/min total) + N+1 redundancy (alternating duty). Huge open-flow headroom covers depth/tubing/stone losses — must not be dumped unregulated into the cages. |
| **12V Marine Bilge Pumps** | 1100 GPH, submersible ×2 | Water circulation/flushing and a controlled salinity experiment. Do not claim reliable salinity correction in open slotted cages: tidal flushing and dense brine sinking will defeat dosing unless the cage is temporarily isolated or the test is closed-loop. |

---

## Power System

| Component | Specification |
|-----------|--------------|
| **AC Source** | 220V AC drop from the household solar system (array + **>600 Ah** battery bank + inverter) — available 24/7 |
| **AC Protection** | 30 mA RCD/GFCI + 2-pole breaker at the house end; outdoor-rated earthed cable (2.5 mm²) run overhead or in conduit; IP67/IP68 weatherproof connectors, drip loops, no submerged joints, monthly RCD test, and licensed-electrician commissioning. |
| **On-Site DC Supply** | 12V 30A (360 W) switching PSU → pumps, relay module, controller; LM2596S 24V/12V → 5V USB step-down module for the camera node. Treat 30A and the branch fuses as provisional until pump nameplate current, inrush, conductor ampacity, and voltage drop are measured and signed off. |

**Night load:** the planning estimate is ~50–60 W (≈ 5 A at 12 V) for aeration, but verify the actual pump duty cycle and inverter losses before relying on the household bank. The full Power Budget is in [`docs/BOM.md`](docs/BOM.md).

> **Single point of failure:** no on-site battery, so a tripped breaker or a cut cable stops aeration. The house bridge must alarm on heartbeat loss; use a local UPS/supercapacitor only if a last-gasp packet or unattended-night runtime is required.

---

## Software & Firmware

- **Microcontroller Firmware:** ESP32 (Arduino/PlatformIO) — sensor reading, LoRa P2P communication, relay control
- **LoRa Link:** point-to-point 915 MHz (E22-900M22S / SX1262) between the float node and the house bridge node — no LoRaWAN server (TTN/ChirpStack) needed. 915 MHz is the Philippine NTC licence-free SRD band for this low-duty telemetry; confirm maximum EIRP and type-approval before deployment.
- **Dashboard:** a **Next.js + Firebase (Realtime Database + Auth)** webapp — the house bridge node writes sensor data into the Realtime Database, and dashboard commands are read back and relayed down over LoRa
- **Camera:** WiFi-based streaming from ESP32-CAM (local AP during pond visits)

### Control Modes

| Mode | Drives the pumps | When to use |
|------|------------------|-------------|
| **AUTO** (default) | Firmware thresholds — low salinity → bilge pump, low DO → aerator | Normal operation, and the fallback whenever anything else fails |
| **MANUAL** | The operator, per pump, from the webapp or the local selector | Testing, calibration, maintenance, harvest |
| **OFF** | Nobody — all actuators off | Servicing the float only; never while crabs are stocked |

Rules the firmware enforces:

- Boot, reset or link loss → **AUTO**, with aeration defaulting **ON**
- MANUAL auto-reverts to AUTO after 5 min without a command, so a forgotten switch cannot disable life-support aeration
- A DO reading below 3 ppm overrides MANUAL, forces aeration on, sends an immediate emergency uplink and raises an alarm
- The float sends a heartbeat at least every 60 seconds; the house bridge alarms on missed heartbeats
- Mode and pump state are echoed in every uplink, so the dashboard shows what the device actually believes

**Downlink packet:** device ID + sequence number + mode/pump command + expiry/ack field + CRC-8. The receiver validates the ID and CRC, applies the command only if it is fresh, and returns an ACK/NAK. Normal polling may be 5–15 minutes, but emergency sensor events bypass the schedule.

---

## BOM & Budget

Every part, price, product link and rating lives in [`docs/BOM.md`](docs/BOM.md), together with the compatibility verification and the power budget. Headline totals: **Tier 1 sensors ≈ ₱39,787–48,879**; **full sensor set ≈ ₱53,182–63,473** (per-category ranges are in the BOM).

Purchase is phased to match the project timeline — the four phases add up to the build total above (≈ **₱53,182–63,473** at item prices):

### Phase 1 — Bench Testing (~₱2,906)
2 × ESP32 DevKit + 2 × E22-900M22S/915 MHz antenna + DS18B20 + relay module + one bilge pump + the AUTO/OFF/MANUAL selector → build & test the control loop and the LoRa link on the bench (12V from a lab supply, or buy the on-site PSU now).

### Phase 2 — Remote Dashboard (~₱7,127–7,727)
House bridge node (ESP32 + E22-900M22S) + EC/salinity sensor + refractometer + calibration standard → close the remote monitoring loop.

### Phase 3 — Power to the Float (~₱3,127–4,969)
Protected 220V AC drop from the house (30 mA RCD + breaker) + 12V 30A DC supply + LM2596S camera module + fuses/glands → run the float off the household solar system. A licensed electrician must install and commission the mains side.

### Phase 4 — Full Feature Set (~₱34,850–46,400)
DO sensor + pH sensor (+ buffers) + ESP32-CAM camera + second circulation pump + fattening boxes + frame/netting + **4× round foam floats 50×90** + the PVC air/water grid + sensor hub + air pumps + misc hardware + a controlled salinity-test setup → complete system before deployment. Open-water brine correction is not a guaranteed feature.

> **₱50,000 hardware ceiling:** if the build runs over, the **camera is the first cut** — #3 plus its LM2596S module (#19) is ₱748. The DO kit (₱12–13k) is what pushes the full set past the ceiling, so it belongs in the last phase. The BOM carries the full cut order and the resulting **₱39,934–48,526** configuration; the DO probe and the aerators are never cut. The brine reserve is only retained for a controlled salinity experiment, not as proof of open-water correction.

---

## Getting Started

### Prerequisites

- [Arduino IDE](https://www.arduino.cc/en/software) or [PlatformIO](https://platformio.org/)
- ESP32 board package (esp32 by Espressif)
- `RadioLib` (LoRa/SX1262 driver for the E22-900M22S) · `OneWire` / `DallasTemperature` (DS18B20) · `DFRobot_EC` / `DFRobot_pH` (DFRobot sensors)
- 3 × ESP32 DevKit 38-pin + 3 × E22-900M22S + 915 MHz antennas (float node, house bridge, spare — see [`docs/BOM.md`](docs/BOM.md) §1)
- pH/EC calibration standards and a handheld refractometer (see [`docs/BOM.md`](docs/BOM.md) §6)

### Hardware Setup

1. **Scaffold the floating grid** on foam-filled pontoon floats, with the frame and gantry in UV-stabilized PVC/FRP or properly sealed marine-grade material; use Grade 316 stainless or nylon fasteners in brackish water.
2. **Mount the 8 fattening boxes** on the grid, one crab per box, shaded from direct sun.
3. **Position the controller node** (ESP32 + E22-900M22S) in the centre of the setup in an IP65 enclosure.
4. **Connect the sensors** to the ESP32 ADC/GPIO pins per the pin map in [`Components.md`](docs/Components.md). For the minimum pH divider use 10 kΩ series + 18 kΩ shunt (5 V → about 3.21 V); for professional-grade pH, use an ADS1115 I2C ADC and 3.3 V level shifting.
5. **Wire the relay module** to the air pumps and bilge pumps, following the WiFi/boot-safe pin map in [`Components.md`](docs/Components.md) (no analog on ADC2 pins while WiFi runs, nothing on boot-strap pins 0/12/15).
6. **Run the power drop** — 220V AC from the house (30 mA RCD + breaker at the house end, weatherproof IP67/IP68 terminations, drip loops, no submerged joints) → 12V 30A DC supply on the float → fused distribution. Have a licensed electrician verify conductor size, earthing, breaker coordination and Philippine Electrical Code compliance.
7. **Calibrate before trusting any reading** — 2-point pH (4.01 / 6.86), EC with 1.413 mS/cm and 12.88 mS/cm standards following the sensor procedure, and cross-check salinity with the refractometer. Treat 12.88 mS/cm as a conductivity standard, not as 35 ppt seawater.

### Firmware Installation

1. Create the `firmware/` directory (planned — see [Project Structure](#project-structure)) and open it in Arduino IDE or PlatformIO.
2. Install required libraries:
   - `RadioLib` (SX1262 — the E22-900M22S needs RadioLib's SX126x driver; SPI SCK 18/MISO 19/MOSI 23/NSS 5, RST 14, DIO1 26, BUSY 17)
   - `OneWire` / `DallasTemperature` (DS18B20)
   - `DFRobot_EC` / `DFRobot_pH` (DFRobot sensors)
3. Configure the radio settings identically on every node (915 MHz, SF9–12, matching bandwidth/coding rate/sync word). Record the configured output/EIRP and confirm NTC type-approval for the 915 MHz SRD band.
4. WiFi stays **off** on the float node; only the house bridge joins the network.
5. Upload the firmware to each ESP32. Implement the device-ID/sequence/CRC-8 packet, ACK/NAK, 60-second heartbeat, immediate DO emergency uplink, and 5-minute MANUAL auto-revert before any life-support test.

### Dashboard Configuration

1. **Create the back end** — a Firebase project with the **Realtime Database** (live telemetry) and Firebase Auth for the farmer accounts.
2. **Deploy the webapp** — a Next.js app on Vercel's free tier (or Firebase Hosting), with the service-account values set as env vars: `FIREBASE_DATABASE_URL`, `FIREBASE_PROJECT_ID`, `FIREBASE_CLIENT_EMAIL`, `FIREBASE_PRIVATE_KEY`.
3. **Connect the uplinks** — the house bridge node relays each float packet into the Realtime Database (`devices/<id>/state`), where the webapp's `onValue()` listeners stream it straight to the UI.
4. **Build the control panel** — live readings, pump state, the AUTO/MANUAL/OFF selector, per-pump overrides and alert thresholds; a route handler writes the command into the Realtime Database where the house bridge polls it and relays it down over LoRa.

---

## Project Structure

```
crab-grid-smart-aeration/
├── README.md              # This file — start here
├── LICENSE                # MIT License
├── .gitignore
├── docs/                   # Documentation and builder guides
│   ├── Components.md      # Engineering specs and validation rationale
│   ├── HARDWARE.md        # Hardware guide (specs, pin map, BOM mirror)
│   ├── BOM.md             # Bill of Materials — prices, product links, ratings
│   ├── SETUP.md           # Environment, accounts and first power-up
│   ├── WIRING.md          # Assembly, wiring and verified ESP32 pin map
│   ├── FIRMWARE.md        # Firmware architecture and packet protocol
│   ├── TESTING.md         # Test plan and acceptance criteria
│   ├── CALIBRATION.md     # Sensor calibration procedures
│   ├── TROUBLESHOOTING.md # Fault isolation guide
│   ├── APP.md             # Dashboard webapp specification
│   ├── SYSTEM-ARCHITECTURE.md # Thesis system architecture diagram (Mermaid)
│   ├── BLOCK-DIAGRAM.md   # Thesis hardware block diagram (Mermaid)
│   ├── FLOWCHART.md       # Thesis operational flowchart (Mermaid)
│   ├── STACKS.md          # Technology stack reference
│   └── images/            # planned
├── firmware/              # planned — ESP32 source code
│   ├── main_controller/  # planned — ESP32+E22-900M22S sensor + LoRa firmware
│   ├── house_bridge/     # planned — LoRa ⇄ WiFi relay node firmware
│   └── camera_node/      # planned — ESP32-CAM streaming firmware
└── dashboard/            # planned — Next.js + Realtime Database webapp

Items marked "planned" are not yet in the repository — create each item as its roadmap phase completes.
```

---

## Compatibility Notes

All system-level checks, with their verdicts, are in [`docs/BOM.md`](docs/BOM.md). The following gates still need action before deployment:

| Open item | Action |
|-----------|--------|
| Camera is WiFi-only (video cannot fit LoRa) | Use its own AP during pond visits, run it at the house with the bridge node, or fit an LTE camera |
| No affordable NH₃ sensor | pH + temperature can estimate the toxic fraction/risk only; absolute NH₃ requires measured TAN or a laboratory test |
| PH-4502C outputs up to 5V | Use 10 kΩ series + 18 kΩ shunt for the minimum divider, or an ADS1115 for better precision |
| No on-site battery | Alert on heartbeat loss at the house bridge; confirm the house inverter stays on 24/7; add a small 12V UPS or supercapacitor if a last-gasp packet or unattended-night runtime is required |
| Control needs internet, hosting and the bridge node | Local selector as fail-safe; MANUAL auto-reverts to AUTO after 5 minutes; emergency sensor events bypass the polling interval; ACK/NAK confirms commands |

---

## Roadmap

- [x] Concept definition and documentation
- [x] Component selection and BOM verification
- [ ] Phase 1: Bench testing with ESP32 + E22-900M22S + basic sensors
- [ ] Phase 2: House bridge node integration and remote dashboard
- [ ] Phase 3: Power to the float (AC drop from household solar + 12V DC supply)
- [ ] Phase 4: Full feature set (DO, pH, camera, boxes)
- [ ] NTC/Philippine Electrical Code sign-off and licensed-electrician commissioning
- [ ] Field testing in mangrove environment
- [ ] System validation

---

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request or open an Issue.

1. Fork the repository
2. Create your feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add some amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

---

## License

Distributed under the MIT License. See [`LICENSE`](LICENSE) for more information.

---

## Contact

**QPPD** — [github.com/qppd](https://www.github.com/qppd)

Project Link: [https://github.com/qppd/crab-grid-smart-aeration](https://github.com/qppd/crab-grid-smart-aeration)

---

<div align="center">

**Built with love for sustainable aquaculture**

</div>
