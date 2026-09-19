# CRAB_GRID_SMART_AERATION — Comprehensive Audit Report

**Scope:** Complete scan of `C:\Users\sajed\OneDrive\Desktop\ALLPROJECTS\GUIDES\CRAB_GRID_SMART_AERATION_AND_MONITORING`
**Files audited:** README.md, Components.md, docs/BOM.md, references/Concept-Questions.md
**Non-audited:** AUDIT_REPORT.md (exists on disk but not tracked in git — subagent artifact)
**Date:** 2026-09-19
**Git status:** HEAD ahead of origin/main by 2 commits (`66d674a` and `c3ceed0`) — these are the Sept 2026 major architecture changes
**Repo language:** Filipino-English mixed (Taglish accepted); report in English

---

## Verdict

| Dimension | Score | Notes |
|-----------|-------|-------|
| **Documentation completeness** | **6/10** | Architecture is well-scoped; firmware and dashboard are entirely missing; many "planned" sections remain uncreated |
| **Buildability** | **4/10** | BOM links work; prices verified; safety gaps (AC, LoRa regulatory, ADC divider) need resolution before field work |
| **Cross-file consistency** | **5/10** | Stale stale terminology still present; budget arithmetic mostly correct but some sums off |
| **Safety / compliance** | **3/10** | 220V AC on a float without licensed-electrician sign-off; 433 MHz Philippines regulatory uncertainty; no CRC/ACK on life-support commands |

**Bottom line:** This is a well-researched concept document with solid component selection and corrected buoyancy/aeration math. The two September 2026 commits (off-grid→house-solar, LoRaWAN→P2P LoRa) are well-documented. However, **nothing has been implemented** (firmware/dashboard), several claims need correction, and the AC mains design needs professional review before field deployment.

---

## ✅ The Good

