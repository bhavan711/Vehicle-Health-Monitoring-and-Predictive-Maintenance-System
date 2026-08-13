# J1939 Vehicle Health Monitoring System (VHMS)

A 4-node CAN bus vehicle health monitoring system built on the SAE J1939 protocol, designed for commercial vehicles and recalibrated to Indian regulatory specs (AIS-052/CMVR).

## Overview

The system monitors brake, engine, and safety-critical parameters across a distributed CAN bus network, streams telemetry to the cloud, and runs an on-device ML model for predictive maintenance — flagging failures before they happen instead of after.

## Architecture

```
[Brake ECU] ─┐
[Engine ECU] ─┼── J1939 CAN Bus ── [Gateway ECU] ── MQTT ── [ThingsBoard Dashboard]
[Safety/TPMS ECU] ─┘
```

- **4 nodes:** Brake ECU, Engine ECU, Safety/TPMS ECU, Gateway
- **Hardware:** ESP32 + MCP2515 CAN controller per node
- **Protocol stack:** Full J1939-21/73/81 compliance
  - 29-bit extended CAN IDs
  - Real SAE-defined PGNs (Parameter Group Numbers)
  - Address claiming procedure
  - Transport protocol (BAM broadcast + RTS/CTS point-to-point)
- **Cloud:** Gateway publishes over MQTT to a ThingsBoard dashboard for real-time visualization

## Predictive Maintenance (ML)

- Model: XGBoost classifier, AUC ≈ 0.705
- Trained on vehicle health features derived from CAN telemetry
- Distilled into lightweight C headers (`vehicle_health_model.h`, `engine_health_model.h`) for direct on-device inference on ESP32 — no cloud round-trip needed for predictions

## Physics & Regulatory Modeling

- FHWA-based rollover and braking distance models, recalibrated using Indian bus specifications (AIS-052/CMVR)
- Cross-validated against real accident data from MoRTH (Ministry of Road Transport and Highways)
- Three-way regulatory comparison: US (FMVSS) vs EU vs India (AIS-052/CMVR)

## Hardware Used

| Component | Purpose |
|---|---|
| ESP32 | Node MCU, TWAI/CAN interfacing, MQTT client |
| MCP2515 | CAN bus controller (SPI) |
| Commercial vehicle sensor suite | Brake, engine, TPMS parameter inputs |

## Repository Structure

```
├── nodes/
│   ├── brake_ecu/
│   ├── engine_ecu/
│   ├── safety_tpms_ecu/
│   └── gateway/
├── ml/
│   ├── training/              # XGBoost training scripts
│   └── models/                # Distilled C header models
├── dashboard/                 # ThingsBoard config/widgets
└── docs/                      # Protocol notes, regulatory comparison
```

## Getting Started

1. Flash each node's firmware from `nodes/<node_name>/` onto its ESP32 via Arduino IDE / PlatformIO
2. Wire MCP2515 to ESP32 over SPI 
3. Configure Gateway node with your MQTT broker / ThingsBoard credentials
4. Power on nodes — address claiming and CAN handshake happen automatically
5. View live telemetry on the ThingsBoard dashboard

## Status

Research prototype — built as part of an internship project on vehicle health monitoring systems.

## License

MIT 
