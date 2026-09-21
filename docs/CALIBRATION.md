# Sensor Calibration Guide

**Project:** Solar Automated LoRa-Integrated Mud Crab Aeration Network with Crab Grid Observation

Calibrate before trusting any reading, and log every calibration. Calibration constants are stored in NVS/EEPROM by the firmware ([`FIRMWARE.md`](FIRMWARE.md) §7) so a reboot never loses them.

## 1. Schedule

| Task | Frequency |
|------|-----------|
| pH two-point calibration | Before deployment, then every 2–4 weeks, after every probe cleaning |
| EC two-point calibration | Before deployment, monthly, after probe cleaning |
| DO single-point calibration | Before each deployment |
| DO membrane cap + electrolyte service | Monthly or on drift ([`BOM.md`](BOM.md) #25) |
| Temperature cross-check | Before deployment, monthly |
| Refractometer zero check | Weekly and before every salinity cross-check |
| RCD test button | Monthly (electrical, shore end) |

## 2. pH (E-201-C + PH-4502C)

1. Rinse the probe with distilled water and blot (never wipe) dry.
2. Calibrate with **4.01** then **6.86** buffers at pond temperature; follow the PH-4502C offset procedure and the `DFRobot_pH` two-point flow.
3. Acceptance: reading within ±0.05 pH of each buffer; slope check fails if the probe is dried out or contaminated.
4. Store the electrode in **KCl storage solution** ([`BOM.md`](BOM.md) #26) — never dry, never distilled water; the probe dies in weeks otherwise.
5. If using the 10 kΩ/18 kΩ divider, repeat the calibration after any hardware change to the divider or the ADC path; an ADS1115 path is preferred for professional-grade resolution.

## 3. EC / Salinity (DFRobot K=10)

1. Two-point calibration with **1.413 mS/cm** and **12.88 mS/cm** standards per the DFRobot K=10 procedure.
2. **12.88 mS/cm is a conductivity calibration standard, not 35 ppt seawater.** At 25 °C, 35 ppt seawater is approximately 53 mS/cm.
3. Convert calibrated EC to salinity with the manufacturer/PSS-78 method; apply temperature compensation.
4. Cross-check the converted salinity against the handheld refractometer ([`BOM.md`](BOM.md) #27) at deployment and after any rain event.

## 4. Dissolved Oxygen (SEN0237-A)

1. Service the membrane cap and electrolyte first if due; air bubbles under the membrane invalidate readings.
2. Warm up and polarize as the datasheet requires, then **single-point calibrate in water-saturated air**.
3. Provide gentle flow across the membrane during readings — stagnant water reads low.
4. Cross-check against a handheld DO meter; keep spare membrane caps and electrolyte at the bench ([`TESTING.md`](TESTING.md) SEN-04).
5. One DO probe is a single point of failure: schedule aeration as the documented fallback while the probe is serviced.

## 5. Temperature (DS18B20 ×2)

1. Cross-check both probes against a reference thermometer in the same water bath ([`TESTING.md`](TESTING.md) SEN-01).
2. A reading of **−127 °C means a wiring or pull-up fault** — see [`TROUBLESHOOTING.md`](TROUBLESHOOTING.md) §4.

## 6. Refractometer

1. Zero against distilled water at ambient temperature before use; ATC handles small drift, not bad zeros.
2. Use it as the ground truth for the EC sensor's salinity conversion — never the reverse.

## 7. Calibration Log Template

| Date | Sensor | Standards/buffers used | Offset/slope | Cross-check result | By |
|------|--------|------------------------|--------------|--------------------|----|
| | | | | | |

The dashboard shows calibration due dates when `config/calibration` is filled in ([`APP.md`](APP.md) §1 and §2).
