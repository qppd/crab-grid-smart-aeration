<div align="center">

# 🦀 Solar Automated LoRa-Integrated Mud Crab Aeration Network with Crab Grid Observation

An off-grid smart aquaculture system for mud crab fattening in mangrove areas — combining physical crab separation, continuous water quality monitoring, automated corrective actions, LoRa remote control, and overhead visual security.

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Platform: ESP32](https://img.shields.io/badge/Platform-ESP32-red.svg)](https://www.espressif.com/)
[![LoRa: 433 MHz P2P](https://img.shields.io/badge/LoRa-433%20MHz%20P2P-green.svg)]()
[![Power: 220V AC from House Solar](https://img.shields.io/badge/Power-220V%20AC%20from%20House%20Solar-yellow.svg)]()
[![Status: In Development](https://img.shields.io/badge/Status-In%20Development-orange.svg)]()

**Author:** [QPPD](https://www.github.com/qppd)

</div>

---

> 📚 **Where to find what:** this README = overview, architecture and timeline · [`Components.md`](Components.md) = engineering specs and validation rationale · [`docs/BOM.md`](docs/BOM.md) = prices, product links, compatibility verification and power budget.

---

## 📖 Table of Contents

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
3. Automatically triggers corrective actions (aeration, salinity pumps) before conditions become fatal
4. Transmits data over **LoRa** to a bridge node at the house (where WiFi exists) for real-time remote monitoring
5. Provides visual security through an overhead camera system

The prototype is powered from the **household's existing solar installation** (220V AC, available 24/7 from the array, the inverter and a battery bank of over 600 Ah), so it needs no panels or batteries of its own — only a protected AC drop from the house to the float plus a 12V DC supply on site.

---

## Core Capabilities

| # | Feature | Description |
|---|---------|-------------|
| 🛡️ | **Physical Protection** | Individual floating cages (horizontal crab fattening boxes) physically separate crabs to prevent cannibalism. |
| 📊 | **Water Quality Sensing** | Continuous monitoring of dissolved oxygen, pH, temperature and salinity — plus a derived ammonia estimate — targeting the invisible threats that kill crabs. |
| 🚨 | **Automated First Aid** | When conditions become dangerous (e.g., salinity dropping from heavy rain), the system automatically triggers aerators or pumps to correct water quality before crabs die. |
| 🎛️ | **Manual & Automatic Control** | Runs itself in **AUTO** (firmware thresholds drive the pumps) and accepts **MANUAL** or **OFF** from the dashboard webapp, with a physical selector on the box as the fail-safe. |
| 📡 | **Remote Monitoring & Control** | LoRa carries sensor data and pump commands to the house bridge node, which feeds the remote dashboard — even where there is no internet or Wi-Fi at the pond. |
| 📷 | **Visual Security** | Overhead camera to monitor crab molting, deter animal predators, and spot human poachers. |

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
                            │  House Bridge Node (ESP32 + RA-02)  │
                            │  at the house, where WiFi exists    │
                            └──────────────┬──────────────────────┘
                                           │ LoRa (433 MHz, P2P)
                            ┌──────────────┴──────────────────────┐
                            │   ESP32 DevKit + RA-02 (SX1278)     │
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
| **Boxes & frames** | Slotted plastic boxes, shaded under the frame; frame/gantry in bamboo or marine plywood so no bare plastic sits in direct sun |
| **Floaters** | Ready-made foam-filled pontoon floats (×8–12) — puncture-proof, ≥ 50 kg buoyancy per cage. The PVC pipes are **conduit, never flotation**: water-filled pipe weighs ~1 kg per litre (see [Components.md](Components.md)) |
| **Aeration** | 2 × 12V DC air pumps (30–60 L/min, multi-outlet) + air stone per cage — sized for 8 cages with N+1 redundancy |
| **Air & water routing** | Two separate circuits inside the PVC pipe grid: an air header with per-cage drop tubes and stones, and a water header fed by the bilge pumps with a valved outlet per cage. Flexible hose runs from pump to cage are an option/backup |
| **Sensor hub** | Centred on the grid for balance and symmetric cable runs; probes at mid-depth (~20–30 cm), clear of the aerator plume and the brine outlet. One DO probe covers the grid because aeration is a whole-grid action |
| **Controller & Power** | Controller centred in the setup for optimal wiring. Powered by a 220V AC drop from the household solar system (24/7) → 30 mA RCD + breaker → 12V 30A DC supply |

> 📋 See [`Components.md`](Components.md) for the full component specifications and [`docs/BOM.md`](docs/BOM.md) for the detailed Bill of Materials with verified product links and pricing.

---

## Electronics & Hardware

| Component | Specification | Purpose |
|-----------|--------------|---------|
| **ESP32 DevKit 38-pin + RA-02 (SX1278)** | ESP32-WROOM-32 + 433 MHz LoRa, +20 dBm | Main controller node — reads all sensors, runs automation logic, communicates over point-to-point LoRa |
| **House Bridge Node (ESP32 + RA-02)** | Same radio stack, at the house | LoRa ⇄ WiFi bridge — pushes float telemetry to the Firebase dashboard and relays dashboard commands back down over LoRa |
| **ESP32-CAM** | OV2640, WiFi/BT | Overhead camera node for visual monitoring |
| **8-Channel Relay Module** | 12V coil, optocoupler, low-level trigger | Drives aerators and salinity pumps with spare channels |
| **DFRobot EC/Salinity Sensor** | K=10 (seawater-range) | Measures salinity — key parameter for rain dilution detection |
| **pH Electrode E-201-C** | PH-4502C analog board | Monitors pH (target: 7.5–8.5 for brackish water) |
| **DS18B20 Temperature Probes** | Waterproof, stainless steel ×2 | Digital temperature sensing (1-Wire) |
| **DFRobot Dissolved Oxygen Kit** | SEN0237-A, galvanic probe — **one unit (single measurement point)** | Measures dissolved oxygen — the #1 crab killer; mounted on the centred hub |
| **12V DC Air Pumps** | Electromagnetic, ≥ 30–60 L/min, ≥ 4 outlets each ×2 | Oxygenation for all 8 cages (~2 L/min per cage) + N+1 redundancy |
| **12V Marine Bilge Pumps** | 1100 GPH, submersible ×2 | Dynamic salinity correction during rain events |

---

## Power System

| Component | Specification |
|-----------|--------------|
| **AC Source** | 220V AC drop from the household solar system (array + **>600 Ah** battery bank + inverter) — available 24/7 |
| **AC Protection** | 30 mA RCD/GFCI + 2-pole breaker at the house end; outdoor-rated earthed cable (2.5 mm²) run overhead or in conduit |
| **On-Site DC Supply** | 12V 30A (360 W) switching PSU → pumps, relay module, controller; 12V→5V buck for the camera node |

**Night load:** ~50–60 W (≈ 5 A at 12 V) for aeration, a life-support load easily covered by the house battery bank and inverter — the full ⚡ Power Budget is in [`docs/BOM.md`](docs/BOM.md).

> ⚠️ **Single point of failure:** no on-site battery, so a tripped breaker or a cut cable stops aeration. Alert on uplink loss and keep the breaker at the house end reachable.

---

## Software & Firmware

- **Microcontroller Firmware:** ESP32 (Arduino/PlatformIO) — sensor reading, LoRa P2P communication, relay control
- **LoRa Link:** point-to-point 433 MHz between the float node and the house bridge node — no LoRaWAN server (TTN/ChirpStack) needed
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
- MANUAL auto-reverts to AUTO after 30 min without a command, so a forgotten switch cannot kill the crabs
- A DO reading below 3 ppm overrides MANUAL, forces aeration on and raises an alarm
- Mode and pump state are echoed in every uplink, so the dashboard shows what the device actually believes

**Downlink payload (2 bytes):** byte 0 = mode (0 AUTO, 1 MANUAL, 2 OFF), byte 1 = pump bitmap (air pumps 1/2, bilge 1/2). Commands are applied on the device's **next uplink** (the float polls the bridge on its 5–15 min interval) — that latency is exactly why the local selector exists.

---

## BOM & Budget

Every part, price, product link and rating lives in [`docs/BOM.md`](docs/BOM.md), together with the compatibility verification and the power budget. Headline totals: **Tier 1 sensors ≈ ₱32,200–44,600**; **full sensor set ≈ ₱45,600–59,200** (per-category ranges are in the BOM).

Purchase is phased to match the thesis timeline — the four phases add up to the build total above (≈ **₱45,600–59,200** at item prices):

### Phase 1 — Bench Testing (~₱2,950–3,150)
2 × ESP32 DevKit + 2 × RA-02/antenna + DS18B20 + relay module + one bilge pump + the AUTO/OFF/MANUAL selector → build & test the control loop and the LoRa link on the bench (12V from a lab supply, or buy the on-site PSU now).

### Phase 2 — Remote Dashboard (~₱7,000–7,400)
House bridge node (ESP32 + RA-02) + EC/salinity sensor + refractometer + calibration standard → close the remote monitoring loop.

### Phase 3 — Power to the Float (~₱3,200–5,200)
Protected 220V AC drop from the house (30 mA RCD + breaker) + 12V 30A DC supply + AC-fail module + fuses/glands → run the float off the household solar system.

### Phase 4 — Full Feature Set (~₱32,500–43,400)
DO sensor + pH sensor (+ buffers) + ESP32-CAM camera + second bilge pump + fattening boxes + frame/netting + foam pontoons + the PVC air/water grid + sensor hub + air pumps + misc hardware + brine reserve → complete system before panel defense.

> 💰 **₱50,000 hardware ceiling:** if the build runs over, the **camera is the first cut** — #3 plus its buck (#19) is ≈ ₱705–769. The DO kit (₱12–13k) is what pushes the full set past the ceiling, so it belongs in the last phase. The BOM carries the full cut order and the resulting **₱32,400–44,200** configuration; the DO probe and the aerators are never cut.

---

## Getting Started

### Prerequisites

- [Arduino IDE](https://www.arduino.cc/en/software) or [PlatformIO](https://platformio.org/)
- ESP32 board package (esp32 by Espressif)
- `sandeepmistry/LoRa` library (SX1278/RA-02) · `OneWire` / `DallasTemperature` (DS18B20) · `DFRobot_EC` / `DFRobot_pH` (DFRobot sensors)
- 3 × ESP32 DevKit 38-pin + 3 × RA-02 + antennas (float node, house bridge, spare — see [`docs/BOM.md`](docs/BOM.md) §1)
- pH/EC calibration standards and a handheld refractometer (see [`docs/BOM.md`](docs/BOM.md) §6)

### Hardware Setup

1. **Scaffold the floating grid** on foam-filled pontoon floats, with the frame and gantry in bamboo or marine plywood.
2. **Mount the 8 fattening boxes** on the grid, one crab per box, shaded from direct sun.
3. **Position the controller node** (ESP32 + RA-02) in the centre of the setup in an IP65 enclosure.
4. **Connect the sensors** to the ESP32 ADC/GPIO pins per the pin map in [`Components.md`](Components.md) — with a voltage divider on the pH channel.
5. **Wire the relay module** to the air pumps and bilge pumps, following the WiFi/boot-safe pin map in [`Components.md`](Components.md) (no analog on ADC2 pins while WiFi runs, nothing on boot-strap pins 0/12/15).
6. **Run the power drop** — 220V AC from the house (30 mA RCD + breaker at the house end) → 12V 30A DC supply on the float → fused distribution.
7. **Calibrate before trusting any reading** — 2-point pH (4.01 / 6.86), EC with the 12.88 mS/cm standard, and cross-check salinity with the refractometer.

### Firmware Installation

1. Create the `firmware/` directory (planned — see [Project Structure](#project-structure)) and open it in Arduino IDE or PlatformIO.
2. Install required libraries:
   - `LoRa` by sandeepmistry (RA-02 / SX1278, the standard 18/19/23/5/14/26 SPI pin set)
   - `OneWire` / `DallasTemperature` (DS18B20)
   - `DFRobot_EC` / `DFRobot_pH` (DFRobot sensors)
3. Configure the radio settings identically on every node (433 MHz, SF10–12, matching bandwidth/coding rate/sync word).
4. WiFi stays **off** on the float node; only the house bridge joins the network.
5. Upload the firmware to each ESP32.

### Dashboard Configuration

1. **Create the back end** — a Firebase project with the **Realtime Database** (live telemetry) and Firebase Auth for the farmer accounts.
2. **Deploy the webapp** — a Next.js app on Vercel's free tier (or Firebase Hosting), with the service-account values set as env vars: `FIREBASE_DATABASE_URL`, `FIREBASE_PROJECT_ID`, `FIREBASE_CLIENT_EMAIL`, `FIREBASE_PRIVATE_KEY`.
3. **Connect the uplinks** — the house bridge node relays each float packet into the Realtime Database (`devices/<id>/state`), where the webapp's `onValue()` listeners stream it straight to the UI.
4. **Build the control panel** — live readings, pump state, the AUTO/MANUAL/OFF selector, per-pump overrides and alert thresholds; a route handler writes the command into the Realtime Database where the house bridge polls it and relays it down over LoRa.

---

## Project Structure

```
crab-grid-smart-aeration/
├── README.md              # This file
├── Components.md          # Full components specification list
├── LICENSE                # MIT License
├── .gitignore
├── docs/                  # Documentation
│   ├── BOM.md             # Bill of Materials — product links, ratings, prices
│   ├── architecture.md    # ⏳ planned
│   ├── wiring-diagrams/   # ⏳ planned
│   └── images/            # ⏳ planned
├── firmware/              # ⏳ planned — ESP32 source code
│   ├── main_controller/  # ⏳ planned — ESP32+RA-02 sensor + LoRa firmware
│   ├── house_bridge/     # ⏳ planned — LoRa ⇄ WiFi relay node firmware
│   └── camera_node/      # ⏳ planned — ESP32-CAM streaming firmware
└── dashboard/            # ⏳ planned — Next.js + Realtime Database webapp
⏳ = planned / not yet in the repository — create each item as its roadmap phase completes.
```

---

## Compatibility Notes

All thirteen system-level checks, with their verdicts, are in [`docs/BOM.md`](docs/BOM.md). Five items still need action before deployment:

| Open item | Action |
|-----------|--------|
| Camera is WiFi-only (video cannot fit LoRa) | Use its own AP during pond visits, run it at the house with the bridge node, or fit an LTE camera |
| No affordable NH₃ sensor | Estimate unionized ammonia from pH + temperature and cite the method in the paper |
| PH-4502C outputs up to 5V | Add a voltage divider before the 3.3V ESP32 ADC |
| No on-site battery | Alert on uplink loss (AC-fail module); confirm the house inverter stays on 24/7; add a small 12V UPS if unattended nights are critical |
| Control needs internet, hosting and the bridge node | Local selector as fail-safe; MANUAL auto-reverts to AUTO; commands land on the next uplink |

---

## Roadmap

- [x] Concept definition and documentation
- [x] Component selection and BOM verification
- [ ] Phase 1: Bench testing with ESP32 + RA-02 + basic sensors
- [ ] Phase 2: House bridge node integration and remote dashboard
- [ ] Phase 3: Power to the float (AC drop from household solar + 12V DC supply)
- [ ] Phase 4: Full feature set (DO, pH, camera, boxes)
- [ ] Field testing in mangrove environment
- [ ] Thesis panel defense

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

**Built with ❤️ for sustainable aquaculture**

</div>
