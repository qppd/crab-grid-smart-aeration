# Components List

**Project:** Solar Automated LoRaWAN-Integrated Mud Crab Aeration Network with Crab Grid Observation

> Every count and specification below was **validated against web sources on Sept 2026** — SEAFDEC/AQD (Philippines) mud crab culture standards, FAO mud crab manual, DFRobot sensor datasheets, and Shopee/Lazada PH listings. Rationale for each correction is stated inline; see [`docs/BOM.md`](docs/BOM.md) for the verified product links.

---

## Physical Structure

| Component | Spec / Count | Validated? | Purpose / Notes |
|-----------|--------------|-----------|-----------------|
| Floating Cages | **8 × individual crab fattening boxes** (plastic, with slots/holes for water exchange) | ✅ Correct count | 1 crab per box = anti-cannibalism, matching commercial practice (individual boxes/cages are the standard mud crab fattening method). Mud crabs tolerate crowding **in water** — the constraint is separation, not volume, so 8 units is adequate for a 500 sqm prototype. |
| Cage size (target) | ≥ 45 × 40 × 35 cm water-filled, e.g. Allied box (457 × 406 × 406 mm) | ✅ Sized | Holds a 0.5–1.5 kg fattening crab without contact with hot plastic walls; FAO/industry boxes range from 31 × 26 × 10 cm up to 458 × 406 × 354 mm. |
| Floaters | **Ready-made foam-filled pontoon floats** (HDPE/LLDPE shell, one-piece foam-filled) — **8–12 units**, NOT hollow PVC pipes | 🔴 **Corrected** | Buoyancy math: full cage + water + crab ≈ 25–30 kg → needs ≥ 50 kg buoyancy per cage (2:1 safety). A **sealed hollow 4″ PVC pipe** gives only ~2.6 kg/m — it would sink. Foam-filled pontoons provide 50–100+ kg each, are puncture-proof (foam cannot leak out), and are UV/stab-resistant. PVC **soft piping** remains for the aeration grid only. |
| Cage frames / gantry | Bamboo or marine-plywood grid + shaded frame | ✅ OK | The fattening boxes themselves are plastic, so they are shaded under the frame and kept in water contact; gantry/contact surfaces use non-plastic materials so no bare plastic sits in direct sun. |
| Water circulation | Air-driven: airline manifold delivering air stones into each cage | ✅ Refined | Aerators double as circulators — bubbling creates movement inside each cage. |

## Electronics & Hardware

| Component | Spec / Count | Validated? | Purpose / Notes |
|-----------|--------------|-----------|-----------------|
| Controller | Heltec WiFi LoRa 32 V3 (ESP32-S3 + SX1262), centered in the setup, IP65 enclosure | ✅ OK | Runs automation logic; links sensors, pumps, and the LoRaWAN link. |
| Water Quality Sensors | 4 sensor types: EC/salinity (K=10), pH, dissolved oxygen, temperature (2× DS18B20 probes) | ✅ Validated ranges | **EC K=10 (0–100 mS/cm)** covers seawater (35 ppt ≈ 53 mS/cm) ✔ · **pH E-201-C** covers 7.5–8.5 brackish target ✔ · **DO galvanic probe (SEN0237-A)** covers ≥ 5 ppm DO requirement ✔ · DS18B20 covers 27–30 °C band ✔ |
| LoRaWAN Gateway | RAK7268 WisGate Edge Lite 2 (AS923, 8-channel) — installed at the house where internet exists, **not** on the float | ⚠️ Placement note | Off-grid remote monitoring & pump control from a dashboard. LoRa spans ~1–10 km; the camera also rides on this placement (WiFi only). |
| Aerator | **2 × 12V DC electromagnetic air pump, ≥ 30–60 L/min, ≥ 4 outlets each** (NOT 2× mini aquarium pumps) | 🔴 **Corrected** | Mini pumps (~0.5–2 L/min each) cannot aerate 8 cages. Validated sizing: ~1–2 L/min per 100 L water; 8 cages × ~40 L ≈ 320 L → need **≥ 8 outlets at ~2 L/min each ≈ 16–20 L/min**; 2 pumps give N+1 redundancy. Pumps alternate duty. |
| Air stones / tubing | 8–10 × air stones + 20 m 4 mm silicone tube + gang valve manifold | ✅ Added | Distributes air from the 2 pumps into each of the 8 cages. |
| Dynamic Salinity Pump | **2 × 12V Marine Bilge Pump 1100 GPH** (submersible) | ✅ OK | Corrects salinity drops (e.g., after heavy rain) by mixing brine/high-salinity water; draws ~3–5 A ≤ relay 10 A ✔ |
| Relay Module | **8-channel**, 12V coil, optocoupler, low-level trigger | ✅ Chosen | 2 air pumps + 2 bilge pumps = 4 loads, so a 4-ch board would leave zero expansion; 8-ch covers every actuator path plus spares. |
| Camera | Overhead, one unit initially (optional, ESP32-CAM) | ✅ OK | Monitor molting, deter predators, spot poachers. ⚠️ WiFi-only — see BOM Compatibility #3. |
| Power Source | **220V AC drop from the household solar system** (array + battery bank **> 600 Ah** + inverter, available 24/7) + **12V 30 A DC supply** on the float | 🔴 **Corrected Sept 2026 — no on-site solar/battery** | The site already sits on a household solar installation with a battery bank over 600 Ah, so panels, MPPT and an on-site battery are unnecessary. Night aeration is a life-support load of ~38W (air pumps) + node 1.5W + bilge duty ≈ 50–60W ≈ **600 Wh/night ≈ 5 A at 12V** — roughly an **8–10× margin** for the house bank. On-site work is only a 30 mA RCD + earthed outdoor AC run + 12V 30 A PSU (see BOM §4). ⚠️ With no on-site battery, the AC drop is the single point of failure. |

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
