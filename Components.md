# Components List

**Project:** Solar Automated LoRa-Integrated Mud Crab Aeration Network with Crab Grid Observation

> Every count and specification below was **validated against web sources on Sept 2026** — SEAFDEC/AQD (Philippines) mud crab culture standards, FAO mud crab manual, DFRobot sensor datasheets, and Shopee/Lazada PH listings. Rationale for each correction is stated inline; see [`docs/BOM.md`](docs/BOM.md) for the verified product links.

---

## Physical Structure

| Component | Spec / Count | Validated? | Purpose / Notes |
|-----------|--------------|-----------|-----------------|
| Floating Cages | **8 × individual crab fattening boxes** (plastic, with slots/holes for water exchange) | ✅ Correct count | 1 crab per box = anti-cannibalism, matching commercial practice (individual boxes/cages are the standard mud crab fattening method). Mud crabs tolerate crowding **in water** — the constraint is separation, not volume, so 8 units is adequate for a 500 sqm prototype. |
| Cage size (target) | ≥ 45 × 40 × 35 cm water-filled, e.g. Allied box (457 × 406 × 406 mm) | ✅ Sized | Holds a 0.5–1.5 kg fattening crab without contact with hot plastic walls; FAO/industry boxes range from 31 × 26 × 10 cm up to 458 × 406 × 354 mm. |
| Floaters | **Ready-made foam-filled pontoon floats** (HDPE/LLDPE shell, one-piece foam-filled) — **8–12 units**, NOT hollow PVC pipes | 🔴 **Corrected** | Buoyancy math: full cage + water + crab ≈ 25–30 kg → needs ≥ 50 kg buoyancy per cage (2:1 safety). A **sealed hollow 4″ PVC pipe** gives only ~2.6 kg/m — it would sink. Foam-filled pontoons provide 50–100+ kg each, are puncture-proof (foam cannot leak out), and are UV/stab-resistant. The pipe grid is **conduit only, never flotation** — water-filled pipe adds ~1 kg per litre (≈15 kg for a 15 L run), which the 2:1 factor already absorbs. |
| Cage frames / gantry | Bamboo or marine-plywood grid + shaded frame | ✅ OK | The fattening boxes themselves are plastic, so they are shaded under the frame and kept in water contact; gantry/contact surfaces use non-plastic materials so no bare plastic sits in direct sun. |
| Air / water routing | **The PVC pipe grid is the distribution network**: a separate air header (drop tube + stone per cage) and a water header fed by the bilge pumps (valved outlet per cage); flexible hose runs pump→cage as an option/backup | ✅ Added Sept 2026 | **Never one lumen for both** — water floods an air line and kills aeration, while air pockets choke water flow. Keep the air circuit above the waterline or add a drip loop. A 25 mm (1") header with 8 × 6–8 mm outlets balances flow; split the grid in two, one half per bilge pump. Add flush ports — salt and algae will settle in the runs. |
| Sensor hub | Probe cluster **centred on the grid**, holding EC/pH/DO/2× DS18B20 at mid-depth (~20–30 cm below the surface) | ✅ Added Sept 2026 | Centred = balanced float load and symmetric cable runs. Mid-depth readings are representative: the surface is skewed by aeration and sun, the bottom by sludge and salinity stratification. Keep it clear of the aerator plume and the brine outlet, and make it removable for calibration. |
| Water circulation | Air-driven through the pipe grid, plus brine/high-salinity water injected through the water header | ✅ Refined | Bubbling creates movement inside each cage; the valved outlets let the bilge pumps correct salinity per zone instead of dumping it in one spot. |

## Electronics & Hardware

| Component | Spec / Count | Validated? | Purpose / Notes |
|-----------|--------------|-----------|-----------------|
| Controller | **ESP32 DevKit (38-pin, ESP32-WROOM-32) + EBYTE RA-02 (SX1278, 433 MHz)**, centred in the setup, IP65 enclosure | ✅ OK | Runs automation logic; links sensors, pumps, and the LoRa point-to-point link. WiFi stays off on the float. Budget pick over the Heltec V3: same SX12xx radio family, ~₱740/node (board + module + antenna) instead of ₱1,974, and the pin map below avoids every WiFi/boot conflict. |
| Water Quality Sensors | 4 sensor types: EC/salinity (K=10), pH, dissolved oxygen (**one probe**), temperature (2× DS18B20 probes) | ✅ Validated ranges | **EC K=10 (0–100 mS/cm)** covers seawater (35 ppt ≈ 53 mS/cm) ✔ · **pH E-201-C** covers 7.5–8.5 brackish target ✔ · **DO galvanic probe (SEN0237-A)** covers ≥ 5 ppm DO requirement ✔ · DS18B20 covers 27–30 °C band ✔. One DO probe is a single measurement point for the whole grid, which works because aeration is a whole-grid action (both air pumps stage together) — spot-check the far cages with a handheld meter periodically to confirm the hub reading tracks them. |
| LoRa ⇄ WiFi Bridge | **A second ESP32 DevKit + RA-02 (433 MHz) at the house** where WiFi/internet exists — the "bridge node" | 🔴 **Replaced LoRaWAN gateway (Sept 2026)** | The ₱10,645 LoRaWAN gateway is gone. The bridge receives float uplinks and pushes them into the Firebase webapp; queued commands are pulled from the webapp and sent back down over LoRa — plain point-to-point radio on the licence-free 433 MHz ISM band (PH), no TTN/ChirpStack server. ~1–3 km line-of-sight over open water at +20 dBm, with SF10–12 stretching the margin. If the pond ends up farther or obstructed, swap in the EBYTE E22-900M22S (915 MHz, +22 dBm) — same SPI wiring. |
| Aerator | **2 × 12V DC electromagnetic air pump, ≥ 30–60 L/min, ≥ 4 outlets each** (NOT 2× mini aquarium pumps) | 🔴 **Corrected** | Mini pumps (~0.5–2 L/min each) cannot aerate 8 cages. Validated sizing: ~1–2 L/min per 100 L water; 8 cages × ~40 L ≈ 320 L → need **≥ 8 outlets at ~2 L/min each ≈ 16–20 L/min**; 2 pumps give N+1 redundancy. Pumps alternate duty. |
| Air stones / tubing | 8–10 × air stones + 20 m 4 mm silicone tubing + gang valve manifold | ✅ Added | Distributes air from the 2 pumps into each of the 8 cages. |
| Dynamic Salinity Pump | **2 × 12V Marine Bilge Pump 1100 GPH** (submersible) | ✅ OK | Corrects salinity drops (e.g., after heavy rain) by mixing brine/high-salinity water; draws ~3–5 A ≤ relay 10 A ✔ |
| Relay Module | **8-channel**, 12V coil, optocoupler, low-level trigger | ✅ Chosen | 2 air pumps + 2 bilge pumps = 4 loads, so a 4-ch board would leave zero expansion; 8-ch covers every actuator path plus spares. |
| Camera | Overhead, one unit initially (optional, ESP32-CAM) — **first thing to cut if the build passes ₱50,000** | ✅ OK | Monitor molting, deter predators, spot poachers. ⚠️ WiFi-only — see BOM Compatibility #3. |
| Control Interface | **Local rotary selector (AUTO / OFF / MANUAL)** + status LEDs at the controller box, plus webapp control | ✅ Added Sept 2026 | The LoRa link is polled point-to-point — a command lands only on the next uplink — so a local selector is the fail-safe for a life-support load. Mode and pump state are echoed in every uplink. |
| Dashboard / Webapp | **Next.js + Firebase Realtime Database + Auth**, free-tier hosting (Vercel) | ✅ Added Sept 2026 | Live readings, AUTO/MANUAL/OFF, per-pump overrides, thresholds. RTDB's `onValue()` sync is a natural fit for telemetry: keep live state in one node and trim the history log so it stays inside the free tier. |
| Power Source | **220V AC drop from the household solar system** (array + battery bank **> 600 Ah** + inverter, available 24/7) + **12V 30 A DC supply** on the float | 🔴 **Corrected Sept 2026 — no on-site solar/battery** | The site already sits on a household solar installation with a battery bank over 600 Ah, so panels, MPPT and an on-site battery are unnecessary. Night aeration is a life-support load of ~38W (air pumps) + node 1.5W + bilge duty ≈ 50–60W ≈ **600 Wh/night ≈ 5 A at 12V** — roughly an **8–10× margin** for the house bank. On-site work is only a 30 mA RCD + earthed outdoor AC run + 12V 30 A PSU (see BOM §4). ⚠️ With no on-site battery, the AC drop is the single point of failure. |

### Controller pin map — ESP32 38-pin DevKit (WiFi/boot-safe)

> Validated for the ESP32-WROOM-32 DevKit (Makerlab PH, ₱349). Rules: **no analog on ADC2** (GPIO 0/2/4/13/14/15/25/26/27 stop working as analog while WiFi runs), **no actuators on boot-strapping pins** (GPIO 0, 2, 12 and 15 glitch or break boot; 5 is usable only for outputs that idle HIGH, which the radio NSS below does), **UART0 (GPIO 1/3) kept free** for flashing and logs. The float node never enables WiFi and the house bridge reads no analog, but the map is safe under both configurations.

| Function | GPIO | Why it's safe |
|-----------|------|---------------|
| LoRa RA-02 SCK / MISO / MOSI | 18 / 19 / 23 | Hardware VSPI — independent of WiFi |
| LoRa RA-02 NSS | 5 | Boot-strapping pin, but NSS **idles HIGH (radio deselected)** — safe at boot |
| LoRa RA-02 RST | 14 | Driven after boot; the brief boot-time PWM glitch just resets the radio harmlessly |
| LoRa RA-02 DIO0 (IRQ) | 26 | Idle LOW at boot, no strapping role; WiFi blocks only ADC2 *analog* use, not digital |
| DS18B20 ×2 (shared 1-Wire bus) | 16 | Free digital pin; external 4.7 kΩ pull-up to 3.3 V |
| pH analog output (via voltage divider ≤ 3.3 V) | 34 | **ADC1** — works with WiFi on; input-only pin |
| EC/salinity analog output | 35 | **ADC1** — input-only |
| DO sensor analog output | 36 (VP) | **ADC1** — input-only |
| AC-fail detection module | 39 (VN) | Digital input; input-only |
| Relay 1 — air pump 1 | 25 | Non-strapping output; firmware drives HIGH (relay OFF) before init |
| Relay 2 — air pump 2 | 27 | Same (digital output — the ADC2 restriction affects analog reads only) |
| Relay 3 — bilge pump 1 | 33 | Non-strapping output; same boot practice |
| Relay 4 — bilge pump 2 | 13 | ADC2 pin, but only its **analog** function is WiFi-blocked — digital output is unaffected |
| Rotary selector bit A (AUTO/MANUAL) | 4 | Input with internal pull-up; selector shorts to GND |
| Rotary selector bit B (AUTO/OFF) | 21 | Input with internal pull-up; selector shorts to GND |
| Mode LED (external) | 22 | Output |
| Link/status LED | 2 (onboard LED) | The devkit's boot LED doubles as the link indicator |

**Leave unconnected:** GPIO 0 (boot strapping — low at reset = flash mode), GPIO 12 (HIGH at boot = flash-voltage fault → boot failure), GPIO 15 (boot strapping), GPIO 1/3 (USB serial). Relay modules are low-level trigger: if a channel chatters at boot, add a 10 kΩ pull-up from its input to 3.3 V. The house bridge reuses the same SPI wiring (18/19/23/5/14/26).

---

## Validated Water-Quality Targets (deployment reference)

From **SEAFDEC/AQD (Philippines)** mud crab culture standards:

| Parameter | Target / Safe Range | Sensor |
|-----------|--------------------|--------|
| Dissolved Oxygen | **≥ 5 ppm** | DFRobot SEN0237-A galvanic DO probe |
| Salinity | **28–32 ppt** (grow-out tolerates 10–34 ppt) | DFRobot EC K=10 (0–100 mS/cm ↔ 0–~66 ppt) |
| Temperature | **27–30 °C** | 2× DS18B20 stainless waterproof probes |
| pH | **7.5–8.5** | E-201-C electrode + PH-4502C board |
| NH₃ (unionized) | ≤ 0.1 ppm — **no affordable sensor**; estimate from pH + temperature (documented academic method) | derived |
