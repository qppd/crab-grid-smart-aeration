# BOM — Solar Automated LoRa-Integrated Mud Crab Aeration Network with Crab Grid Observation

> **Sources:** Shopee PH / Lazada PH / official stores. Links showing a rating were verified live via web search on Sept 2026. Rows that read "check listing", "electrical supply" or "any hardware/pet store" are **unverified category searches** — check the rating (target ≥ 4.7★) before ordering. Ratings shown = product or shop rating from the listing; all are **≥ 4.7★** or from shops with **thousands of ratings / >1k followers**.
> Prices in **PHP**, rounded, subject to seller promotions.
>
> **Validated against:** SEAFDEC/AQD mud crab culture standards (water-quality targets), the FAO mud crab manual (cage/fattening practice), DFRobot datasheets (sensor ranges), and aquaculture aeration/buoyancy sizing rules. 🔴 marks a spec that changed from the original concept, with the reason in the same cell.

---

## 1. Boards & Connectivity

> 🔴 **Changed Sept 2026 — LoRaWAN → plain point-to-point LoRa, and Heltec → budget ESP32 (₱10,645+ saved).** The RAK7268 LoRaWAN gateway and 2 × Heltec V3 (≈ ₱14,600) are replaced by **3 × ESP32 DevKit (₱349) + 3 × EBYTE E22-900M22S SX1262 (₱389) + 3 × 915 MHz antenna (₱100)** ≈ ₱2,500 for three node sets (float node, house bridge, dev/spare). Same radio physics, no TTN/ChirpStack subscription-server dependency, and every part is stocked at Makerlab PH. Full verified listing: [makerlab.ph](https://makerlab.ph/) (search "esp32", "lora").

| # | Item | Qty | Est. Price | Rating (verified) | URL | Why compatible |
|---|------|-----|-----------|-------------------|-----|----------------|
| 1 | **ESP32 DevKit, 38-pin (ESP32-WROOM-32)** — main controller node (float) | 3 (float node + house bridge + spare/dev) | ₱349 ea | Makerlab PH — in stock | [Makerlab: Type-C ESP32 30/38-pin](https://makerlab.ph/search?q=esp32) · alt ₱350: [30/38-pin board](https://makerlab.ph/search?q=esp32) | WiFi+BT MCU reads all analog/digital water sensors; 3.3V logic — sensor boards below output ≤3.4V ✔; WiFi stays OFF on the float. **Pin map avoids all WiFi (ADC2) and boot-strap conflicts — see [Components.md](../Components.md) §Controller pin map** |
| 2 | **EBYTE E22-900M22S (SX1262, 915 MHz) LoRa module + 915 MHz antenna (₱100 ea)** — one per node: float, house bridge, spare | 3 | ₱489 ea (verify current listing) | Makerlab PH — search "lora" | [Makerlab: LoRa SX1262](https://makerlab.ph/search?q=lora) · [915 MHz antenna](https://makerlab.ph/search?q=lora) | SPI wiring (SCK 18/MISO 19/MOSI 23/NSS 5, RST 14, **DIO1 26, BUSY 17**) — SX1262 needs a BUSY pin, not DIO0. SX1262 delivers up to +22 dBm on the 915 MHz PH licence-free SRD band. Confirm measured EIRP and NTC type-approval before deployment. **3.3V logic only — never apply 5V to the SPI pins.** ⚠️ Never power without an antenna — instant PA damage. Use **RadioLib**'s SX1262 driver (`sandeepmistry/LoRa` does not support SX126x). |
| 3 | **ESP32-CAM** (OV2640, WiFi/BT) — overhead camera node | 1 | ₱649 | Makerlab PH — in stock | [Makerlab: ESP32-CAM OV2640](https://makerlab.ph/search?q=esp32) | ⚠️ Streams over WiFi only — see **Compatibility Notes #3** |
| 4 | **8-Channel Relay Module, 12V coil, optocoupler** — drives air pumps + bilge pumps | 1 | ₱273 | 4.8★ (12,201 ratings) | [Shopee listing](https://shopee.ph/DC-12V-8-Channel-Relay-Module-with-Optocoupler-Isolation-PLC-Control-Relay-Output-8-Way-Relay-Module-for-Arduino-i.266699902.19079723116) | 🔴 **8-ch, not 4-ch**: 2 air pumps + 2 bilge pumps = 4 loads, so a 4-ch board leaves no spare channels for cage valves or a second DO stage; 10A contacts ≥ pump inrush ✔; choose **low-level trigger** for ESP32 3.3V GPIO |

## 2. Water Quality Sensors

| # | Item | Qty | Est. Price | Rating (verified) | URL | Measures |
|---|------|-----|-----------|-------------------|-----|----------|
| 5 | **DFRobot Gravity Analog EC/Salinity Sensor (K=10)** — seawater-range probe | 1 | ₱5,489 | 4.9★ (DFRobot official store, >1k followers) | [Shopee — DFRobot official](https://shopee.ph/DFRobot-Analog-Electrical-Conductivity-Sensor-Meter-i.18252381.9084450334) · [Datasheet](https://wiki.dfrobot.com/dfr0300-h/) | **Conductivity/salinity** — range **0–100 mS/cm** covers the conductivity around seawater (35 ppt is approximately 53 mS/cm at 25 °C). Use the manufacturer/PSS-78 conversion; do not use a fixed 0.66 ppt-per-mS/cm factor. |
| 6 | **pH Electrode E-201-C + PH-4502C analog board** | 1 | ₱1,395 | 4.9★ (22 ratings) · probe alone 4.8★ (53,991) | [Shopee — kit](https://shopee.ph/pH-sensor-PH-4502C-kit-for-Arduino-Analog-Output-E201-BNC-probe-i.64815518.1583999627) · [Shopee — spare probe](https://shopee.ph/pH-Electrode-Sensor-BNC-Connector-Probe-E-201-pH-Composite-electrode-E-201C-i.52451576.28361135684) | **pH** (brackish target 7.5–8.5 per SEAFDEC) |
| 7 | **Waterproof DS18B20 temperature probe (stainless)** | 2 | ₱79 ea | 4.7★ (3,179 ratings) | [Shopee listing](https://shopee.ph/BEOHESP-DS18B20-Sensor-Stainless-Steel-Waterproof-Temperature-Sensor-1-3-5meters-Probe-Temperature-Probe-i.1231138459.57712308409) | **Temperature** (27–30 °C band; digital 1-Wire, no ADC needed) |
| 8 | **DFRobot Gravity Analog Dissolved Oxygen Kit (SEN0237-A)** | 1 probe | ₱12,000–13,199 | 4.9★ (53 ratings, DFRobot official store) | [Shopee — DFRobot official](https://shopee.ph/DFRobot-Gravity-Analog-Dissolved-Oxygen-Sensor-Meter-Kit-For-Arduino-i.18252381.2379265111) · [Makerlab PH ₱12,000](https://makerlab.ph/products/gravity-analog-dissolved-oxygen-sensor-meter-kit-for-arduino) | **Dissolved oxygen** — SEAFDEC target **≥ 5 ppm**; DO < 3 ppm is lethal; galvanic probe, no polarization. **One probe only** (single measurement point): mount it on the centred hub at mid-depth, clear of the aerator plume, and provide gentle flow across the membrane. If it drifts or dies, use a handheld DO meter/scheduled aeration and replace the membrane cap (#25). |

> **Targets:** the SEAFDEC/AQD ranges these sensors were chosen against are tabulated in [`Components.md`](../Components.md). The one gap is unionized NH₃ (≤ 0.1 ppm) — no affordable sensor exists. pH + temperature can estimate the toxic fraction/risk only; an absolute NH₃ concentration requires measured Total Ammonia Nitrogen (TAN) or a laboratory test. |
>
> **Start with two sensors:** buy **#5 (EC/salinity)** + **#7 (DS18B20)** first (₱5,647), then add pH and DO as budget allows. Salinity + temperature drive the automated first-aid loop (rain-dilution scenario).

## 3. Actuators (Aeration + Pumps)

| # | Item | Qty | Est. Price | Rating (verified) | URL | Purpose |
|---|------|-----|-----------|-------------------|-----|---------|
| 9 | **12V DC electromagnetic air pump, 30–60 L/min rated open-flow, ≥ 4 outlets** (aerator path) | 2 | ~₱850–1,200 ea | unverified — check ≥ 4.7★ | [Shopee search: 12V air pump](https://shopee.ph/search?keyword=12v%20air%20pump%20aquarium) · [Solar-DC 30W class](https://shopee.ph/list/solar%20air%20pump) | 🔴 **Do not substitute mini pumps** (~0.5–2 L/min cannot aerate 8 cages). Biological target: 1.5–2 L/min per 100 L; 8 cages × ~40 L ≈ 320 L → **about 4.8–6.4 L/min delivered total**, split across 2 pumps (N+1 redundancy, alternating duty). The 30–60 L/min rating is open-flow headroom for depth, tubing and stone losses; use a manifold, needle valves and per-branch flow measurement so excess air is not dumped into the cages. |
| 10 | **Air stones ×8–10 + 4 mm airline tubing (20 m) + gang valve manifold** | 1 set | ~₱300 | any hardware/pet store | [Shopee search: air stone](https://shopee.ph/search?keyword=air%20stone%20aquarium) | One stone per cage; the manifold balances flow so every cage gets ~2 L/min (the rigid alternative to driving air through the pipe grid, #31) |
| 11 | **12V Marine Bilge Pump 1100 GPH** (submersible) — circulation/flush pump | 2 | ₱649 ea | 4.8★ (100% chat response) | [Shopee listing](https://shopee.ph/12v-1100gph-Automatic-Submersible-Non-Automatic-Marine-Electric-Bilge-Pump-Submersible-Boat-Bilge-Water-Pump-For-Ponds-Pools-Spas-Silent-Boat-Caravan-RV-Submersible-i.385158564.53853831269) · cheaper alt 4.6★/294: [₱338](https://shopee.ph/1100GPH-12v-Submersible-Water-Pump-with-Switch-for-Boat-Automatic-Submersible-Small-Boat-Bilge-Pump-i.1768137647.45160954686) | Provides circulation/flushing and can support a controlled salinity experiment. It does not reliably correct salinity in open slotted cages because tidal exchange flushes the dose and dense brine sinks. Use isolation sleeves or a closed-loop/RAS enclosure for a defensible salinity-control test. Verify actual running/inrush current against the relay and branch fuse. |

## 4. Power System (220V AC from Household Solar)

> 🔴 **Changed Sept 2026 — no on-site solar/battery.** The deployment is wired from the house, whose existing solar installation (array + battery bank **> 600 Ah** + inverter) supplies **220V AC 24/7**. Panels, MPPT and on-site battery are therefore **not purchased**; the on-site work is a protected AC drop plus a 12V DC supply for the pumps, relays and controller. The old LVD (battery deep-discharge protection) goes away with the battery. The deferred off-grid variant is kept at the end of this section for reference.

| # | Item | Qty | Est. Price | Rating (verified) | URL | Sizing rationale |
|---|------|-----|-----------|-------------------|-----|------------------|
| 12 | **Outdoor-rated 220V AC drop** (3-core, 2.5 mm², earthed; ~30–100 m) | 1 | ₱1,500–2,500 | electrical supply; buy outdoor-rated | [Shopee search: outdoor extension cord](https://shopee.ph/search?keyword=outdoor%20extension%20cord%20heavy%20duty) | Planning estimate: 220V × ~0.5 A ≈ 100 W. A licensed electrician must verify the actual load, conductor ampacity, voltage drop, earthing, connector IP rating and Philippine Electrical Code compliance. Route overhead or in conduit with drip loops; use IP67/IP68 weatherproof terminations and no submerged joints. |
| 13 | **RCD/GFCI 30 mA + 2-pole breaker in a small enclosure** | 1 | ₱800–1,200 | electrical supply | [Shopee search: 30mA RCD breaker](https://shopee.ph/search?keyword=rcd%2030ma%20circuit%20breaker) | 🔴 **New — mandatory for 220V within reach of water.** Install the 30 mA RCD and breaker at the house/shore end, before the cable reaches the water. Use weatherproof enclosures/connectors, drip loops, no submerged joints, and a monthly test button. Final breaker coordination and conductor/fuse sizing require a licensed electrician. |
| 14 | **12V DC switching PSU, 12V 30 A (360 W)** — replaces panels + MPPT + battery | 1 | ₱600–900 | ≥4.7★ (thousands of ratings) | [Shopee search: 12V 30A power supply](https://shopee.ph/search?keyword=12v%2030a%20power%20supply) | Planning peak estimate ≈ 100 W (2 × 38 W air pumps + 2 × 30 W bilge + node/camera) ≈ **8.4 A**; verify every pump nameplate, simultaneous duty, inrush, PSU efficiency and voltage drop before finalizing. Treat 30 A as a provisional supply rating, not proof that a 15 A main fuse is correct; size fuses to the conductors and each branch load. Mount above the waterline in the IP65/66 enclosure (#18, glands #22). |

> 🔌 **If there is ever no household supply** (pond relocation): add 2 × 100W panels + MPPT 20A + 12V 200Ah LiFePO4 (~₱30,000) — that sizing (600 Wh/night ≈ 50 Ah → 2+ nights) is already validated.
>
> ⚡ **Check the house inverter before deploying:** it has to stay on 24/7 (an inverter switched off at night means no aeration), be rated well above the ~100 W peak (~300 VA+ is ample, inrush included), and be pure-sine if possible — pump motors run hotter on modified-sine output.
>
> ⚠️ **Single point of failure:** with no on-site battery, a tripped breaker, cut cable or inverter fault stops aeration — a life-support load. Alert on loss-of-heartbeat (no uplink ≈ power or link down) and keep the house-end breaker reachable; the AC-fail module (#20) makes it an immediate alarm. A small 12 V battery + charger (~₱3–5k) as a pump-only UPS covers unattended nights.

## 5. Structure & Materials

| # | Item | Qty | Est. Price | Rating (verified) | URL | Notes |
|---|------|-----|-----------|-------------------|-----|-------|
| 15 | **Allied Crab Fattening Box** (individual compartments) | 8 | ₱399 ea = ₱3,192 | 4.7★ (370 ratings) | [Shopee listing](https://shopee.ph/Allied-Crab-Fattening-Box-Cage-Indoor-Crab-Farming-i.411519694.28807834583) | ✅ **Count verified (8) and spec fits:** 457 × 406 × 406 mm holds a 0.5–1.5 kg crab; 1 crab per box = anti-cannibalism ✔; drill slots for water exchange; shade covers needed outdoors (note #6) |
| 16 | 🔴 **Foam-filled pontoon floats** (HDPE/LLDPE, ready-made fish-cage floats) | 8–12 | ~₱400–800 ea = ₱3,500–6,000 | source from local fish-cage suppliers / Shopee-Lazada | [Lazada search: floating pontoon](https://www.lazada.com.ph/tag/plastic-floating-pontoon/) · [Shopee search: fish cage float](https://shopee.ph/search?keyword=fish%20cage%20float) | 🔴 **Not hollow PVC pipes.** Size from the measured dry above-water load (boxes, frame, pumps, electronics, retained pipe water and crab), not from the weight of water inside a slotted submerged cage. Use manufacturer-rated floats (target ≥50 kg each), verify actual load distribution, and apply at least a 2:1 safety factor. Sealed hollow 4″ PVC gives ~2.6 kg/m and is not primary flotation. The pipe grid is conduit only (see §8). |
| 17 | **UV-stabilized PVC/FRP frame preferred; bamboo / marine-plywood alternative + netting + shade cloth** | — | ₱2,000–4,000 | — | [Lazada search: HDPE net](https://www.lazada.com.ph/catalog/?q=hdpe+netting) | Avoid bare plastic in direct sun: the boxes are shaded and stay in water contact. Raw bamboo and ordinary plywood rot/split in a warm brackish mangrove; if retained, seal all faces/end grain with marine epoxy or PU and make contact details replaceable. Specify Grade 316 stainless or nylon fasteners. The air/water pipe grid (#31) rides on this frame. |
| 18 | Misc: waterproof enclosures (IP65/66), cable, epoxy, Grade 316 stainless/nylon fasteners, mooring rope, cable ties | — | ₱2,500–3,500 | — | [Shopee search: IP65 enclosure](https://shopee.ph/search?keyword=ip65%20waterproof%20enclosure) | Electronics must be sealed — brackish mangrove environment. Use Grade 316 stainless or nylon fasteners; 304 stainless can pit in warm chloride-rich water. Air tubing and the gang valve manifold are #10, fuses are #21, and the pipe grid is §8. |

---

## 6. Electrical Protection, Calibration & Consumables

> The items other sections depend on but are easy to forget: the camera's 12V→5V buck, calibration standards (pH and EC), DO probe membranes, pH electrode storage fluid, and the salt reserve the brine-mixing scenario actually pumps. Ratings verified as described in the header.

| # | Item | Qty | Est. Price | Rating (verified) | URL | Why needed |
|---|------|-----|-----------|-------------------|-----|------------|
| 19 | **12V→5V 3A buck converter** (camera node power) | 1 | ₱56–120 | 4.8★ | [Shopee listing](https://shopee.ph/DC-6-24V-12V/24V-to-5V-3A-CAR-USB-Charger-Module-DC-Buck-step-down-Converter-5V-power-supply-module-i.1137847711.24079849445) | Feeds the ESP32-CAM 5V rail from the 12 V supply ✔ — required by Compatibility #2 |
| 20 | **220V AC opto-isolated detection module** (mains-fail input to a GPIO) | 1 | ₱100–250 | 4.8★ | [Shopee search: AC 220V optocoupler detection module](https://shopee.ph/search?keyword=220v%20ac%20opto%20isolation%20detection%20module) | 🔴 **Replaces the LVD, which is obsolete with no on-site battery.** It can log a mains-present state only while the float electronics remain powered. If the same AC feed fails, the ESP32 dies and cannot transmit; the house bridge must alarm on heartbeat loss. Add a small UPS/supercapacitor only if a last-gasp packet is required. |
| 21 | **DC blade fuse + holder set** (provisional main/controller/pump ratings) | 1 set | ₱50–150 | check listing; buy ≥ 4.7★ | [Shopee search: blade fuse holder](https://shopee.ph/search?keyword=blade%20fuse%20holder%20waterproof) | A 12 V supply feeding unprotected pump wiring on a float is a melt/fire risk. Do not finalize the 15 A/5 A/10 A values from estimates: measure running and inrush current, check conductor ampacity and temperature rise, then size one main fuse and per-branch fuses to the protected conductors and loads. |
| 22 | **PG7/IP68 nylon cable glands** (10-pc kit) | 1 kit | ₱78–120 | 4.8★ (9,383 ratings) | [Shopee listing](https://shopee.ph/Nylon-Cable-Gland-10pcs.-PG7-~-PG63-IP68-Waterproof-Connector-Durable-Plastic-Cable-Fitting-i.1468298505.40724321556) | Item 18 buys the IP65 enclosure but no feedthroughs — sensor/pump cables through plain drilled holes destroy the IP rating |
| 23 | **pH buffer calibration set 4.01 / 6.86 / 9.18** | 1 set | ₱275 | see listing | [Shopee listing](https://shopee.ph/pH-Calibration-Solution-pH-4.01-6.86-9.18-Buffer-for-pH-Meter-Hydroponics-Lab-Professional-Grade-i.911703494.48855977022) | PH-4502C is factory-offset only — 2-point calibration (4.01 + 6.86) is required before any pH reading is meaningful; re-calibrate every 2–4 weeks |
| 24 | **EC calibration standards: 1.413 mS/cm (1413 µS/cm) + 12.88 mS/cm** | 2 bottles | ~₱300–600 | check listing; buy ≥ 4.7★ | [Shopee search: EC 1.413 mS/cm solution](https://shopee.ph/search?keyword=ec%20calibration%20solution%201.413) · [Shopee search: EC 12.88 mS/cm solution](https://shopee.ph/search?keyword=ec%20calibration%20solution%2012.88) | Follow the DFRobot K=10 two-point procedure. 12.88 mS/cm is a conductivity calibration standard, not 35 ppt seawater; convert calibrated EC to salinity with the manufacturer/PSS-78 method. |
| 25 | **DO probe spare membrane caps + electrolyte refill** (SEN0237-A consumables) | 1 kit | ~₱500–1,000 | check DFRobot store | [DFRobot wiki (maintenance)](https://wiki.dfrobot.com/sen0237-a/) · [Shopee search](https://shopee.ph/search?keyword=dissolved%20oxygen%20probe%20membrane) | Galvanic DO probes consume membrane caps + electrolyte — without spares the ₱12k sensor goes dead mid-thesis. Calibrate in water-saturated air (single point) before each deployment, and note that with only one probe a failed membrane means losing the DO trigger until it is replaced |
| 26 | **pH electrode storage solution (KCl)** | 1 bottle | ~₱150–300 | check listing; buy ≥ 4.7★ | [Shopee search: KCl storage solution](https://shopee.ph/search?keyword=ph%20electrode%20storage%20solution%20kcl) | The E-201-C electrode dies in weeks if stored dry or in distilled water — KCl storage is what makes the ₱1,395 probe last the project |
| 27 | **Handheld salinity refractometer 0–100 ppt, ATC** | 1 | ~₱500–800 | 4.7★ (47,728 ratings) | [Shopee listing](https://shopee.ph/Salinity-Refractometer-For-Seawater-And-Marine-Fishkeeping-Aquarium-0-100-Ppt-With-Automatic-Temperature-Compensation-i.119376804.27414109823) | Ground-truth cross-check for the EC sensor during calibration and for verifying brine mixing after rain events — no power needed |
| 28 | **Brine reserve for a controlled salinity experiment only: rock/feed-grade salt ~25 kg + sealed 100–120 L drum** | 1 set | ~₱1,400–1,700 | check listing | [Shopee search: rock salt](https://shopee.ph/search?keyword=rock%20salt%20feed%20grade) | 25 kg salt can make roughly 90 L of saturated brine (about 26 wt%) under typical conditions. Do not present dosing into open slotted cages as reliable salinity correction: tidal flushing and density-driven sinking will remove the dose. Use temporary isolation sleeves or a closed-loop/RAS enclosure, and log tide/current plus pre/post salinity at multiple points. |

> Dev-time only (not system hardware): a USB-TTL adapter (~₱80) to flash the ESP32-CAM if you don't already own one.

---

## 7. Control Interface (Manual + Automatic)

> 🔴 **Added Sept 2026.** Aeration is life-support and the LoRa link is polled point-to-point — a command lands only on the device's next uplink (5–15 min interval). AUTO is therefore the default, and a local selector exists so the pumps never depend on the cloud.

| # | Item | Qty | Est. Price | Rating (verified) | URL | Why needed |
|---|------|-----|-----------|-------------------|-----|------------|
| 29 | **3-position rotary selector (AUTO / OFF / MANUAL)** + knob + panel label | 1 | ₱150–300 | electrical supply | [Shopee search: rotary selector switch](https://shopee.ph/search?keyword=rotary%20selector%20switch%203%20position) | Local fail-safe mode switch — readable at a glance and works with no internet, bridge node or phone. Wires to 2 GPIO inputs with debounce in firmware |
| 30 | **Status LEDs (mode + pump running) + resistors** | 1 set | ₱50–100 | hardware supply | [Shopee search: 5mm LED assortment](https://shopee.ph/search?keyword=5mm%20led%20assorted%20resistor) | Shows the *actual* relay state at the box: which mode is latched and which pump is energised, without opening the dashboard |

**Mode logic (firmware):** AUTO on boot, reset or link loss with aeration ON · MANUAL auto-reverts to AUTO after 30 min without a command · DO below 3 ppm overrides MANUAL and forces aeration on · mode and pump state echoed in every uplink. **Downlink payload (2 bytes):** byte 0 = mode (0 AUTO, 1 MANUAL, 2 OFF), byte 1 = pump bitmap (air pumps 1/2, bilge 1/2).

**Webapp (free tier):** **Firebase Realtime Database** + Firebase Auth, with Next.js on Vercel — no running cost at this scale, and RTDB's JSON tree with `onValue()` listeners suits live telemetry better than a document store here. Keep live state in one node (`devices/<id>/state`) for the UI and append history under a trimmed `logs/<id>/<timestamp>` path, or export CSV, so the thesis still has data to chart. Server env vars: `FIREBASE_DATABASE_URL`, `FIREBASE_PROJECT_ID`, `FIREBASE_CLIENT_EMAIL`, `FIREBASE_PRIVATE_KEY`.

---

## 8. Distribution Network & Sensor Hub

> 🔴 **Added Sept 2026.** The PVC pipe grid carries the air and the water — but **never both in the same lumen**: water floods an air line and stops aeration, while air pockets choke water flow. Keep the air circuit above the waterline or add a drip loop. Remember the pipes are conduit, not flotation — water-filled pipe weighs ~1 kg/L, which the 2:1 buoyancy factor already absorbs.

| # | Item | Qty | Est. Price | Rating (verified) | URL | Why needed |
|---|------|-----|-----------|-------------------|-----|------------|
| 31 | **PVC pipe network, 25 mm (1")** — pipe, tees, elbows, reducers, end caps, 8 × 6–8 mm per-cage outlets + small valves | 1 set | ₱1,500–3,000 | hardware supply | [Shopee search: PVC pipe fittings](https://shopee.ph/search?keyword=pvc%20pipe%20fittings%201%20inch) | Doubles as the air header (drop tubes and stones are #10) and as the water header fed by the bilge pumps. Split the grid into two halves — one per bilge pump — for balanced flow, and add flush ports for salt and algae |
| 32 | **Flexible hose, 10–12 mm, ~20 m** (pump → per-cage outlet) | 1 set | ₱500–1,000 | hardware supply | [Shopee search: vinyl hose 12mm](https://shopee.ph/search?keyword=vinyl%20hose%2012mm) | The flexible alternative to rigid runs: easier to reposition per cage and to spot a blockage |
| 33 | **Sensor hub** — probe holder/bracket, slotted PVC or plastic cage, stainless bolts, cable ties | 1 set | ₱300–800 | hardware supply | [Shopee search: stainless bolts cable ties](https://shopee.ph/search?keyword=stainless%20bolts%20cable%20ties) | Holds the EC/pH/DO/temp probes in one centred, removable cluster at mid-depth — pull it out for calibration and cleaning |

---

## ✅ Compatibility Verification (system-level)

| # | Check | Verdict |
|---|-------|---------|
| 1 | **LoRa band / regulatory status** | ✅ 915 MHz (E22-900M22S / SX1262) — the Philippine NTC licence-free SRD band for low-duty telemetry; both ends on the same band/freq/SF. Confirm maximum EIRP and type-approval before deployment. No TTN/ChirpStack — the dashboard talks to the **house bridge node** over WiFi. |
| 2 | **Voltage chain and mains safety** | ⚠️ 220V AC (household solar, 24/7) → shore-end 30 mA RCD + breaker (#13) → weatherproof feedthroughs → **12V 30 A PSU (#14)** → 12V pumps/relays directly; 12V→5V buck (#19) for the camera node and the ESP32 DevKit 5V pin (onboard 3.3 V LDO feeds the E22-900M22S — never at 5V). No on-site panel/MPPT/battery. Licensed-electrician installation, PEC review, IP67/IP68 connectors, drip loops, no submerged joints, monthly RCD test, and measured fuse/conductor coordination are deployment gates. |
| 3 | **Camera connectivity** | ⚠️ ESP32-CAM uses **WiFi, not LoRa** (video can't fit LoRa bandwidth). Options: (a) farmer connects phone to camera AP during pond visits, (b) put camera at the house with the bridge node where WiFi exists, (c) upgrade to an LTE camera — budget decision for the panel |
| 4 | **Ammonia sensing gap** | ⚠️ No affordable verified NH₃ sensor on Shopee. Industry option: DFRobot RS485 NH₄⁺ sensor (~$209 / ~₱12k, [dfrobot.com](https://www.dfrobot.com/blog-20760.html)). **Thesis workaround:** pH + temperature can estimate the toxic fraction/risk only; an absolute NH₃ concentration requires measured Total Ammonia Nitrogen (TAN) or a laboratory test. |
| 5 | **Sensor ADC levels** | ⚠️ PH-4502C outputs up to ~5V with 2.5V offset → use 10 kΩ series + 18 kΩ shunt (5 V → about 3.21 V) or an ADS1115 I2C ADC with 3.3 V level shifting. DFRobot EC/DO boards output ≤3.4V → direct ✔ |
| 6 | **Overheating** | ✅ Boxes are shaded under the frame and stay in water contact; frame/gantry surfaces are non-plastic, so no bare plastic sits in direct sun |
| 7 | **Load vs. relay & supply** | ✅ Bilge pump 3–5A, air pump <2A each — all ≤ 10A per relay channel; 8-ch module covers all loads + spares. Peak total ≈ 8.4 A drawn from the 12V 30 A PSU (~3.5× headroom) |
| 8 | **Aeration capacity** | ⚠️ 8 cages ≈ 320 L total → target approximately 4.8–6.4 L/min delivered (1.5–2 L/min per 100 L). The 2 × 30–60 L/min pumps are open-flow rated and may be oversized; use valves/manifolds and measure each branch. N+1 redundancy remains valid. |
| 9 | **Buoyancy** | ⚠️ Use manufacturer-rated foam-filled pontoons (target ≥50 kg each), but size them from the measured dry above-water load and load distribution. Water inside a slotted submerged cage is not a dead load; physically load-test the assembled grid and apply at least a 2:1 safety factor. Hollow PVC is not primary flotation. |
| 10 | **Night operation (life-support, doc requirement)** | ⚠️ Planning draw ~50–60 W (≈ 5 A at 12 V), but verify actual duty cycle, inverter efficiency and battery usable capacity. The house bank must stay on 24/7. No on-site battery → alert on heartbeat loss; add a small UPS/supercapacitor only if last-gasp telemetry or unattended-night runtime is required. |
| 11 | **Control path** | ⚠️ AUTO is the default with a local selector as fail-safe (§7); MANUAL auto-reverts after 5 minutes. Normal command latency may be one uplink interval (5–15 min), but emergency sensor events must transmit immediately. Use device ID, sequence, CRC-8, expiry and ACK/NAK. |
| 12 | **Distribution & buoyancy** | ✅ Separate air and water circuits in the pipe grid with the air kept above the waterline, per-cage valved outlets and flush ports (§8); water-filled pipe (~1 kg/L) still inside the 2:1 buoyancy factor |
| 13 | **ESP32 pin conflicts (WiFi + boot)** | ✅ **Validated — [Components.md](../Components.md) §Controller pin map.** No analog on ADC2 while WiFi runs: all three analog sensors on **ADC1** (34/35/36) ✔. GPIO 4/21 selector inputs and used relay inputs need external 10 kΩ pull-ups in the marine enclosure. No loads on boot-strap pins (0/2/5/12/15): NSS=5 idles HIGH, RST=14 glitch is a harmless radio reset, GPIO 0/12/15 left unconnected ✔. UART0 (1/3) free for flashing ✔. Relays are low-level trigger → drive HIGH before init and confirm the module's input pull-up behavior. |

## ⚡ Power Budget (validated)

| Load | Watts | Duty | Wh/day |
|------|-------|------|--------|
| ESP32 + E22-900M22S controller node | ~1.5 | 24 h | 36 |
| Air pumps (aeration, staged 1–2 running) | ~38 max | 16–24 h (incl. night) | ~600 worst case |
| Bilge pumps (salinity events) | ~60 | intermittent (~1 h/day avg) | ~60 |
| House bridge node (ESP32 + E22-900M22S, at house w/ mains) | ~1.5 | 24 h | 0 (house supply) |
| ESP32-CAM node | ~1.5 | visits only | ~10 |
| **Total on-site DC load** | | | **≈ 430–700 Wh/day** |
| **On-site supply: 12V 30 A PSU (#14)** | 360 W | — | peak ≈ 100 W ≈ 8.4 A → ~3.5× headroom for inrush ✔ |
| **Supply: household solar → 220V AC** | array + **> 600 Ah** bank + inverter | 24/7 | ≈ 5.8 kWh usable vs ~0.6 kWh/night → **~8–10× margin** ✔ |
| **Night draw (life-support)** | ~50–60 W ≈ **5 A @ 12 V** | 12 h | **≈ 600 Wh** — supplied by the house bank; inverter adds ~15% on the AC side |

## 💰 Budget Summary

| Category | Est. Cost |
|----------|----------|
| Boards & connectivity (3× ESP32, 3× E22-900M22S+antenna, ESP32-CAM, 8-ch relay) | ₱3,436 |
| Sensors (Tier 1: EC + 2× DS18B20 → full set incl. pH + DO) | ₱5,647 → ₱19,042–20,241 |
| Aeration + pumps (2× 12V 30–60 L/min air pumps, air stones/manifold, 2× bilge) | ₱3,300–4,000 |
| Power — AC drop + protection + 12V supply (outdoor AC run, RCD/breaker, 12V 30 A PSU) | ₱2,900–4,600 |
| Structure & materials (boxes ×8, foam-filled pontoons ×8–12, frame, misc) | ₱11,200–16,700 |
| Protection, calibration & consumables (§6: buck, AC-fail detect, fuses, glands, buffers, EC standards, DO membranes, KCl, refractometer, brine reserve) | ₱3,409–5,315 |
| Control interface (§7: AUTO/OFF/MANUAL selector + status LEDs) | ₱200–400 |
| Distribution & sensor hub (§8: PVC air/water grid, per-cage outlets, hose, probe holder) | ₱2,300–4,800 |
| **TOTAL (Tier 1 sensors)** | **≈ ₱32,350–44,900** |
| **TOTAL (all sensors)** | **≈ ₱45,750–59,500** |

> Category figures are ranges; each total is the sum of its low and high ends. The webapp itself costs ₱0 on free tiers (Firebase Spark + Vercel Hobby).
>
> 💰 **₱50,000 hardware ceiling:** the camera (#3) and its buck (#19) are the designated first cut (₱705–769) if the build runs over — it is the only optional subsystem. The DO kit (₱12–13k) is what pushes the full set past the ceiling, so it belongs in the last phase; the DO probe and the aerators are never cut.
>
> **Where the money actually goes:** solar generation costs ₱0 here — the household already supplies it, so the power category is only the AC drop, its protection and the 12 V supply (₱2,900–4,600). Two items now dominate: the DO kit (#8) at ₱12,000–13,199 and the EC sensor (#5) at ₱5,489 — together ₱17,500–18,700. The old #1 spend, the LoRaWAN gateway (₱10,645), is gone: the entire radio stack (3 × ESP32 + E22-900M22S + antenna) now costs ~₱2,500. The high end of every range assumes the most expensive listing; the low ends are real listings linked in each row.

> **Which items belong to which phase** (per-phase costs are in the README's BOM & Budget section): ① bench — ESP32 ×2 (#1) + E22-900M22S/antenna ×2 (#2) — the float node plus a bench partner node — DS18B20 (#7), relay module (#4), one bilge pump (#11), selector + LEDs (#29–30) · ② dashboard — house bridge node (1 × ESP32 #1 + E22-900M22S/antenna #2), EC sensor (#5), refractometer (#27), EC standards (#24) · ③ power to the float — AC drop (#12), RCD (#13), 12V 30A PSU (#14), AC-fail module (#20), buck (#19), fuses (#21), glands (#22) · ④ full feature set — DO (#8), pH (#6) + buffers (#23), camera (#3), second bilge pump (#11), boxes (#15), frame + netting + shade (#17), pontoons (#16), misc hardware (#18), pipe grid + hose + sensor hub (#31–33), air pumps and stones (#9–10), brine reserve (#28), DO membranes (#25), KCl (#26).

### 🎯 ₱50,000-capped build

Apply these cuts in order until the running total is under ₱50,000:

| Order | Cut | Saves | What you give up |
|-------|-----|-------|------------------|
| 1 | Camera #3 + its buck #19 | ₱705–769 | Overhead monitoring → visual checks during pond visits. Nothing else depends on it |
| 2 | DO kit #8 + membranes #25 | ₱12,500–14,199 | The DO trigger. Until the probe is bought, run night aeration on a fixed schedule (e.g. 30 min on / 30 min off) and keep the temperature + salinity logic |
| 3 | Dev/spare node set (1 × ESP32 #1 + E22-900M22S/antenna #2) | ₱838 | No hot spare if the controller dies mid-defense |
| 4 | Second bilge pump #11 | ~₱649 | Two-zone salinity correction becomes single-zone (one grid half) |
| 5 | pH kit #6 + buffers #23 | ₱1,670 | pH monitoring; the NH₃ estimate loses its pH input |

**With cuts 1–2 applied** (the configuration that fits the ceiling): boards ₱2,787 · sensors ₱7,042 · aeration ₱3,298–3,998 · power ₱2,900–4,600 · structure ₱11,192–16,692 · protection & calibration ₱2,853–4,195 · control interface ₱200–400 · distribution & hub ₱2,300–4,800 → **≈ ₱32,550–44,500**.

**With cuts 1–5 applied:** **≈ ₱29,300–41,100** (the ₱338 1100 GPH bilge listing, 4.6★, becomes the single pump).

> The high ends assume the most expensive listing on every line, so the cap is held by **sourcing** as much as by scope: buy the low-end listings linked in each row and re-check the running total before each phase.

**Never cut:** the air pumps and stones (#9–10), the power chain and its protection (#12–14, #20–22), the boxes and pontoons (#15–16), the EC sensor (#5 — it is the rain-dilution trigger) and the brine reserve (#28). These keep the crabs alive and drive the automated first-aid demo.
