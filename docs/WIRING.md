# Wiring and Assembly Guide

**Project:** Solar Automated LoRa-Integrated Mud Crab Aeration Network with Crab Grid Observation

This document is the physical build reference: assembly order, the verified ESP32 38-pin pin map (WiFi- and boot-safe), subsystem wiring tables, and the pre-energize checklist. Electrical ratings and product links live in [`HARDWARE.md`](HARDWARE.md) and [`BOM.md`](BOM.md).

## 1. Safety Gates

1. The 220 V AC drop, RCD/breaker, and earthing are installed and commissioned by a **licensed electrician** (Philippine Electrical Code). Builders work on 12 V DC only.
2. Never power a LoRa node without its **antenna** attached — instant PA damage.
3. Weatherproof everything: IP65/66 enclosures, IP68 cable glands, drip loops, **no submerged joints**.
4. Never test pumps in a pond with crabs stocked; use buckets on the bench.

## 2. Assembly Order

| Step | Task | Reference |
|------|------|-----------|
| 1 | Bench-assemble the controller node: ESP32 + E22 on jumper wires first, verify uplink with a second node | [`TESTING.md`](TESTING.md) LNK-01 |
| 2 | Wire the relay module and selector on the bench; verify the three positions decode correctly | §5, §6 below |
| 3 | Mount ESP32, relay module, PSU area, and fuse box inside the IP65 enclosure; glands for every cable exit | §7 |
| 4 | Build the pontoon grid: 4 round foam floats (50 × 90 cm), frame/gantry, 8 slotted fattening boxes, shading | [`Components.md`](Components.md) — Physical Structure |
| 5 | Lay the PVC air header and water header as **separate circuits**; air circuit above the waterline or drip-looped | [`Components.md`](Components.md) — Air/water routing |
| 6 | Build the sensor hub: centred, removable probe cluster at mid-depth (~20–30 cm) | [`Components.md`](Components.md) — Sensor hub |
| 7 | Electrician: shore-end 30 mA RCD + 2-pole breaker, earthed 2.5 mm² outdoor run to the float | §8 |
| 8 | Commissioning: rails check, calibration, then the [`TESTING.md`](TESTING.md) matrix | — |

## 3. Verified ESP32 38-Pin Pin Map (float node)

Every pin below was checked against the three ESP32-WROOM-32 constraint classes: **ADC2 pins lose analog function while WiFi runs**, **boot-strapping pins (0, 2, 5, 12, 15) must not be loaded at reset**, and **UART0 (1, 3) must stay free for flashing/logs**. The float node runs WiFi-off, but the map is safe even if WiFi is later enabled, because **all three analog sensors sit on ADC1**.

| Function | GPIO | WiFi/boot verification |
|----------|------|------------------------|
| LoRa E22 SCK / MISO / MOSI | 18 / 19 / 23 | Hardware VSPI; independent of WiFi. Safe |
| LoRa E22 NSS | 5 | Strapping pin, but NSS idles HIGH (radio deselected) at reset. Safe |
| LoRa E22 RST | 14 | Driven after boot; the boot-time PWM glitch only resets the radio. Digital-only use |
| LoRa E22 DIO1 (IRQ) | 26 | SX1262 interrupt line; digital use only. Safe |
| LoRa E22 BUSY | 17 | SX1262 mandatory line; plain digital pin. Safe |
| DS18B20 ×2 (shared 1-Wire) | 16 | Free digital pin; external 4.7 kΩ pull-up to 3.3 V |
| pH analog output | 34 | **ADC1** — works with WiFi on; input-only (no internal pull-up: the sensor board drives it) |
| EC/salinity analog output | 35 | **ADC1** — input-only |
| DO analog output | 36 (VP) | **ADC1** — input-only |
| Relay 1 — air pump 1 | 25 | Digital output; ADC2 analog function unused, so WiFi-safe |
| Relay 2 — air pump 2 | 27 | Digital output; same rule |
| Relay 3 — bilge pump 1 | 33 | Digital output; non-strapping. Safe |
| Relay 4 — bilge pump 2 | 13 | Digital output; ADC2 analog unused. Non-strapping. Safe |
| Selector bit A (AUTO/MANUAL) | 4 | Digital input; ADC2 analog unused. External 10 kΩ pull-up, selector shorts to GND |
| Selector bit B (AUTO/OFF) | 21 | Digital input; not an ADC pin. External 10 kΩ pull-up |
| Mode LED | 22 | Output; non-strapping. Safe |
| Link/status LED | 2 | Onboard LED; strapping pin but loaded only after boot by the board itself. Safe as used |
| Spare (input) | 39 (VN) | Input-only, ADC1 — reserved for a future sensor |
| Spare | 32 | ADC1-capable/digital — reserved |

**Deliberately unconnected:** GPIO 0 (boot-mode strap), GPIO 12 (must be LOW at reset — HIGH causes flash-voltage boot failure), GPIO 15 (strap), GPIO 1/3 (UART0 flashing and logs). GPIO 6–11 are internal flash and are not exposed.

