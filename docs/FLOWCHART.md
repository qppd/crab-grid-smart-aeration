# System Flowchart

**Project:** Solar Automated LoRa-Integrated Mud Crab Aeration Network with Crab Grid Observation

This document presents the operational flowchart of the system in the format required . It traces one complete control cycle: sensor acquisition and automatic first-aid logic at the float node, LoRa transmission to the house bridge node, telemetry storage in Firebase, dashboard command relay, and the fail-safe mode rules enforced by the firmware ([`FIRMWARE.md`](FIRMWARE.md) §2).

## 1. Flowchart

```mermaid
flowchart TD
    subgraph FLOAT["Float Node - ESP32 + E22-900M22S (pond side)"]
        A([Start: power on or reset]) --> B[Initialize GPIO, sensor interfaces, relays and LoRa radio]
        B --> C[Initialize mode: AUTO with aeration ON]
        C --> D{Local selector mode}
        D -->|OFF| E[Keep all actuators off]
        D -->|MANUAL| F[Apply operator command from selector or webapp]
        F --> F1{No command received for 5 minutes?}
        F1 -->|Yes| D
        D -->|AUTO| G[Read DO, pH, EC/salinity and temperature probes]
        G --> H{DO below 3 ppm?}
        H -->|Yes| I[Force aeration ON and raise alarm]
        I --> I1[Queue immediate emergency uplink]
        H -->|No| J{Salinity below threshold?}
        J -->|Yes| K[Energize assigned bilge pump]
        J -->|No| L[Run scheduled aeration per thresholds]
        M[Compose uplink: sensor data, mode, pump state and heartbeat]
        E --> M
        F1 -->|No| M
        I1 --> M
        K --> M
        L --> M
        M --> N[Transmit LoRa packet to the house bridge]
        N --> O{Downlink command received?}
        O -->|Yes| P[Validate device ID, sequence, expiry and CRC-8]
        P -->|Valid and fresh| Q[Apply mode and pump command, return ACK]
        P -->|Invalid or stale| R[Return NAK and keep current mode]
        O -->|No| S[Wait for the next heartbeat cycle]
        Q --> D
        R --> D
        S --> D
    end

    subgraph BRIDGE["House Bridge Node - ESP32 + E22-900M22S (where WiFi exists)"]
        T[Receive uplink and verify device ID and CRC-8] --> U[Write telemetry to the Firebase Realtime Database]
        U --> V[Poll the Realtime Database for queued commands]
        V --> W{Fresh dashboard command queued?}
        W -->|Yes| X[Send the command downlink over LoRa]
        X --> Y{ACK received from the float node?}
        Y -->|Timeout or NAK| X
        Y -->|ACK| Z[Wait for the next uplink]
        W -->|No| Z
    end

    subgraph DASH["Remote Dashboard - Next.js + Firebase on Vercel"]
        AA["onValue() listeners stream live telemetry to the web UI"] --> AB[Operator selects mode or per-pump override]
        AB --> AC[Route handler writes the command into the Realtime Database]
    end

    N --> T
    AC --> V
```

**Figure 1.** Overall operational flowchart of the Solar Automated LoRa-Integrated Mud Crab Aeration Network.

## 2. Flow Description

1. **Initialization.** On power-up or reset the node forces mode AUTO with aeration ON — the fail-safe default ([`FIRMWARE.md`](FIRMWARE.md) §2).
2. **Mode selection.** The local selector decodes AUTO, MANUAL, or OFF ([`WIRING.md`](WIRING.md) §6). OFF exists for servicing only.
3. **Automatic first aid.** In AUTO the firmware reads dissolved oxygen, pH, EC/salinity, and temperature. DO below 3 ppm overrides every mode, forces aeration, and triggers an immediate emergency uplink.
4. **Salinity logic.** A sub-threshold salinity run engages the assigned bilge pump for flushing or controlled brine testing; open-water salinity correction is not claimed ([`Components.md`](Components.md) — Water circulation).
5. **Uplink.** Every cycle composes one frame — sensor data, mode, pump state, heartbeat — transmitted over 915 MHz LoRa point-to-point.
6. **Downlink handling.** Commands are accepted only when the device ID, sequence, expiry window, and CRC-8 all validate; otherwise the float returns a NAK and keeps its current mode.
7. **Bridge and cloud.** The house bridge verifies each uplink, writes telemetry to the Realtime Database, relays queued dashboard commands downlink, and tracks ACK/NAK results.
8. **Dashboard.** `onValue()` listeners stream the live state to the web UI; operator commands enter the Realtime Database queue ([`APP.md`](APP.md) §4).

## 3. Fail-Safe Notes

- MANUAL auto-reverts to AUTO after 5 minutes without a fresh command.
- Heartbeat loss beyond 90 s raises an alarm at the house bridge.
- The physical selector always overrides dashboard intent at the device.
