<div align="center">

# 🦀 Solar Automated LoRaWAN-Integrated Mud Crab Aeration Network with Crab Grid Observation

An off-grid smart aquaculture system for mud crab fattening in mangrove areas — combining physical crab separation, continuous water quality monitoring, automated corrective actions, LoRaWAN remote control, and overhead visual security.

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Platform: ESP32](https://img.shields.io/badge/Platform-ESP32-red.svg)](https://www.espressif.com/)
[![LoRaWAN: AS923](https://img.shields.io/badge/LoRaWAN-AS923-green.svg)](https://lora-alliance.org/)
[![Solar Powered](https://img.shields.io/badge/Power-Solar%2012V-yellow.svg)]()
[![Status: In Development](https://img.shields.io/badge/Status-In%20Development-orange.svg)]()

**Author:** [QPPD](https://www.github.com/qppd)

</div>

---

## 📖 Table of Contents

- [About the Project](#about-the-project)
- [Core Capabilities](#core-capabilities)
- [System Architecture](#system-architecture)
- [Structure & Layout](#structure--layout)
- [Electronics & Hardware](#electronics--hardware)
- [Power System](#power-system)
- [Software & Firmware](#software--firmware)
- [Bill of Materials (BOM)](#bill-of-materials-bom)
- [Budget Summary](#budget-summary)
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

Mud crab (*Scylla serrata*) fattening is a vital livelihood in Southeast Asian mangrove communities, but crabs are notoriously cannibalistic — they eat each other when crowded or molting. Traditional farming relies on manual water monitoring, which is reactive rather than proactive. In remote mangrove areas, there is no reliable electricity or Wi-Fi, making conventional IoT solutions impractical.

This project solves these problems with a **fully autonomous, off-grid aquaculture system** that:

1. Physically protects crabs using individual floating cages
2. Continuously senses water quality with multiple environmental sensors
3. Automatically triggers corrective actions (aeration, salinity pumps) before conditions become fatal
4. Transmits data over **LoRaWAN** to a remote dashboard for real-time monitoring
5. Provides visual security through an overhead camera system

The entire system runs on **solar power with battery backup**, making it deployable in the most remote coastal locations.

---

## Core Capabilities

| # | Feature | Description |
|---|---------|-------------|
| 🛡️ | **Physical Protection** | Individual floating cages (horizontal crab fattening boxes) physically separate crabs to prevent cannibalism. |
| 📊 | **Water Quality Sensing** | Continuous monitoring for dissolved oxygen, ammonia, pH, temperature, and salinity — the invisible threats that kill crabs. |
| 🚨 | **Automated First Aid** | When conditions become dangerous (e.g., salinity dropping from heavy rain), the system automatically triggers aerators or pumps to correct water quality before crabs die. |
| 📡 | **Off-Grid Remote Control** | LoRaWAN enables farmers to monitor sensor data and control pumps from a remote dashboard — even in mangrove areas with no electricity or Wi-Fi. |
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
          │  Water Sensors │  │  Relay   │ │ │  Camera    │  │  Solar Power │
          │  · EC/Salinity │  │  Module  │ │ │  ESP32-CAM │  │  System      │
          │  · pH Probe    │  │  (8-Ch)  │ │ │  OV2640    │  │  · 200W Panels│
          │  · DS18B20 ×2  │  └────┬─────┘ │ └────────────┘  │  · MPPT 20A │
          │  · DO Sensor   │       │       │                  │  · 200Ah     │
          └────────────────┘  ┌────┴─────┐ │                  │    LiFePO4   │
                              │ Actuators│ │                  └──────────────┘
                              │ · Air    │ │
                              │   Pumps  │ │
                              │ · Bilge  │ │
                              │   Pumps  │ │
                              └──────────┘ │
                                           │
                              ┌─────────────┴───────────┐
                              │   8 × Individual Crab   │
                              │   Fattening Cages on     │
                              │   PVC Floater Grid       │
                              └─────────────────────────┘
```

---

## Structure & Layout

| Specification | Detail |
|---------------|--------|
| **Pond Area** | 500 sqm (modular conceptual prototype) |
| **Capacity** | 8 individual crab fattening cages |
| **Cage Material** | Non-plastic frames to prevent overheating |
| **Floaters** | Ready-made foam-filled pontoon floats (×8–12) — puncture-proof, ≥ 50 kg buoyancy per cage. Hollow PVC pipes fail buoyancy math (see [Components.md](Components.md)) |
| **Aeration** | 2 × 12V DC air pumps (30–60 L/min, multi-outlet) + air stone per cage — sized for 8 cages with N+1 redundancy |
| **Water Circulation** | Air-driven through tubes for movement |
| **Controller Placement** | Centered in the setup for optimal wiring |

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
| **Solar Panel** | 2 × 100W 12V Monocrystalline (IP67) |
| **Charge Controller** | MPPT 20A, 12V/24V with LCD display |
| **Battery** | 12V 200Ah LiFePO4 with BMS |

**Power Budget:**
- Daytime generation: 2 × 100W × 5 sun-hours = **1,000 Wh/day** (≈ 500 Wh at 50% cloud derating)
- Daily load: ~430–700 Wh/day (incl. night aeration — air pumps are a life-support load)
- Night-only load: ~50–60W × 12h = **600 Wh** (50 Ah)
- Battery reserve: 200 Ah → **2+ nights of operation** without solar charging

---

## Software & Firmware

- **Microcontroller Firmware:** ESP32-S3 (Arduino/PlatformIO) — sensor reading, LoRaWAN communication, relay control
- **LoRaWAN Stack:** AS923 region configuration for Philippine deployment
- **Dashboard:** The Things Network (TTN) or ChirpStack for data visualization and pump control
- **Camera:** WiFi-based streaming from ESP32-CAM (local AP during pond visits)

---

## Bill of Materials (BOM)

See the complete Bill of Materials with verified product links, ratings, and pricing:

| Category | Est. Cost (PHP) |
|----------|----------------|
| Boards & Connectivity | ~₱15,500 |
| Sensors (Tier 1: EC + DS18B20) | ~₱5,600 |
| Sensors (Full set: + pH + DO) | ~₱21,500 |
| Aeration + Pumps (air pumps ×2 + stones, bilge ×2) | ~₱4,500 |
| Power System (2× panels, MPPT, 200Ah battery) | ~₱30,000 |
| Structure & Materials (cages, pontoons, misc) | ~₱12,500 |
| **Total (Tier 1 sensors)** | **≈ ₱62,000** |
| **Total (all sensors)** | **≈ ₱78,000** |

> 📋 See [`docs/BOM.md`](docs/BOM.md) for detailed product links, verified ratings, and sourcing from Shopee PH / Lazada PH.

---

## Budget Summary

The project supports a **phased purchase strategy** to match a thesis timeline:

### Phase 1 — Bench Testing (~₱3,000)
Heltec V3 ×2 + DS18B20 + relay module + bilge pump → build & test the control loop on the bench.

### Phase 2 — Remote Dashboard (~₱16,000)
RAK7268 Gateway + EC/Salinity sensor → close the remote monitoring loop.

### Phase 3 — Off-Grid Deployment (~₱30,000)
Solar panels + MPPT controller + 200Ah LiFePO4 battery → go fully off-grid.

### Phase 4 — Full Feature Set (~₱28,000)
DO sensor + pH sensor + ESP32-CAM camera + crab fattening cages + pontoons + air pumps → complete system before panel defense.

---

## Getting Started

### Prerequisites

- [Arduino IDE](https://www.arduino.cc/en/software) or [PlatformIO](https://platformio.org/)
- Heltec ESP32 Board Package
- A [The Things Network](https://www.thethingsnetwork.org/) or ChirpStack account
- RAK7268 LoRaWAN Gateway (configured for AS923)

### Hardware Setup

1. **Scaffold the floating grid** on foam-filled pontoon floats with non-plastic cage frames.
2. **Mount 8 crab fattening cages** on the floater grid with one crab per cage.
3. **Position the controller node** (Heltec V3) in the center of the setup in an IP65 waterproof enclosure.
4. **Connect sensors** to the ESP32-S3 ADC/GPIO pins.
5. **Wire the relay module** to drive air pumps and bilge pumps.
6. **Install the solar power system** — panel → MPPT controller → battery → load distribution.

### Firmware Installation

1. Open the firmware directory in Arduino IDE or PlatformIO.
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
PROJECT2/
├── README.md              # This file
├── Components.md          # Full components specification list
├── docs/
│   └── BOM.md             # Bill of Materials with verified product links
├── references/
│   └── Concept-Questions.md  # Project concept and Q&A
├── firmware/              # ESP32 source code
│   ├── main_controller/   # Heltec V3 sensor + LoRaWAN firmware
│   └── camera_node/       # ESP32-CAM streaming firmware
├── dashboard/             # Dashboard configuration and scripts
├── docs/                  # Additional documentation
│   ├── architecture.md
│   ├── wiring-diagrams/
│   └── images/
├── LICENSE                # MIT License
└── .gitignore
```

---

## Compatibility Notes

| # | Check | Status |
|---|-------|--------|
| 1 | LoRa band (Philippines = AS923) | ✅ RAK7268 and Heltec V3 both support AS923 |
| 2 | Voltage chain (12V battery → pumps/relays, 3.3V logic) | ✅ Onboard LDO + buck converters |
| 3 | Camera connectivity (WiFi only, not LoRaWAN) | ⚠️ Connect via local AP during visits or place near internet |
| 4 | Ammonia sensing (no affordable NH₃ sensor) | ⚠️ Estimate from pH + temperature (cited academic method) |
| 5 | Sensor ADC levels (PH-4502C outputs up to 5V) | ⚠️ Voltage divider needed for ESP32 3.3V input |
| 6 | Crab cage overheating (plastic walls) | ✅ Non-plastic frames + shading |
| 7 | Relay load capacity | ✅ All pumps ≤ 10A per channel; 8-ch module covers all loads + spares |
| 8 | Night operation | ✅ 200Ah battery covers 2+ nights (aeration runs at night) |
| 9 | Floater buoyancy | ✅ Foam-filled pontoons ≥ 50 kg buoyancy per cage (2:1 safety); hollow PVC pipes would sink — corrected |

---

## Roadmap

- [x] Concept definition and documentation
- [x] Component selection and BOM verification
- [ ] Phase 1: Bench testing with Heltec V3 + basic sensors
- [ ] Phase 2: LoRaWAN gateway integration and remote dashboard
- [ ] Phase 3: Off-grid solar deployment
- [ ] Phase 4: Full feature set (DO, pH, camera, cages)
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
