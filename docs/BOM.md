# BOM — Solar Automated LoRaWAN-Integrated Mud Crab Aeration Network with Crab Grid Observation

> **Sources:** Shopee PH / Lazada PH / official stores. Links showing a rating were verified live via web search on Sept 2026. Rows that read "check listing", "electrical supply" or "any hardware/pet store" are **unverified category searches** — check the rating (target ≥ 4.7★) before ordering. Ratings shown = product or shop rating from the listing; all are **≥ 4.7★** or from shops with **thousands of ratings / >1k followers**.
> Prices in **PHP**, rounded, subject to seller promotions.
>
> **Validated against:** SEAFDEC/AQD mud crab culture standards (water-quality targets), the FAO mud crab manual (cage/fattening practice), DFRobot datasheets (sensor ranges), and aquaculture aeration/buoyancy sizing rules. 🔴 marks a spec that changed from the original concept, with the reason in the same cell.

---

## 1. Boards & Connectivity

| # | Item | Qty | Est. Price | Rating (verified) | URL | Why compatible |
|---|------|-----|-----------|-------------------|-----|----------------|
| 1 | **Heltec WiFi LoRa 32 V3** (ESP32-S3 + SX1262, 0.96" OLED) — main controller node | 2 (1 main + 1 spare/dev) | ₱1,974 ea | Lazada 4.9★ (25,384 ratings on Heltec shop) | [Shopee listing](https://shopee.ph/Heltec-WiFi-LoRa-32-V3-Development-Board-SX1262-0.96-Inch-OLED-Display-BT-WIFI-Lora-Kit-for-Arduino-IOT-Meshtastic-LoRaWAN-i.1252464771.54157827789) · [Lazada search](https://www.lazada.com.ph/catalog/?q=heltec+lora+32+v3) | ESP32-S3 reads all analog/digital water sensors; SX1262 covers AS923 frequencies (PH band) in firmware; 3.3V logic — sensor boards below output ≤3.4V ✔ |
| 2 | **RAK7268 WisGate Edge Lite 2** LoRaWAN gateway (AS923) | 1 | ₱10,645 | Circuitrocks — top PH electronics shop (98–100% chat response, thousands of orders) | [Shopee — Circuitrocks](https://shopee.ph/Circuitrocks-Lora-Lorawan-Gateway-Wisgate-Edge-Lite-As923-Rakwireless-i.20469516.14668739279) · [RAK official](https://store.rakwireless.com/products/rak7268-8-channel-indoor-lorawan-gateway) | Native **AS923** support = PH LoRaWAN region ✔; 8-channel. **Placement: at the house, where internet exists — not on the float** — it needs mains/WiFi backhaul; LoRa spans 1–10 km from the pond |
| 3 | **ESP32-CAM** (OV2640, WiFi/BT) — overhead camera node | 1 | ₱649 | 4.9★ (41 ratings) | [Shopee listing](https://shopee.ph/ESP32-CAM-with-Integrated-USB-OV2640-GCS2145-Camera-No-Downloader-Module-Needed-ESP32-CAM-Arduino-i.929264657.50814166054) | ⚠️ Streams over WiFi only — see **Compatibility Notes #3** |
| 4 | **8-Channel Relay Module, 12V coil, optocoupler** — drives air pumps + bilge pumps | 1 | ₱273 | 4.8★ (12,201 ratings) | [Shopee listing](https://shopee.ph/DC-12V-8-Channel-Relay-Module-with-Optocoupler-Isolation-PLC-Control-Relay-Output-8-Way-Relay-Module-for-Arduino-i.266699902.19079723116) | 🔴 **8-ch, not 4-ch**: 2 air pumps + 2 bilge pumps = 4 loads, so a 4-ch board leaves no spare channels for cage valves or a second DO stage; 10A contacts ≥ pump inrush ✔; choose **low-level trigger** for ESP32 3.3V GPIO |

## 2. Water Quality Sensors

| # | Item | Qty | Est. Price | Rating (verified) | URL | Measures |
|---|------|-----|-----------|-------------------|-----|----------|
| 5 | **DFRobot Gravity Analog EC/Salinity Sensor (K=10)** — seawater-range probe | 1 | ₱5,489 | 4.9★ (DFRobot official store, >1k followers) | [Shopee — DFRobot official](https://shopee.ph/DFRobot-Analog-Electrical-Conductivity-Sensor-Meter-i.18252381.9084450334) · [Datasheet](https://wiki.dfrobot.com/dfr0300-h/) | **Salinity** — range **0–100 mS/cm ≈ 0–66 ppt** covers 28–32 ppt target and even full seawater (35 ppt ≈ 53 mS/cm) ✔ |
| 6 | **pH Electrode E-201-C + PH-4502C analog board** | 1 | ₱1,395 | 4.9★ (22 ratings) · probe alone 4.8★ (53,991) | [Shopee — kit](https://shopee.ph/pH-sensor-PH-4502C-kit-for-Arduino-Analog-Output-E201-BNC-probe-i.64815518.1583999627) · [Shopee — spare probe](https://shopee.ph/pH-Electrode-Sensor-BNC-Connector-Probe-E-201-pH-Composite-electrode-E-201C-i.52451576.28361135684) | **pH** (brackish target 7.5–8.5 per SEAFDEC) |
| 7 | **Waterproof DS18B20 temperature probe (stainless)** | 2 | ₱79 ea | 4.7★ (3,179 ratings) | [Shopee listing](https://shopee.ph/BEOHESP-DS18B20-Sensor-Stainless-Steel-Waterproof-Temperature-Sensor-1-3-5meters-Probe-Temperature-Probe-i.1231138459.57712308409) | **Temperature** (27–30 °C band; digital 1-Wire, no ADC needed) |
| 8 | **DFRobot Gravity Analog Dissolved Oxygen Kit (SEN0237-A)** | 1 | ₱12,000–13,199 | 4.9★ (53 ratings, DFRobot official store) | [Shopee — DFRobot official](https://shopee.ph/DFRobot-Gravity-Analog-Dissolved-Oxygen-Sensor-Meter-Kit-For-Arduino-i.18252381.2379265111) · [Makerlab PH ₱12,000](https://makerlab.ph/products/gravity-analog-dissolved-oxygen-sensor-meter-kit-for-arduino) | **Dissolved oxygen** — SEAFDEC target **≥ 5 ppm**; DO < 3 ppm is lethal; galvanic probe, no polarization |

> **Targets:** the SEAFDEC/AQD ranges these sensors were chosen against are tabulated in [`Components.md`](../Components.md). The one gap is unionized NH₃ (≤ 0.1 ppm) — no affordable sensor exists, so it is computed from pH + temperature (cite the method in the paper).
>
> **Start with two sensors:** buy **#5 (EC/salinity)** + **#7 (DS18B20)** first (₱5,647), then add pH and DO as budget allows. Salinity + temperature drive the automated first-aid loop (rain-dilution scenario).

## 3. Actuators (Aeration + Pumps)

| # | Item | Qty | Est. Price | Rating (verified) | URL | Purpose |
|---|------|-----|-----------|-------------------|-----|---------|
| 9 | **12V DC electromagnetic air pump, ≥ 30–60 L/min, ≥ 4 outlets** (aerator path) | 2 | ~₱850–1,200 ea | unverified — check ≥ 4.7★ | [Shopee search: 12V air pump](https://shopee.ph/search?keyword=12v%20air%20pump%20aquarium) · [Solar-DC 30W class](https://shopee.ph/list/solar%20air%20pump) | 🔴 **Do not substitute mini pumps** (~0.5–2 L/min cannot aerate 8 cages). Sizing: ~1–2 L/min per 100 L; 8 cages × ~40 L ≈ 320 L → **≥ 8 outlets at ~2 L/min ≈ 16–20 L/min total**, split across 2 pumps (N+1 redundancy, alternating duty; ~25–38W combined). Pump ratings are open-flow, so a 30 L/min-class unit delivers well under that through stones — that headroom is intentional, not spare capacity |
| 10 | **Air stones ×8–10 + 4 mm airline tubing (20 m) + gang valve manifold** | 1 set | ~₱300 | any hardware/pet store | [Shopee search: air stone](https://shopee.ph/search?keyword=air%20stone%20aquarium) | One stone per cage; manifold balances flow so every cage gets ~2 L/min |
| 11 | **12V Marine Bilge Pump 1100 GPH** (submersible) — dynamic salinity pump | 2 | ₱649 ea | 4.8★ (100% chat response) | [Shopee listing](https://shopee.ph/12v-1100gph-Automatic-Submersible-Non-Automatic-Marine-Electric-Bilge-Pump-Submersible-Boat-Bilge-Water-Pump-For-Ponds-Pools-Spas-Silent-Boat-Caravan-RV-Submersible-i.385158564.53853831269) · cheaper alt 4.6★/294: [₱338](https://shopee.ph/1100GPH-12v-Submersible-Water-Pump-with-Switch-for-Boat-Automatic-Submersible-Small-Boat-Bilge-Pump-i.1768137647.45160954686) | Matches doc spec (2× 12V OR 1100 GPH bilge) — mixes brine/high-salinity water during rain events; draws ~3–5A ≤ relay 10A ✔ |

## 4. Power System (220V AC from Household Solar)

> 🔴 **Changed Sept 2026 — no on-site solar/battery.** The deployment is wired from the house, whose existing solar installation (array + battery bank **> 600 Ah** + inverter) supplies **220V AC 24/7**. Panels, MPPT and on-site battery are therefore **not purchased**; the on-site work is a protected AC drop plus a 12V DC supply for the pumps, relays and controller. The old LVD (battery deep-discharge protection) goes away with the battery. The deferred off-grid variant is kept at the end of this section for reference.

| # | Item | Qty | Est. Price | Rating (verified) | URL | Sizing rationale |
|---|------|-----|-----------|-------------------|-----|------------------|
| 12 | **Outdoor-rated 220V AC drop** (3-core, 2.5 mm², earthed; ~30–100 m) | 1 | ₱1,500–2,500 | electrical supply; buy outdoor-rated | [Shopee search: outdoor extension cord](https://shopee.ph/search?keyword=outdoor%20extension%20cord%20heavy%20duty) | 220V × ~0.5 A ≈ 100 W → voltage drop is negligible even at 100 m on 2.5 mm² (<1%). Route overhead or in conduit with a drip loop; never along or below the waterline |
| 13 | **RCD/GFCI 30 mA + 2-pole breaker in a small enclosure** | 1 | ₱800–1,200 | electrical supply | [Shopee search: 30mA RCD breaker](https://shopee.ph/search?keyword=rcd%2030ma%20circuit%20breaker) | 🔴 **New — mandatory for 220V within reach of water.** A 30 mA RCD trips well before a lethal shock and a 6–10 A breaker protects the run. Put the isolation switch at the **house** end so the float can be killed from dry ground |
| 14 | **12V DC switching PSU, 12V 30 A (360 W)** — replaces panels + MPPT + battery | 1 | ₱600–900 | ≥4.7★ (thousands of ratings) | [Shopee search: 12V 30A power supply](https://shopee.ph/search?keyword=12v%2030a%20power%20supply) | Peak load ≈ 100 W (2 × 38 W air pumps + 2 × 30 W bilge + node/camera) ≈ **8.4 A** → 30 A gives ~3.5× headroom for pump inrush; 15 A is the practical minimum. Mount above the waterline in the IP65/66 enclosure (#18, glands #22) |

> 🔌 **If there is ever no household supply** (pond relocation): add 2 × 100W panels + MPPT 20A + 12V 200Ah LiFePO4 (~₱30,000) — that sizing (600 Wh/night ≈ 50 Ah → 2+ nights) is already validated.
>
> ⚠️ **Single point of failure:** with no on-site battery, a tripped breaker, cut cable or inverter fault stops aeration — a life-support load. Alert on loss-of-heartbeat (no uplink ≈ power or link down) and keep the house-end breaker reachable; the AC-fail module (#20) makes it an immediate alarm. A small 12 V battery + charger (~₱3–5k) as a pump-only UPS covers unattended nights.

## 5. Structure & Materials

| # | Item | Qty | Est. Price | Rating (verified) | URL | Notes |
|---|------|-----|-----------|-------------------|-----|-------|
| 15 | **Allied Crab Fattening Box** (individual compartments) | 8 | ₱399 ea = ₱3,192 | 4.7★ (370 ratings) | [Shopee listing](https://shopee.ph/Allied-Crab-Fattening-Box-Cage-Indoor-Crab-Farming-i.411519694.28807834583) | ✅ **Count verified (8) and spec fits:** 457 × 406 × 406 mm holds a 0.5–1.5 kg crab; 1 crab per box = anti-cannibalism ✔; drill slots for water exchange; shade covers needed outdoors (note #6) |
| 16 | 🔴 **Foam-filled pontoon floats** (HDPE/LLDPE, ready-made fish-cage floats) | 8–12 | ~₱400–800 ea = ₱3,500–6,000 | source from local fish-cage suppliers / Shopee-Lazada | [Lazada search: floating pontoon](https://www.lazada.com.ph/tag/plastic-floating-pontoon/) · [Shopee search: fish cage float](https://shopee.ph/search?keyword=fish%20cage%20float) | 🔴 **Not hollow PVC pipes.** Buoyancy math: submerged cage+water+crab ≈ 25–30 kg each → ≥ 50 kg buoyancy per cage (2:1 safety). Sealed hollow 4″ PVC gives ~2.6 kg/m and would sink. Foam-filled floats: 50–100+ kg each, puncture-proof (foam can't leak), UV-resistant — the industry standard for PH fish cages |
| 17 | **Bamboo / marine-plywood frame + netting + shade cloth** (cage grid, non-plastic contact surfaces) | — | ₱2,000–4,000 | — | [Lazada search: HDPE net](https://www.lazada.com.ph/catalog/?q=hdpe+netting) | Avoid bare plastic in direct sun: the boxes are shaded and stay in water contact, and the frame/gantry is bamboo or marine plywood |
| 18 | Misc: air tubing, gang valves, waterproof enclosures (IP65), cables, fuses, epoxy, SS bolts, mooring rope | — | ₱2,500–3,500 | — | [Shopee search: IP65 enclosure](https://shopee.ph/search?keyword=ip65%20waterproof%20enclosure) | Electronics must be sealed — brackish mangrove environment |

---

## 6. Electrical Protection, Calibration & Consumables

> The items other sections depend on but are easy to forget: the camera's 12V→5V buck, calibration standards (pH and EC), DO probe membranes, pH electrode storage fluid, and the salt reserve the brine-mixing scenario actually pumps. Ratings verified as described in the header.

| # | Item | Qty | Est. Price | Rating (verified) | URL | Why needed |
|---|------|-----|-----------|-------------------|-----|------------|
| 19 | **12V→5V 3A buck converter** (camera node power) | 1 | ₱56–120 | 4.8★ | [Shopee listing](https://shopee.ph/DC-6-24V-12V/24V-to-5V-3A-CAR-USB-Charger-Module-DC-Buck-step-down-Converter-5V-power-supply-module-i.1137847711.24079849445) | Feeds the ESP32-CAM 5V rail from the 12 V supply ✔ — required by Compatibility #2 |
| 20 | **220V AC opto-isolated detection module** (mains-fail input to a GPIO) | 1 | ₱100–250 | 4.8★ | [Shopee search: AC 220V optocoupler detection module](https://shopee.ph/search?keyword=220v%20ac%20opto%20isolation%20detection%20module) | 🔴 **Replaces the LVD, which is obsolete with no on-site battery.** The AC drop is now the single point of failure for a life-support load — this gives the firmware a mains-present GPIO so it can raise a LoRaWAN alarm and log the outage the moment the house supply drops |
| 21 | **DC blade fuse + holder set** (main ~15A, controller ~5A, pumps ~10A) | 1 set | ₱50–150 | check listing; buy ≥ 4.7★ | [Shopee search: blade fuse holder](https://shopee.ph/search?keyword=blade%20fuse%20holder%20waterproof) | A 12 V supply feeding unprotected pump wiring on a float is a melt/fire risk. One main fuse, plus per-branch fuses sized to each load |
| 22 | **PG7/IP68 nylon cable glands** (10-pc kit) | 1 kit | ₱78–120 | 4.8★ (9,383 ratings) | [Shopee listing](https://shopee.ph/Nylon-Cable-Gland-10pcs.-PG7-~-PG63-IP68-Waterproof-Connector-Durable-Plastic-Cable-Fitting-i.1468298505.40724321556) | Item 18 buys the IP65 enclosure but no feedthroughs — sensor/pump cables through plain drilled holes destroy the IP rating |
| 23 | **pH buffer calibration set 4.01 / 6.86 / 9.18** | 1 set | ₱275 | see listing | [Shopee listing](https://shopee.ph/pH-Calibration-Solution-pH-4.01-6.86-9.18-Buffer-for-pH-Meter-Hydroponics-Lab-Professional-Grade-i.911703494.48855977022) | PH-4502C is factory-offset only — 2-point calibration (4.01 + 6.86) is required before any pH reading is meaningful; re-calibrate every 2–4 weeks |
| 24 | **EC calibration standard 12.88 mS/cm** | 1 bottle | ~₱150–300 | check listing; buy ≥ 4.7★ | [Shopee search: EC 12.88 mS/cm solution](https://shopee.ph/search?keyword=ec%20calibration%20solution%2012.88) | DFRobot EC K=10 firmware requires calibration; 12.88 mS/cm ≈ 35 ppt sits exactly at the seawater point that matters for this project |
| 25 | **DO probe spare membrane caps + electrolyte refill** (SEN0237-A consumables) | 1 kit | ~₱500–1,000 | check DFRobot store | [DFRobot wiki (maintenance)](https://wiki.dfrobot.com/sen0237-a/) · [Shopee search](https://shopee.ph/search?keyword=dissolved%20oxygen%20probe%20membrane) | Galvanic DO probes consume membrane caps + electrolyte — without spares the ₱12k sensor goes dead mid-thesis |
| 26 | **pH electrode storage solution (KCl)** | 1 bottle | ~₱150–300 | check listing; buy ≥ 4.7★ | [Shopee search: KCl storage solution](https://shopee.ph/search?keyword=ph%20electrode%20storage%20solution%20kcl) | The E-201-C electrode dies in weeks if stored dry or in distilled water — KCl storage is what makes the ₱1,395 probe last the project |
| 27 | **Handheld salinity refractometer 0–100 ppt, ATC** | 1 | ~₱500–800 | 4.7★ (47,728 ratings) | [Shopee listing](https://shopee.ph/Salinity-Refractometer-For-Seawater-And-Marine-Fishkeeping-Aquarium-0-100-Ppt-With-Automatic-Temperature-Compensation-i.119376804.27414109823) | Ground-truth cross-check for the EC sensor during calibration and for verifying brine mixing after rain events — no power needed |
| 28 | **Brine reserve: rock/feed-grade salt ~25 kg + sealed 100–120 L drum** | 1 set | ~₱1,400–1,700 | check listing | [Shopee search: rock salt](https://shopee.ph/search?keyword=rock%20salt%20feed%20grade) | The salinity-correction scenario pumps "brine" — nothing in the BOM stored or made it. 25 kg salt → ~90 L saturated brine (≈26 wt%) in a drum staged beside the float, dosed by the bilge pumps |

> Dev-time only (not system hardware): a USB-TTL adapter (~₱80) to flash the ESP32-CAM if you don't already own one.

---

## ✅ Compatibility Verification (system-level)

| # | Check | Verdict |
|---|-------|---------|
| 1 | **LoRa band (Philippines = AS923)** | ✅ RAK7268 variant sold is AS923; Heltec V3 SX1262 radio spans 915–923 MHz — select **AS923** region in firmware (TTN/ChirpStack) |
| 2 | **Voltage chain** | ✅ 220V AC (household solar, 24/7) → 30 mA RCD + breaker (#13) → **12V 30 A PSU (#14)** → 12V pumps/relays directly; 12V→5V buck (#19) for the camera node; Heltec V3 onboard 3.3V LDO. No on-site panel/MPPT/battery. Earthed outdoor run with the RCD at the house end — the safety requirement for 220V at the pond |
| 3 | **Camera connectivity** | ⚠️ ESP32-CAM uses **WiFi, not LoRaWAN** (video can't fit LoRa bandwidth). Options: (a) farmer connects phone to camera AP during pond visits, (b) put camera on the gateway-side building where internet exists, (c) upgrade to an LTE camera — budget decision for the panel |
| 4 | **Ammonia sensing gap** | ⚠️ No affordable verified NH₃ sensor on Shopee. Industry option: DFRobot RS485 NH₄⁺ sensor (~$209 / ~₱12k, [dfrobot.com](https://www.dfrobot.com/blog-20760.html)). **Thesis workaround:** estimate NH₃ from pH + temperature (common academic method — cite it in the paper) |
| 5 | **Sensor ADC levels** | ⚠️ PH-4502C outputs up to ~5V with 2.5V offset → add a **voltage divider** before ESP32 ADC (Heltec V3 pins are 3.3V max). DFRobot EC/DO boards output ≤3.4V → direct ✔ |
| 6 | **Overheating** | ✅ Boxes are shaded under the frame and stay in water contact; frame/gantry surfaces are non-plastic, so no bare plastic sits in direct sun |
| 7 | **Load vs. relay & supply** | ✅ Bilge pump 3–5A, air pump <2A each — all ≤ 10A per relay channel; 8-ch module covers all loads + spares. Peak total ≈ 8.4 A draws on the 12V 30 A PSU (~3.5× headroom) |
| 8 | **Aeration capacity** | ✅ 8 cages ≈ 320 L total → ≥ 16–20 L/min air with ≥ 8 outlets; 2 × 30–60 L/min pumps exceed this with N+1 redundancy |
| 9 | **Buoyancy** | ✅ Foam-filled pontoons: ≥ 50 kg net buoyancy per cage (2:1 safety factor over the ~25–30 kg wet load); hollow PVC would fail — corrected |
| 10 | **Night operation (life-support, doc requirement)** | ✅ Night draw ~50–60 W (≈ 5 A at 12 V) is supplied 24/7 by the household solar system (**> 600 Ah** bank + inverter) over the AC drop — the house bank alone covers ~8–10 nights of aeration with no sun. ⚠️ No on-site battery → alert on uplink loss and see the §4 mitigation |

## ⚡ Power Budget (validated)

| Load | Watts | Duty | Wh/day |
|------|-------|------|--------|
| Heltec V3 controller node | ~1.5 | 24 h | 36 |
| Air pumps (aeration, staged 1–2 running) | ~38 max | 16–24 h (incl. night) | ~600 worst case |
| Bilge pumps (salinity events) | ~60 | intermittent (~1 h/day avg) | ~60 |
| Gateway (RAK7268, at house w/ mains) | 6 | — (house supply) | 0 |
| ESP32-CAM node | ~1.5 | visits only | ~10 |
| **Total on-site DC load** | | | **≈ 430–700 Wh/day** |
| **On-site supply: 12V 30 A PSU (#14)** | 360 W | — | peak ≈ 100 W ≈ 8.4 A → ~3.5× headroom for inrush ✔ |
| **Supply: household solar → 220V AC** | array + **> 600 Ah** bank + inverter | 24/7 | ≈ 5.8 kWh usable vs ~0.6 kWh/night → **~8–10× margin** ✔ |
| **Night draw (life-support)** | ~50–60 W ≈ **5 A @ 12 V** | 12 h | **≈ 600 Wh** — supplied by the house bank; inverter adds ~15% on the AC side |

## 💰 Budget Summary

| Category | Est. Cost |
|----------|----------|
| Boards & connectivity (Heltec ×2, gateway, ESP32-CAM, 8-ch relay) | ₱15,500 |
| Sensors (Tier 1: EC + 2× DS18B20 → full set incl. pH + DO) | ₱5,650 → ₱19,000–20,250 |
| Aeration + pumps (2× 12V 30–60 L/min air pumps, air stones/manifold, 2× bilge) | ₱2,700–4,000 |
| Power — AC drop + protection + 12V supply (outdoor AC run, RCD/breaker, 12V 30 A PSU) | ₱2,900–4,600 |
| Structure & materials (boxes ×8, foam-filled pontoons ×8–12, frame, misc) | ₱11,200–16,700 |
| Protection, calibration & consumables (§6: buck, AC-fail detect, fuses, glands, buffers, EC standard, DO membranes, KCl, refractometer, brine reserve) | ₱3,150–4,750 |
| **TOTAL (Tier 1 sensors)** | **≈ ₱41,000–51,000** |
| **TOTAL (all sensors)** | **≈ ₱54,500–65,800** |

> Category figures are ranges; each total is the sum of its low and high ends.

> **Which items belong to which phase** (per-phase costs are in the README's BOM & Budget section): ① bench — Heltec ×2 (#1), DS18B20 (#7), relay (#4), one bilge pump (#11) · ② dashboard — gateway (#2), EC sensor (#5), refractometer (#27), EC standard (#24) · ③ power to the float — AC drop (#12), RCD (#13), 12V 30A PSU (#14), buck (#19), fuses (#21), glands (#22) · ④ full feature set — DO (#8), pH (#6), camera (#3), boxes (#15), pontoons (#16), air pumps and stones (#9–10), brine reserve (#28), DO membranes (#25), KCl (#26).
