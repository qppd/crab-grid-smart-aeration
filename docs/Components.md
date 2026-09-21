# Components List

**Project:** Solar Automated LoRa-Integrated Mud Crab Aeration Network with Crab Grid Observation

> Every count and specification below was **validated against web sources on Sept 2026** — SEAFDEC/AQD (Philippines) mud crab culture standards, FAO mud crab manual, DFRobot sensor datasheets, and Shopee/Lazada PH listings. Rationale for each correction is stated inline; see [`BOM.md`](BOM.md) for the verified product links.

---

## Physical Structure

| Component | Spec / Count | Validated? | Purpose / Notes |
|-----------|--------------|-----------|-----------------|
| Floating Cages | **8 × individual crab fattening boxes** (plastic, with slots/holes for water exchange) | Correct count | 1 crab per box = anti-cannibalism, matching commercial practice (individual boxes/cages are the standard mud crab fattening method). Mud crabs tolerate crowding **in water** — the constraint is separation, not volume, so 8 units is adequate for a 500 sqm prototype. |
| Cage size (target) | ≥ 45 × 40 × 35 cm water-filled, e.g. Allied box (457 × 406 × 406 mm) | Sized | Holds a 0.5–1.5 kg fattening crab without contact with hot plastic walls; FAO/industry boxes range from 31 × 26 × 10 cm up to 458 × 406 × 354 mm. |
| Floaters | **Ready-made foam-filled pontoon floats** (HDPE/LLDPE shell, one-piece foam-filled) — **8–12 units**, NOT hollow PVC pipes | **Corrected** | Size pontoons from the measured dry above-water load, not from the weight of water inside a slotted submerged cage. Water in a slotted cage is continuous with the estuary and is not a dead load while submerged; the crab and plastic have their own buoyancy. Use manufacturer-rated floats (target ≥50 kg each), verify the actual box/frame dry mass and load distribution, then apply at least a 2:1 safety factor. A sealed hollow 4″ PVC pipe gives only ~2.6 kg/m and is not primary flotation. The pipe grid is **conduit only, never flotation** — water-filled pipe adds ~1 kg per litre and must be included in the dry/wet load test. |
| Cage frames / gantry | UV-stabilized PVC/FRP preferred; bamboo or marine plywood only with marine-grade sealing | **Material correction** | Raw bamboo and ordinary plywood rot, split, and attract marine borers in a warm brackish mangrove. If wood is retained, seal all faces and end grain with marine epoxy/PU, keep sacrificial contact details replaceable, and specify Grade 316 stainless or nylon fasteners. |
| Air / water routing | **The PVC pipe grid is the distribution network**: a separate air header (drop tube + stone per cage) and a water header fed by the bilge pumps (valved outlet per cage); flexible hose runs pump→cage as an option/backup | **Never one lumen for both** — water floods an air line and kills aeration, while air pockets choke water flow. Keep the air circuit above the waterline or add a drip loop. A 25 mm (1") header with 8 × 6–8 mm outlets balances flow; split the grid in two, one half per bilge pump. Add flush ports — salt and algae will settle in the runs. |
| Sensor hub | Probe cluster **centred on the grid**, holding EC/pH/DO/2× DS18B20 at mid-depth (~20–30 cm below the surface) | Centred = balanced float load and symmetric cable runs. Mid-depth readings are representative: the surface is skewed by aeration and sun, the bottom by sludge and salinity stratification. Keep it clear of the aerator plume, provide gentle flow across the DO membrane before readings, and make it removable for calibration. One DO probe remains a single-point risk. |
| Water circulation | Air-driven through the pipe grid, plus controlled circulation/flushing through the water header | **Scope correction** | Bubbling creates movement inside each cage; the valved outlets let the circulation pumps flush or mix a test volume. Reliable salinity correction is not claimed for open slotted cages because tidal flushing and dense brine sinking defeat dosing. Use temporary isolation sleeves or a closed-loop/RAS test if salinity control is an objective. |

## Electronics & Hardware

| Component | Spec / Count | Validated? | Purpose / Notes |
|-----------|--------------|-----------|-----------------|
| Controller | **ESP32 DevKit (38-pin, ESP32-WROOM-32) + EBYTE E22-900M22S (SX1262, 915 MHz)**, centred in the setup, IP65 enclosure | OK with compliance gate | Runs automation logic; links sensors, pumps, and the LoRa point-to-point link. WiFi stays off on the float. Budget pick over the Heltec V3: ~₱740/node (board + module + antenna) instead of ₱1,974, and the pin map below avoids every WiFi/boot conflict. 915 MHz is the Philippine NTC licence-free SRD band; confirm EIRP/type-approval before deployment. |
| Water Quality Sensors | 4 sensor types: EC/salinity (K=10), pH, dissolved oxygen (**one probe**), temperature (2× DS18B20 probes) | Validated ranges with limits | **EC K=10 (0–100 mS/cm)** covers the conductivity range around seawater (35 ppt is approximately 53 mS/cm at 25 °C), but salinity must use the manufacturer/PSS-78 conversion, not a fixed linear ppt factor. **pH E-201-C** covers the 7.5–8.5 brackish target. **DO galvanic probe (SEN0237-A)** supports the ≥5 ppm target, but one probe is a single-point failure; provide flow across the membrane, a handheld backup, and spare membrane/electrolyte. DS18B20 covers the 27–30 °C band. |
| LoRa ⇄ WiFi Bridge | **A second ESP32 DevKit + E22-900M22S (915 MHz) at the house** where WiFi/internet exists — the "bridge node" | **Replaced LoRaWAN gateway (Sept 2026)** | The ₱10,645 LoRaWAN gateway is gone. The bridge receives float uplinks and pushes them into the Firebase webapp; queued commands are pulled from the webapp and sent back down over LoRa — plain point-to-point radio, no TTN/ChirpStack server. SX1262 delivers up to +22 dBm on the 915 MHz SRD band. The 1–3 km LOS estimate is a test target, not a regulatory guarantee; confirm NTC type-approval and measured EIRP before deployment. |
| Aerator | **2 × RESUN MPQ-903 (MPQ-03) 12V DC electromagnetic air pump — 35W, 68 L/min open-flow, 0.068 MPa, ~3.5A, 6 outlets** (NOT 2× mini aquarium pumps) | **Corrected** | Mini pumps (~0.5–2 L/min each) cannot aerate 8 cages. The biological target is 1.5–2 L/min per 100 L, so 8 cages × ~40 L ≈ 320 L needs about **4.8–6.4 L/min delivered total** (roughly 0.6–0.8 L/min per cage). The 68 L/min MPQ-03 is deliberately far above the target to cover depth, tubing and stone losses; run ONE unit on duty with the second on N+1 standby (alternating duty), and use a manifold, needle valves and per-branch flow measurement so excess air is not dumped into the cages. |
| Air stones / tubing | 8–10 × air stones + 20 m 4 mm silicone tubing + gang valve manifold | Added | Distributes air from the 2 pumps into each of the 8 cages. |
| Circulation / Flush Pump | **2 × 12V Marine Bilge Pump 1100 GPH** (submersible) | **Function corrected** | Provides circulation/flushing and can support a controlled salinity experiment. It does not reliably correct salinity in open slotted cages because tidal exchange flushes the dose and dense brine sinks. Use isolation sleeves or a closed-loop/RAS enclosure for a defensible salinity-control test; verify pump current against the relay and fuse rating. |
| Relay Module | **8-channel**, 12V coil, optocoupler, low-level trigger | Chosen | 2 air pumps + 2 bilge pumps = 4 loads, so a 4-ch board would leave zero expansion; 8-ch covers every actuator path plus spares. |
| Camera | Overhead, one unit initially (optional, ESP32-CAM) — **first thing to cut if the build passes ₱50,000** | OK | Monitor molting, deter predators, spot poachers. WiFi-only — see BOM Compatibility #3. |
| Control Interface | **Local rotary selector (AUTO / OFF / MANUAL)** + status LEDs at the controller box, plus webapp control | The LoRa link is polled point-to-point — a command lands only on the next uplink — so a local selector is the fail-safe for a life-support load. Mode and pump state are echoed in every uplink. |
| Dashboard / Webapp | **Next.js + Firebase Realtime Database + Auth**, free-tier hosting (Vercel) | Live readings, AUTO/MANUAL/OFF, per-pump overrides, thresholds. RTDB's `onValue()` sync is a natural fit for telemetry: keep live state in one node and trim the history log so it stays inside the free tier. |
| Power Source | **220V AC drop from the household solar system** (array + battery bank **> 600 Ah** + inverter, available 24/7) + **12V 30 A DC supply** on the float | **Corrected** | The site already sits on a household solar installation with a battery bank over 600 Ah, so panels, MPPT and an on-site battery are unnecessary. Night aeration is a planning estimate of 1 × MPQ-03 (35 W) on duty + node 1.5 W + bilge duty ≈ 40–60 W ≈ **480–720 Wh/night ≈ 3.5–5 A at 12V**; verify inverter efficiency and actual duty cycle. On-site work is only a shore-end 30 mA RCD + earthed outdoor AC run + 12V 30 A PSU (see BOM §4), installed and commissioned by a licensed electrician. With no on-site battery, the AC drop is the single point of failure; the house bridge must alarm on heartbeat loss. |

### Controller pin map — ESP32 38-pin DevKit (WiFi/boot-safe)

> Validated for the ESP32-WROOM-32 DevKit (Makerlab PH, ₱349). Rules: **no analog on ADC2** (GPIO 0/2/4/13/14/15/25/26/27 stop working as analog while WiFi runs), **no actuators on boot-strapping pins** (GPIO 0, 2, 12 and 15 glitch or break boot; 5 is usable only for outputs that idle HIGH, which the radio NSS below does), **UART0 (GPIO 1/3) kept free** for flashing and logs. The float node never enables WiFi and the house bridge reads no analog, but the map is safe under both configurations.

| Function | GPIO | Why it's safe |
|-----------|------|---------------|
| LoRa E22-900M22S SCK / MISO / MOSI | 18 / 19 / 23 | Hardware VSPI — independent of WiFi |
| LoRa E22-900M22S NSS | 5 | Boot-strapping pin, but NSS **idles HIGH (radio deselected)** — safe at boot |
| LoRa E22-900M22S RST | 14 | Driven after boot; the brief boot-time PWM glitch just resets the radio harmlessly |
| LoRa E22-900M22S DIO1 (IRQ) | 26 | SX1262 signals RX/TX-done on DIO1 (not DIO0); idle LOW at boot, no strapping role; GPIO 26 is ADC2-capable but used as a digital IRQ |
| LoRa E22-900M22S BUSY | 17 | SX1262 requires a BUSY line (not present on SX127x); GPIO 17 is a free digital pin (ADC2-capable, digital-only) |
| DS18B20 ×2 (shared 1-Wire bus) | 16 | Free digital pin; external 4.7 kΩ pull-up to 3.3 V |
| pH analog output (via 10 kΩ series + 18 kΩ shunt divider, or ADS1115) | 34 | **ADC1** — works with WiFi on; input-only pin. 5 V → about 3.21 V with the passive divider; use a common ground and, preferably, an external 16-bit I2C ADC for professional-grade pH. |
| EC/salinity analog output | 35 | **ADC1** — input-only |
| DO sensor analog output | 36 (VP) | **ADC1** — input-only |

| Relay 1 — air pump 1 | 25 | Non-strapping output; firmware drives HIGH (relay OFF) before init |
| Relay 2 — air pump 2 | 27 | Same (digital output — the ADC2 restriction affects analog reads only) |
| Relay 3 — bilge pump 1 | 33 | Non-strapping output; same boot practice |
| Relay 4 — bilge pump 2 | 13 | ADC2 pin, but only its **analog** function is WiFi-blocked — digital output is unaffected |
| Rotary selector bit A (AUTO/MANUAL) | 4 | Input with external 10 kΩ pull-up to 3.3 V; selector shorts to GND. GPIO 4 is ADC2-capable but is used only as a digital input. |
| Rotary selector bit B (AUTO/OFF) | 21 | Input with external 10 kΩ pull-up to 3.3 V; selector shorts to GND. |
| Mode LED (external) | 22 | Output |
| Link/status LED | 2 (onboard LED) | The devkit's boot LED doubles as the link indicator |

**Leave unconnected:** GPIO 0 (boot strapping — low at reset = flash mode), GPIO 12 (HIGH at boot = flash-voltage fault → boot failure), GPIO 15 (boot strapping), GPIO 1/3 (USB serial). Relay modules are low-level trigger: drive the GPIO HIGH before initializing the pins, keep the relay off at boot, and add an external 10 kΩ pull-up to 3.3 V on each used relay input if the module does not already provide a reliable pull-up. The house bridge reuses the same SPI wiring (18/19/23/5) plus DIO1 26 and BUSY 17.

---

## Validated Water-Quality Targets (deployment reference)

From **SEAFDEC/AQD (Philippines)** mud crab culture standards:

| Parameter | Target / Safe Range | Sensor |
|-----------|--------------------|--------|
| Dissolved Oxygen | **≥ 5 ppm** | DFRobot SEN0237-A galvanic DO probe |
| Salinity | **28–32 ppt** (grow-out tolerates 10–34 ppt) | DFRobot EC K=10 (0–100 mS/cm); convert with the manufacturer/PSS-78 method. At 25 °C, 35 ppt seawater is approximately 53 mS/cm; do not treat 12.88 mS/cm as 35 ppt. |
| Temperature | **27–30 °C** | 2× DS18B20 stainless waterproof probes |
| pH | **7.5–8.5** | E-201-C electrode + PH-4502C board |
| NH₃ (unionized) | **Cannot be calculated from pH + temperature alone**. pH + temperature estimate the toxic fraction/risk; absolute NH₃ requires measured Total Ammonia Nitrogen (TAN) or a laboratory test. | derived risk indicator only |


## Deployment Corrections and Verification Gates

1. **Buoyancy:** weigh the empty boxes, frame, pumps, electronics and water-retaining pipes; measure the actual float rating and load distribution. Do not use the weight of water inside a slotted submerged cage as a pontoon load. Apply at least a 2:1 safety factor and physically load-test the assembled grid before stocking.
2. **Aeration:** the MPQ-03's 68 L/min rating is open-flow, not guaranteed delivered flow. Measure each branch at operating depth with air stones installed; target approximately 4.8–6.4 L/min total for 320 L and regulate excess flow.
3. **Salinity:** an open slotted cage is hydraulically connected to the estuary. Brine dosing is a controlled experiment only unless the cage is temporarily isolated or the test is closed-loop. Record tide, current, dose, and pre/post salinity at multiple points.
4. **Electrical:** the 220 V AC drop is a licensed-electrician installation. Require shore-end 30 mA RCD protection, weatherproof connectors, drip loops, no submerged joints, conductor/fuse coordination, earthing verification, and a monthly RCD test.
6. **Protocol:** use device ID, sequence number, CRC-8, expiry, ACK/NAK, a 60-second heartbeat, and immediate emergency uplinks for DO or other critical thresholds.