**Result: no pin conflicts.** No analog on ADC2, no actuator on a strapping pin, UART0 free, and every relay is a plain digital output.

## 4. Pin-Rule Summary (why the map is conflict-free)

| Rule | How the map complies |
|------|----------------------|
| No analog reads on ADC2 while WiFi runs | All analog sensors on 34/35/36 (ADC1). Pins 4/13/14/25/26/27 are digital-only |
| No loads on boot-strapping pins 0/2/12/15 | 0/12/15 unconnected; 2 carries only the onboard LED after boot |
| GPIO 5 strap tolerance | Used as NSS which idles HIGH — the accepted safe case |
| UART0 free | 1/3 unconnected; logs and flashing unaffected |
| Low-level trigger relays must not chatter at boot | GPIOs are high-Z at reset, so **every used relay input gets an external 10 kΩ pull-up to 3.3 V**; firmware also writes relay pins HIGH before pin init |

## 5. Relay and Actuator Wiring

| Relay channel | GPIO | Load | Notes |
|---------------|------|------|-------|
| 1 | 25 | RESUN MPQ-03 air pump 1 (12 V, ~3.5 A) | 10 A contact rating; branch fuse per [`BOM.md`](BOM.md) #21 |
| 2 | 27 | RESUN MPQ-03 air pump 2 (N+1 standby) | Alternating duty set in firmware |
| 3 | 33 | 12 V bilge pump 1 (1100 GPH) | Water header, grid half A |
| 4 | 13 | 12 V bilge pump 2 (1100 GPH) | Water header, grid half B |

Relay module: 12 V coil, optocoupler, **low-level trigger**. JD-VCC jumper: power the relay coil from 12 V (or 5 V per module spec), keep the optocoupler side on the ESP32's 3.3 V/5 V logic rail — never back-feed 12 V into a GPIO.

## 6. Mode Selector Wiring

3-position rotary switch, 2 poles, each position **shorts the corresponding GPIO to GND** (external 10 kΩ pull-ups to 3.3 V). Decode is active-LOW:

| Position | GPIO 4 (A) | GPIO 21 (B) | Firmware decodes as |
|----------|-----------|-------------|---------------------|
| AUTO | 1 (HIGH) | 1 (HIGH) | AUTO |
| MANUAL | 0 (LOW) | 1 (HIGH) | MANUAL |
| OFF | 0 (LOW) | 0 (LOW) | OFF |

Rule: B LOW → OFF; else A LOW → MANUAL; else AUTO. OFF exists for servicing only — never while crabs are stocked.

## 7. Signal Wiring Table

| From | To | Conductor | Notes |
|------|----|-----------|-------|
| pH board analog out | GPIO 34 | Shielded 2-core | Through 10 kΩ series + 18 kΩ shunt divider (5 V → ~3.21 V), or an ADS1115 I2C ADC for professional-grade resolution |
| EC board analog out | GPIO 35 | Shielded 2-core | Output ≤ 3.4 V — direct to ADC1 |
| DO board analog out | GPIO 36 | Shielded 2-core | Output ≤ 3.4 V — direct to ADC1 |
| DS18B20 ×2 | GPIO 16 | Shielded 3-core | Parallel on one 1-Wire bus; 4.7 kΩ pull-up to 3.3 V at the hub |
| E22 EBYTE | GPIO 18/19/23/5/14/26/17 | Short ribbon/jumpers | 3.3 V logic only — never 5 V on SPI pins |
| All sensor boards | Common GND | — | Single-point ground star at the PSU negative |

Every cable leaving the enclosure passes an IP68 gland; probe cables get drip loops so water never wicks into the box.

## 8. Power Wiring

| Segment | Spec | Notes |
|---------|------|-------|
| House → float | 220 V AC, 3-core 2.5 mm² earthed outdoor run | Overhead or conduit; **shore-end 30 mA RCD + 2-pole breaker**; IP67/IP68 terminations; licensed electrician |
| PSU | 12 V 30 A (360 W) switching PSU | Mounted above the waterline inside the IP65/66 enclosure, vented |
| 12 V distribution | Main fuse → per-branch blade fuses | Size fuses to conductors and loads after measuring pump inrush ([`BOM.md`](BOM.md) #21) |
| 5 V rail | LM2596S 24 V/12 V → 5 V USB module | Camera node; a second small 12 V→5 V buck is recommended to feed the ESP32 5V pin (the onboard regulator runs hot from 12 V) |
| Grounds | 12 V negative, ESP32 GND, relay GND, sensor GND | Common star point |

## 9. Pre-Energize Checklist

1. Antenna on every radio node.
2. Polarity checked with a multimeter on all three rails.
3. Relay inputs show pulled-up (relays OFF) with the ESP32 disconnected.
4. No cable under tension; drip loops formed; glands tight.
5. Branch fuses fitted and rated.
6. Bench supply first, mains last, and the electrician's sign-off before 220 V reaches the float.
