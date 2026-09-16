<div align="center">

# 🦀 Solar Automated LoRaWAN-Integrated Mud Crab Aeration Network with Crab Grid Observation

An off-grid smart aquaculture system for mud crab fattening in mangrove areas — combining physical crab separation, continuous water quality monitoring, automated corrective actions, LoRaWAN remote control, and overhead visual security.

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Platform: ESP32](https://img.shields.io/badge/Platform-ESP32-red.svg)](https://www.espressif.com/)
[![LoRaWAN: AS923](https://img.shields.io/badge/LoRaWAN-AS923-green.svg)](https://lora-alliance.org/)
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
4. Transmits data over **LoRaWAN** to a remote dashboard for real-time monitoring
5. Provides visual security through an overhead camera system

The prototype is powered from the **household's existing solar installation** (220V AC, available 24/7 from an array and a battery bank of over 600 Ah), so it needs no panels or batteries of its own — only a protected AC drop from the house to the float plus a 12V DC supply on site.

---

## Core Capabilities

| # | Feature | Description |
|---|---------|-------------|
| 🛡️ | **Physical Protection** | Individual floating cages (horizontal crab fattening boxes) physically separate crabs to prevent cannibalism. |
| 📊 | **Water Quality Sensing** | Continuous monitoring of dissolved oxygen, pH, temperature and salinity — plus a derived ammonia estimate — targeting the invisible threats that kill crabs. |
| 🚨 | **Automated First Aid** | When conditions become dangerous (e.g., salinity dropping from heavy rain), the system automatically triggers aerators or pumps to correct water quality before crabs die. |
| 📡 | **Remote Monitoring & Control** | LoRaWAN carries sensor data and pump commands to a remote dashboard — even where there is no internet or Wi-Fi at the pond. |
| 📷 | **Visual Security** | Overhead camera to monitor crab molting, deter animal predators, and spot human poachers. |

---

## System Architecture

```
                            ┌─────────────────────────────────────┐
                            │       REMOTE DASHBOARD (TTN /       │
                            │       ChirpStack / Custom UI)       │
                            └──────────────┬──────────────────────┘
                                           │ Internet (WiFi/Ethernet/LTE)
                                           │
                            ┌──────────────┴──────────────────────┐
                            │   RAK7268 LoRaWAN Gateway (AS923)   │
                            └──────────────┬──────────────────────┘
                                           │ LoRa (923 MHz)
                            ┌──────────────┴──────────────────────┐
                            │  Heltec WiFi LoRa 32 V3 (ESP32-S3  │
                            │  + SX1262) — Main Controller Node   │
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
| **Floaters** | Ready-made foam-filled pontoon floats (×8–12) — puncture-proof, ≥ 50 kg buoyancy per cage. Hollow PVC pipes fail buoyancy math (see [Components.md](Components.md)) |
| **Aeration** | 2 × 12V DC air pumps (30–60 L/min, multi-outlet) + air stone per cage — sized for 8 cages with N+1 redundancy |
| **Water Circulation** | Air-driven through tubes for movement |
| **Controller & Power** | Controller centered in the setup for optimal wiring. Powered by a 220V AC drop from the household solar system (24/7) → 30 mA RCD + breaker → 12V 30A DC supply |

> 📋 See [`Components.md`](Components.md) for the full component specifications and [`docs/BOM.md`](docs/BOM.md) for the detailed Bill of Materials with verified product links and pricing.

---

## Electronics & Hardware

| Component | Specification | Purpose |
|-----------|--------------|---------|
| **Heltec WiFi LoRa 32 V3** | ESP32-S3 + SX1262, 0.96" OLED | Main controller node — reads all sensors, runs automation logic, communicates via LoRa |
| **RAK7268 WisGate Edge Lite 2** | 8-channel, AS923 | LoRaWAN gateway — bridges sensor data to cloud dashboard |
| **ESP32-CAM** | OV2640, WiFi/BT | Overhead camera node for visual monitoring |
| **8-Channel Relay Module** | 12V coil, optocoupler, low-level trigger | Drives aerators and salinity pumps with spare channels |
| **DFRobot EC/Salinity Sensor** | K=10 (seawater-range) | Measures salinity — key parameter for rain dilution detection |
| **pH Electrode E-201-C** | PH-4502C analog board | Monitors pH (target: 7.5–8.5 for brackish water) |
| **DS18B20 Temperature Probes** | Waterproof, stainless steel ×2 | Digital temperature sensing (1-Wire) |
| **DFRobot Dissolved Oxygen Kit** | SEN0237-A, galvanic probe | Measures dissolved oxygen — the #1 crab killer |
| **12V DC Air Pumps** | Electromagnetic, ≥ 30–60 L/min, ≥ 4 outlets each ×2 | Oxygenation for all 8 cages (~2 L/min per cage) + N+1 redundancy |
| **12V Marine Bilge Pumps** | 1100 GPH, submersible ×2 | Dynamic salinity correction during rain events |

---

## Power System

| Component | Specification |
|-----------|--------------|
| **AC Source** | 220V AC drop from the household solar system (array + **>600 Ah** battery bank + inverter) — available 24/7 |
| **AC Protection** | 30 mA RCD/GFCI + 2-pole breaker at the house end; outdoor-rated earthed cable (2.5 mm²) run overhead or in conduit |
| **On-Site DC Supply** | 12V 30A (360 W) switching PSU → pumps, relay module, controller; 12V→5V buck for the camera node |

**Night load:** ~50–60 W (≈ 5 A at 12 V) for aeration, a life-support load easily covered by the house battery bank — the full ⚡ Power Budget is in [`docs/BOM.md`](docs/BOM.md).

> ⚠️ **Single point of failure:** no on-site battery, so a tripped breaker or a cut cable stops aeration. Alert on uplink loss and keep the breaker at the house end reachable.

---

## Software & Firmware

- **Microcontroller Firmware:** ESP32-S3 (Arduino/PlatformIO) — sensor reading, LoRaWAN communication, relay control
- **LoRaWAN Stack:** AS923 region configuration for Philippine deployment
- **Dashboard:** The Things Network (TTN) or ChirpStack for data visualization and pump control
- **Camera:** WiFi-based streaming from ESP32-CAM (local AP during pond visits)

---

## BOM & Budget

Every part, price, product link and rating lives in [`docs/BOM.md`](docs/BOM.md), together with the compatibility verification and the power budget. Headline totals: **Tier 1 sensors ≈ ₱41,000–51,000**; **full sensor set ≈ ₱54,500–65,800** (per-category ranges are in the BOM).

Purchase is phased to match the thesis timeline — all four phases ≈ **₱50,000–57,000** for the full build:

### Phase 1 — Bench Testing (~₱5,000)
Heltec V3 ×2 + DS18B20 + relay module + one bilge pump → build & test the control loop on the bench (12V from a lab supply, or buy the on-site PSU now).

### Phase 2 — Remote Dashboard (~₱17,000)
RAK7268 gateway + EC/salinity sensor + refractometer + calibration standard → close the remote monitoring loop.

### Phase 3 — Power to the Float (~₱3,000–4,500)
Protected 220V AC drop from the house (30 mA RCD + breaker) + 12V 30A DC supply + fuses/glands → run the float off the household solar system.

### Phase 4 — Full Feature Set (~₱25,000–30,000)
DO sensor + pH sensor + ESP32-CAM camera + crab fattening boxes + foam pontoons + air pumps + brine reserve → complete system before panel defense.

---

## Getting Started

### Prerequisites

- [Arduino IDE](https://www.arduino.cc/en/software) or [PlatformIO](https://platformio.org/)
- Heltec ESP32 Board Package
- A [The Things Network](https://www.thethingsnetwork.org/) or ChirpStack account
- RAK7268 LoRaWAN Gateway (configured for AS923)
- pH/EC calibration standards and a handheld refractometer (see [`docs/BOM.md`](docs/BOM.md) §6)

### Hardware Setup

1. **Scaffold the floating grid** on foam-filled pontoon floats, with the frame and gantry in bamboo or marine plywood.
2. **Mount the 8 fattening boxes** on the grid, one crab per box, shaded from direct sun.
3. **Position the controller node** (Heltec V3) in the centre of the setup in an IP65 enclosure.
4. **Connect the sensors** to the ESP32-S3 ADC/GPIO pins — with a voltage divider on the pH channel.
5. **Wire the relay module** to the air pumps and bilge pumps.
6. **Run the power drop** — 220V AC from the house (30 mA RCD + breaker at the house end) → 12V 30A DC supply on the float → fused distribution.
7. **Calibrate before trusting any reading** — 2-point pH (4.01 / 6.86), EC with the 12.88 mS/cm standard, and cross-check salinity with the refractometer.

### Firmware Installation

1. Create the `firmware/` directory (planned — see [Project Structure](#project-structure)) and open it in Arduino IDE or PlatformIO.
2. Install required libraries:
   - `Heltec_ESP32`
   - `LoRaWAN_Arduino` (SX1262)
   - `OneWire` / `DallasTemperature` (DS18B20)
   - `DFRobot_EC` / `DFRobot_pH` (DFRobot sensors)
3. Configure your LoRaWAN credentials (DevEUI, AppEUI, AppKey).
4. Select the **AS923** frequency region.
5. Upload the firmware to the Heltec V3.

### Dashboard Configuration

1. Register your LoRaWAN device on TTN or ChirpStack.
2. Create a dashboard to visualize:
   - Real-time sensor readings (DO, pH, temperature, salinity)
   - Pump activation history
   - Alert thresholds and automated responses
3. Enable remote pump control via downlink messages.

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
├── firmware/             # ⏳ planned — ESP32 source code
│   ├── main_controller/  # ⏳ planned — Heltec V3 sensor + LoRaWAN firmware
│   └── camera_node/      # ⏳ planned — ESP32-CAM streaming firmware
└── dashboard/            # ⏳ planned — dashboard configuration and scripts
⏳ = planned / not yet in the repository.
Committed today (5 files): README.md · Components.md · LICENSE · .gitignore · docs/BOM.md
Create each ⏳ item as its roadmap phase completes; see docs/BOM.md for the AC power build.
```

---

## Compatibility Notes

All ten system-level checks, with their verdicts, are in [`docs/BOM.md`](docs/BOM.md). Four items still need action before deployment:

| Open item | Action |
|-----------|--------|
| Camera is WiFi-only (video cannot fit LoRa) | Use its own AP during pond visits, run it at the house with the gateway, or fit an LTE camera |
| No affordable NH₃ sensor | Estimate unionized ammonia from pH + temperature and cite the method in the paper |
| PH-4502C outputs up to 5V | Add a voltage divider before the 3.3V ESP32 ADC |
| No on-site battery | Alert on uplink loss (AC-fail module); add a small 12V UPS if unattended nights are critical |

---

## Roadmap

- [x] Concept definition and documentation
- [x] Component selection and BOM verification
- [ ] Phase 1: Bench testing with Heltec V3 + basic sensors
- [ ] Phase 2: LoRaWAN gateway integration and remote dashboard
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