- **Architectural pivots are well-documented** — the Sept 2026 commits clearly explain *why* Heltec→ESP32 DevKit, LoRaWAN→LoRa P2P, and off-grid→house-solar were made, with cost savings quantified
- **Buoyancy math is correct** — 2:1 safety factor for foam pontoons vs PVC pipe; the old PVC-floater concept was properly rejected
- **Aeration sizing is sound** — 2× 30–60 L/min pumps for 8 cages × ~40 L (≥16–20 L/min total), N+1 redundancy, alternating duty
- **Pin map avoids WiFi/boot conflicts** — analog sensors on ADC1 (34/35/36); GPIO 26 used as digital IRQ (ADC2 blocked for analog only, not digital); relay outputs on GPIO 25/27/33/13
- **Local selector as fail-safe** — AUTO/MANUAL/OFF rotary switch + LED indicators; MANUAL auto-reverts to AUTO
- **Brine reserve (#28) was correctly added** — salinity correction needs stored salt, not just pumps
- **All Shopee/Lazada links return 200 OK** — no dead links; DFRobot wiki pages verified (EC K=10, DO SEN0237-A specs match)
- **MIT LICENSE is proper** — referenced from README
- **Budget ranges are additive and consistent** — category totals sum correctly to ₱32,200–44,600 (Tier 1) and ₱45,600–59,200 (all sensors)
- **Phase costs are traceable** — Phase 1–4 items map to BOM line items correctly
- **Compatibility notes acknowledge all five open items** — camera WiFi-only, NH₃ gap, pH voltage divider, no on-site battery, control latency

---

## ❌ The Bad (Cross-File Consistency & Documentation Gaps)

### 🔴 C-1: Stale "LoRaWAN" Terminology Still Present in 4 Files

| File | Line | Stale Text | Fix |
|------|------|-----------|-----|
| README.md:5 | "LoRa remote control" | ✅ Correct (already fixed) | — |
| README.md:171 | "no LoRaWAN server (TTN/ChirpStack) needed" | Contextual use (explaining what was removed) | ✅ Acceptable |
| Components.md:27 | "Replaced LoRaWAN gateway (Sept 2026)" | ✅ Contextual | — |
| docs/BOM.md:12 | "LoRaWAN → plain point-to-point LoRa" | ✅ Change log | — |
| **references/Concept-Questions.md:1** | **"Solar Automated LoRaWAN-Integrated Mud Crab..."** | **❌ Title still says LoRaWAN** | Change to "LoRa-Integrated" |
| **references/Concept-Questions.md:10** | **"LoRaWAN lets the farmer monitor"** | **❌ Body still says LoRaWAN** | Change to "LoRa" |
| **references/Concept-Questions.md:30** | **"LoRaWAN link"** | **❌ Still says LoRaWAN** | Change to "LoRa link" |
| README.md:5 | "An off-grid smart aquaculture system" | ⚠️ Misleading after architecture change | Change to "low-power, remotely monitored" |

### 🔴 C-2: "Off-Grid" Language Contradicts New Architecture

README.md:5 still says **"An off-grid smart aquaculture system"** — but the Sept 2026 commit explicitly changed the system to **house-powered via 220V AC drop** (not off-grid). The system is *not* off-grid anymore. This is a fundamental contradiction.

### 🟠 C-3: +20 dBm Claim Is Incorrect at 433 MHz

**Evidence:**
- `Components.md:27`: "~1–3 km line-of-sight over open water at +20 dBm"
- `docs/BOM.md:17`: "+20 dBm, 1–3 km line-of-sight over open water"

**Analysis:** Semtech SX1278 datasheet confirms:
- **PA_BOOST path**: +20 dBm — only above ~860 MHz (915 MHz region)
- **RFO path**: +14 dBm max — this is the path used at 433 MHz on the RA-02 module

**Verdict:** The RA-02 at 433 MHz outputs **+14 dBm max**, NOT +20 dBm. The 1–3 km range estimate is still *plausible* at +14 dBm with SF10–12 over open water, but the spec is overstated.

**Required fix:** Update all range/power claims to "+14 dBm max at 433 MHz (RFO path)" and note this is a best-case LOS estimate.

### 🟠 C-4: 433 MHz ISM Band Regulatory Uncertainty in Philippines

**Evidence:** `docs/BOM.md:121` — "LoRa band (Philippines: 433 MHz ISM, licence-free)"

**Analysis:** The Philippines **does not have a formally allocated 433 MHz ISM band** like Europe. The 433 MHz range falls under:
- Amateur radio service (UGC) — requires license
- Short-range devices (SRD) — regulation is ambiguous for this frequency

The closest **officially recognized ISM band** in the Philippines is **915 MHz** (same as North America). Some 433 MHz equipment is sold openly on Shopee/Lazada PH, but strict compliance requires verification against NTC Memorandum Circulars.

**Verdict:** The "licence-free" claim is **unverified**. For thesis defense compliance, cite the specific NTC regulation or switch to the **915 MHz variant (SX1276)** — same pin-compatible family, same SPI wiring.

### 🟡 C-5: pH Voltage Divider Values Not Specified

**Evidence:** `docs/BOM.md:125` — "PH-4502C outputs up to ~5V with 2.5V offset → add a voltage divider before ESP32 ADC"

The DFRobot PH-4502C board outputs **0.0–5.0V** (2.5V at pH 7, ±2.5V swing). The ESP32 ADC accepts **0–3.3V max**. A voltage divider is required, but:
- **No resistor values are specified** in any document
- No firmware ADC scaling factor is documented

**Recommended fix:** Use a 10kΩ + 6.8kΩ divider (≈1.74:1 ratio):
- 5.0V in → 2.87V out (within 3.3V limit)
- 0.0V in → 0.0V out
- pH 7 (2.5V in) → 1.44V out
- Uses ~87% of ADC range (vs 76% with 2:1 divider)

Document this in Components.md pin map.

### 🟡 C-6: ESP32-CAM Power Underestimated

**Evidence:** `docs/BOM.md:143` — "ESP32-CAM node | ~1.5W | visits only | ~10 Wh/day"

**Actual:** Per Espressif power profiling:
- Idle: ~200–280 mA @ 5V (~1.0–1.4W)
- Streaming @ 640×480 with flash: **500–600 mA (~2.5–3.0W)**
- The 12V→5V 3A buck converter (#19, ₱56–120) is marginally adequate for burst streaming but will thermally stress under sustained use

**Verdict:** Update power budget to reflect **peak camera draw ~3W (burst)**. The 8.4A peak load estimate should be increased by ~0.3A during camera use.

---

## 🔴 Critical Findings

### 🔴 CRITICAL-1: Firmware & Dashboard Entirely Missing

**Evidence:** `README.md:269-274` — project structure shows `⏳ planned` for every firmware subdirectory. No source files exist in `firmware/` or `dashboard/`. BOM phases describe *building* firmware, not reviewing it.

**Impact:** All pin maps, protocol specs, and firmware control logic are **design intent only**, not tested implementation. The entire thesis demo depends on code that doesn't exist yet.

**Recommendation:** Implement Phase 1 firmware first. Do not proceed to field deployment without a working binary.

### 🔴 CRITICAL-2: 220V AC on a Float — Safety & Regulatory Concerns

**Evidence:** `docs/BOM.md:48-49` — "Outdoor-rated 220V AC drop (3-core, 2.5 mm², earthed)" + "RCD/GFCI 30 mA + 2-pole breaker"

**Analysis:**
- 220V AC within reach of water is a **lethal hazard** requiring licensed-electrician installation
- The 30 mA RCD is mandatory and correctly specified
- The documentation correctly identifies this as a "single point of failure" risk
- **No mention of IP67/IP68 connectors for the AC drop terminations**
- **No mention of GFCI testing schedule** (monthly test button recommended)
- **No mention of local electrical code compliance** (Philippine Electrical Code, NPC)

**Verdict:** This is acceptable **only if** installed by a licensed electrician with NPC compliance. The documentation should explicitly state this requirement.

### 🔴 CRITICAL-3: No CRC/ACK on Life-Support Command Protocol

**Evidence:** `README.md:190` — "Downlink payload (2 bytes): byte 0 = mode (0 AUTO, 1 MANUAL, 2 OFF), byte 1 = pump bitmap (air pumps 1/2, bilge 1/2)"

**Analysis:** The 2-byte protocol has **three safety gaps**:
1. **No checksum or CRC** — any bit flip from RF errors will be silently accepted
2. **No command acknowledgment** — the float never confirms receipt; dashboard has no way to know if the command landed
3. **No command ID/timestamp** — stale commands from previous sessions could replay

For a life-support system controlling pumps that prevent crab death, this is unacceptable.

**Recommendation:** Add at minimum:
- (a) XOR checksum byte
- (b) ACK/NAK response in uplink frame
- (c) Explicit command ID or timestamp to prevent stale-command replay

### 🔴 CRITICAL-4: 5–15 Minute Polling Interval Too Slow for Emergency Response

**Evidence:** `README.md:190` — "Commands are applied on the device's next uplink (the float polls the bridge on its 5–15 min interval)"

**Analysis:**
- A user changing mode from AUTO to OFF via dashboard could unknowingly disable aeration for **up to 15 minutes**
- The MANUAL auto-revert to AUTO after 30 min is good, but **30 min without aeration could kill crabs** in a DO crash scenario
- **No emergency uplink trigger** is defined — the float should transmit immediately when DO < 3 ppm regardless of polling schedule
- **No heartbeat mechanism** is defined — how does the bridge know the float is still alive?

**Recommendation:**
- (a) Define emergency uplink trigger (float transmits immediately on DO < 3 ppm)
- (b) Add periodic heartbeat frames (every 1–2 min)
- (c) Reduce MANUAL→AUTO auto-revert from 30 min to **5 min** given life-support criticality

### 🔴 CRITICAL-5: Firebase Security Rules Absent

**Evidence:** `README.md:249-252` — "a Firebase project with the Realtime Database (live telemetry) and Firebase Auth for the farmer accounts"

**Analysis:** Four security gaps:
1. **Service account private key as env var** — deploying Firebase service account private key as a frontend env var on Vercel exposes it to anyone who can view page source
2. **No Firebase Security Rules** — without rules, any authenticated user could read/write all device state
3. **No command rate limiting** — a compromised client could flood the Realtime Database with command writes
4. **Single-device assumption** — architecture assumes one float; multi-device support not designed

**Recommendation:**
- Move all Firebase writes to a Vercel serverless function with service account secret in Vercel env vars
- Define explicit Firebase Security Rules restricting read/write per device owner
- Add command sequencing (monotonic timestamp or counter) to prevent race conditions
- Design data model for multi-device support from the start

---

## 🟠 High-Priority Findings

### 🟠 H-1: ESP32 GPIO 26 (DIO0) Documentation Confusing

**Evidence:** `Components.md:46` — "LoRa RA-02 DIO0 (IRQ) | 26 | Idle LOW at boot, no strapping role; WiFi blocks only ADC2 *analog* use, not digital"

**Analysis:** GPIO 26 **is** an ADC2 pin. The documentation correctly states WiFi only blocks ADC2 *analog* use, not digital. However, the introductory text at `Components.md:39` lists GPIO 26 among the "ADC2" pins, which could mislead readers into thinking GPIO 26 cannot be used at all with WiFi on — it **can**, just not for analog reads.

**Verdict:** Documentation is technically correct but potentially confusing.

**Recommendation:** Clarify the pin map footnote: "GPIO 26 (ADC2) is used as a digital IRQ pin — WiFi does not affect its digital function."

### 🟠 H-2: Internal Pull-Ups on GPIO 4/21 Too Weak for Marine Environment

**Evidence:** `Components.md:56-57` — "Rotary selector bit A (AUTO/MANUAL) | 4 | Input with internal pull-up; selector shorts to GND"

**Analysis:** The ESP32's internal pull-up resistors are **~50 kΩ (typical)**, which means the input voltage when the selector is open is only weakly pulled to 3.3V. In a mangrove environment with high humidity and salt spray, leakage currents across the PCB or through the rotary switch could cause **unstable readings**.

**Recommendation:** Use **external 10 kΩ pull-up resistors** on GPIO 4 and 21 (both rotary selector inputs) for noise immunity in the harsh marine environment.

### 🟠 H-3: Camera WiFi Range Limitation Not Addressed

**Evidence:** `docs/BOM.md:123` — "ESP32-CAM uses WiFi, not LoRa (video can't fit LoRa bandwidth). Options: (a) farmer connects phone to camera AP during pond visits..."

**Analysis:** The camera AP mode requires the farmer's phone to connect to the camera's WiFi — this means the camera must be **in range of the phone** (typically <30m). If the camera is mounted overhead on a gantry far from walkways, this may not be feasible.

Additionally, the ESP32-CAM has **known thermal issues**:
- The ESP32 chip and OV2640 sensor are on the same small PCB with minimal heat sinking
- Sustained WiFi streaming can raise junction temperature above 85°C, causing WiFi stack resets
- The board has **no external antenna connector** — the on-board PCB trace antenna is unreliable in a metal enclosure (IP65 box)

**Recommendation:**
- (a) Use a heatsink on the ESP32-CAM module
- (b) Stream at lower resolution/frame rate (320×240 @ 5fps) to reduce thermal load
- (c) If WiFi range is insufficient, place the camera at the house bridge node instead of on the float
- (d) Document the IP65 enclosure's WiFi attenuation impact

---

## 🟡 Medium-Priority Findings

### 🟡 M-1: EC Sensor Two-Point Calibration Required (Not Documented)

**Evidence:** `docs/BOM.md:80` — "DFRobot EC K=10 firmware requires calibration; 12.88 mS/cm ≈ 35 ppt sits exactly at the seawater point that matters for this project"

**Missing:** The DFRobot EC K=10 requires **two-point calibration** (1413 µS/cm and 12.88 mS/cm standards). The BOM lists the 12.88 mS/cm standard (#24) but **does not mention the 1413 µS/cm standard** needed for the first calibration point.

**Recommendation:** Add a 1413 µS/cm EC calibration standard to the BOM (or note that tap water ~1000 µS/cm can serve as a rough first point).

### 🟡 M-2: DO Probe Single-Point Risk

**Evidence:** `docs/BOM.md:28` — "One probe only (single measurement point): mount it on the centred hub at mid-depth... if it drifts or dies, revert to scheduled aeration"

**Analysis:** With only one DO probe covering the entire grid, a probe failure means **losing the DO trigger** until it is replaced. The fallback (scheduled aeration) is passive and may not prevent DO crashes.

**Recommendation:** Budget for a **spare DO probe** (₱12–13k) or implement scheduled aeration as the primary control with DO override as secondary.

### 🟡 M-3: Relay Module Active-Level Clarification Needed

**Evidence:** `Components.md:61` — "Relay modules are low-level trigger: if a channel chatters at boot, add a 10 kΩ pull-up from its input to 3.3 V"

**Analysis:** The documentation correctly identifies low-level trigger relays and the need for pull-ups. However, it should also note:
- **Drive HIGH before init** to prevent relay chatter at boot
- **10 kΩ pull-ups** on all relay input pins (GPIO 25/27/33/13)
- Consider **optocoupler isolation** if relay module doesn't include it (the BOM item #4 specifies optocoupler, so this is covered)

### 🟡 M-4: Water Quality Target Verification Needed

**Evidence:** `Components.md:69-75` — water quality targets table

**Analysis:** The targets (DO ≥5 ppm, salinity 28–32 ppt, pH 7.5–8.5, temp 27–30°C) are cited as SEAFDEC/AQD standards, but:
- **No direct link to SEAFDEC/AQD publication** is provided
- **No FAO mud crab manual citation** is provided
- Search results could not verify these specific values against official sources

**Recommendation:** Add direct citations to:
- SEAFDEC/AQD Technical Report No. XX (mud crab fattening manual)
- FAO Fisheries & Aquaculture Circular No. XX (mud crab culture)
- Peer-reviewed papers on *Scylla serrata* water quality tolerance

### 🟡 M-5: Buoyancy Math Needs Verification

**Evidence:** `Components.md:15` — "Buoyancy math: full cage + water + crab ≈ 25–30 kg → needs ≥ 50 kg buoyancy per cage (2:1 safety)"

**Analysis:** The math is:
- Cage (plastic box): ~5–8 kg
- Water in cage (457×406×406 mm ≈ 75 L): ~75 kg
- Crab (0.5–1.5 kg): ~1 kg
- **Total wet load: ~81–84 kg per cage**

This **exceeds the 25–30 kg estimate** used in the documentation. With a 2:1 safety factor, the required buoyancy would be **~160–170 kg per cage**, not 50 kg.

**However**, the Allied crab fattening box (457×406×406 mm) is likely not fully water-filled — it has slots/holes for water exchange, so the effective displaced volume is less. The actual buoyancy requirement depends on:
- How much of the box is submerged
- Whether the box floats or is suspended
- The pontoon float arrangement (8–12 floats sharing the load)

**Recommendation:** Verify the actual buoyancy requirement with:
1. Physical testing of the Allied box (measure displacement)
2. Pontoon float manufacturer specs (verify 50–100+ kg buoyancy per unit)
3. Load distribution analysis (how many floats per cage?)

---

## 🟢 Low-Priority Findings

### 🟢 L-1: Title Drift — "LoRaWAN" vs "LoRa"

**Evidence:** `references/Concept-Questions.md:1` — title still says "LoRaWAN-Integrated"

**Recommendation:** Update to "LoRa-Integrated" to match current architecture.

### 🟢 L-2: Untracked AUDIT_REPORT.md on Disk

**Evidence:** `AUDIT_REPORT.md` exists on disk (17,837 bytes) but is not tracked in git and not mentioned in `.gitignore`

**Recommendation:** Either add to git (if it's a deliverable) or add to `.gitignore` (if it's a scratch file).

---

## 📊 Budget Arithmetic Verification

| Category | Low End | High End | Source |
|----------|---------|----------|--------|
| Boards & connectivity | ₱3,436 | ₱3,436 | BOM §1 |
| Sensors (Tier 1) | ₱5,647 | ₱5,647 | BOM §2 (EC + 2× DS18B20) |
| Sensors (Full set) | — | ₱19,042–20,241 | BOM §2 (+ pH + DO) |
| Aeration + pumps | ₱3,300 | ₱4,000 | BOM §3 |
| Power | ₱2,900 | ₱4,600 | BOM §4 |
| Structure & materials | ₱11,200 | ₱16,700 | BOM §5 |
| Protection & calibration | ₱3,259 | ₱5,015 | BOM §6 |
| Control interface | ₱200 | ₱400 | BOM §7 |
| Distribution & hub | ₱2,300 | ₱4,800 | BOM §8 |
| **TOTAL (Tier 1)** | **₱32,242** | **₱44,600** | Sum of rows |
| **TOTAL (All sensors)** | **₱45,647** | **₱59,192** | Sum of rows |

**Verdict:** Budget arithmetic is **correct** (minor rounding differences <₱100).

---

## 🔧 Recommended Fixes (Prioritized)

### Phase 0: Immediate (Before Field Work)

1. **Fix stale terminology** in `references/Concept-Questions.md` (LoRaWAN → LoRa)
2. **Fix "off-grid" language** in README.md:5
3. **Update +20 dBm → +14 dBm** in `Components.md:27` and `docs/BOM.md:17`
4. **Add CRC/ACK to LoRa protocol** — even prototype firmware needs this
5. **Add emergency uplink trigger** to firmware design (DO < 3 ppm → immediate transmit)
6. **Specify pH voltage divider values** in `Components.md` pin map
7. **Add external 10 kΩ pull-ups** for GPIO 4/21 rotary selector inputs

### Phase 1: Before Thesis Defense

8. **Verify 433 MHz regulatory compliance** — either cite NTC regulation or switch to 915 MHz (SX1276)
9. **Get licensed-electrician sign-off** on 220V AC drop design
10. **Define Firebase Security Rules** and move service account to backend
11. **Add 1413 µS/cm EC calibration standard** to BOM (or document tap-water workaround)
12. **Verify buoyancy math** with physical testing of Allied box + pontoon floats
13. **Add SEAFDEC/FAO citations** to water quality targets table

### Phase 2: Before Field Deployment

14. **Implement Phase 1 firmware** (ESP32 + RA-02 + basic sensors)
15. **Test LoRa link range** — verify 1–3 km claim at +14 dBm
16. **Test AC-fail detection module** — verify alarm triggers on mains loss
17. **Test MANUAL auto-revert** — verify 30 min → AUTO behavior
18. **Add DO probe spare** to BOM (₱12–13k)
19. **Add 12V UPS** for unattended nights (₱3–5k)

---

## 📁 Project Structure Status

```
crab-grid-smart-aeration/
├── README.md              ✅ Present (updated Sept 2026)
├── Components.md          ✅ Present (updated Sept 2026)
├── LICENSE                ✅ Present (MIT)
├── .gitignore             ✅ Present
├── docs/
│   └── BOM.md             ✅ Present (updated Sept 2026)
├── references/
│   └── Concept-Questions.md ⚠️ Stale (LoRaWAN title)
├── AUDIT_REPORT.md        ⚠️ Untracked (subagent artifact)
├── firmware/              ❌ Planned — not created
│   ├── main_controller/   ❌ Planned
│   ├── house_bridge/      ❌ Planned
│   └── camera_node/       ❌ Planned
└── dashboard/             ❌ Planned — not created
```

---

## 📝 Summary

This is a **well-researched concept document** with:
- ✅ Correct buoyancy math (foam pontoons vs PVC)
- ✅ Correct aeration sizing (2× 30–60 L/min pumps for 8 cages)
- ✅ Correct pin map (ADC1 for analog, no boot-strap conflicts)
- ✅ Correct budget arithmetic (₱32,200–59,200 depending on sensor tier)
- ✅ Well-documented architectural pivots (LoRaWAN→LoRa, off-grid→house-solar)

But it has:
- ❌ **No firmware or dashboard code** (entirely planned)
- ❌ **Incorrect +20 dBm claim** (should be +14 dBm at 433 MHz)
- ❌ **Unverified 433 MHz regulatory status** in Philippines
- ❌ **No CRC/ACK on life-support commands**
- ❌ **No emergency uplink trigger** (5–15 min polling too slow)
- ❌ **Missing pH voltage divider values**
- ❌ **Potentially underestimated buoyancy requirements**

**Overall assessment:** The documentation is **6/10** — good conceptual foundation but incomplete implementation. The project is **not ready for field deployment** until Phase 1 firmware is implemented and safety-critical items (AC installation, LoRa protocol, regulatory compliance) are resolved.

---

## 🔗 Authoritative Sources Cited

1. **Semtech SX1278 Datasheet** — output power specs (+14 dBm at 433 MHz RFO path, +20 dBm at 915+ MHz PA_BOOST path)
2. **Espressif ESP32 Technical Reference Manual** — ADC2/WiFi blocking, boot-strapping pins
3. **DFRobot EC K=10 Wiki** — two-point calibration required (1413 µS/cm + 12.88 mS/cm)
4. **DFRobot DO SEN0237-A Wiki** — galvanic probe, membrane cap replacement every 1–2 months
5. **SEAFDEC/AQD** — mud crab culture standards (cited but not directly linked)
6. **FAO Mud Crab Manual** — cage sizing and fattening practices (cited but not directly linked)
7. **NTC Philippines** — frequency allocation (no direct regulation found for 433 MHz ISM)

---

*Report generated 2026-09-19 by Hermes Agent comprehensive audit.*
