# CRAB_GRID_SMART_AERATION — Audit History (2026-09)

All findings below were addressed in the September 2026 commits. This file is kept as historical record; current state is in README.md, Components.md, and docs/BOM.md.

---

## What Was Found & Fixed

### Scientific Corrections
- **Buoyancy:** Removed false "submerged cage wet weight = 25-30 kg" claim. Water in slotted cages is neutrally buoyant; size pontoons from dry above-water load.
- **Aeration:** Recalculated biological target to ~4.8-6.4 L/min total (not the misleading 16-20 L/min). Added requirement to measure and regulate per-cage flow.
- **Ammonia:** Corrected misconception that pH + temperature alone calculate NH₃ concentration. Now states TAN or lab test is required.
- **EC/salinity:** Fixed erroneous "12.88 mS/cm = 35 ppt". Added explicit PSS-78 conversion and second calibration standard (1.413 mS/cm).
- **Brine dosing:** Documented that open-water salinity correction in slotted cages is not reliable due to tidal flushing.

### Electrical Safety
- Added licensed-electrician sign-off requirement for 220V AC drop.
- Specified weatherproof IP67/IP68 terminations, drip loops, monthly RCD test.
- Clarified AC-fail GPIO cannot transmit after mains loss; house bridge alarms on heartbeat loss.

### Protocol & Compliance
- Upgraded from 2-byte raw payload to ID+sequence+CRC-8+ACK/NAK packet format.
- Added 60-second heartbeat, 5-minute MANUAL auto-revert (was 30 min).
- Changed frequency from 433 MHz to 915 MHz (Philippine NTC licence-free SRD band).
- Updated module from RA-02/SX1278 to E22-900M22S/SX1262.
- Changed library from sandeepmistry/LoRa to RadioLib (SX1262 requires BUSY pin, not DIO0).

### Pin Map Updates
- LoRa NSS: GPIO 5 (unchanged)
- LoRa RST: GPIO 14 (unchanged)
- LoRa IRQ: GPIO 26 (DIO0 → DIO1, new for SX1262)
- LoRa BUSY: GPIO 17 (new for SX1262, not present on SX1278)
- Analog sensors: ADC1 pins 34/35/36 (unchanged)
- Rotary selector: GPIO 4/21 with external 10kΩ pull-ups

### Budget Adjustment
- Added second EC calibration standard (#24): ₱150-300 → ₱300-600
- Revised totals: Tier 1 ₱32,350-44,900; Full set ₱45,750-59,500

---

## Files Modified

- README.md — title, descriptions, badge links, hardware setup steps, roadmap
- Components.md — floaters, frames, sensors, LoRa node, pin map, quality targets
- docs/BOM.md — module specs, regulatory status, calibration standards, budget totals
- references/Concept-Questions.md — stale terminology cleanup

## Commits

- `99730b0` — docs-audit: reconcile all scientific, electrical, and regulatory findings
- `6fd2810` — Switch to 915 MHz LoRa primary (EBYTE E22-900M22S / SX1262)
- `11b43e8` — docs-final: fix stale AI-isms and audit-report footers
