# SiK-Multipoint Firmware with HOSTMUX / COBS Support

> **⚠️ WARNING:** This repository represents a specialized fork/branch extending the original SiK multipoint radio firmware with robust **HOSTMUX (COBS-encoded framing)** and a companion **Python UDP bridge** for multi-drone/multi-node telemetry routing. The upstream project can be found at [RFDesign/SiK](https://github.com/RFDesign/SiK/tree/SiK_Multipoint). It is maintained here for historical and reference purposes.

---

## Overview

This repository provides firmware and tools for SiLabs Si1000 ISM radios (such as HopeRF HM-TRP, RF50-DEMO, and RFD900) optimized for **multipoint mesh/star topologies**. 

In addition to standard SiK features, this version introduces **HOSTMUX mode**, which uses Consistent Overhead Byte Stuffing (COBS) and CRC-16 framing over the serial link to reliably multiplex data and AT commands across multiple remote nodes from a single base radio.

---

## Key Architecture & Features

### 1. HOSTMUX Protocol & COBS Framing
* **Framing Structure:** Replaces raw slip/stream parsing with robust COBS-encoded frames containing frame type, destination Node ID, payload length, payload, and a 16-bit CRC.
* **Frame Types:**
  * `HOSTMUX_FRAME_TYPE_TO_NODE (1)`: Uplink data routed from base to a specific remote node.
  * `HOSTMUX_FRAME_TYPE_FROM_NODE (2)`: Downlink data received from a remote node back to base.
  * `HOSTMUX_FRAME_TYPE_AT_CMD (3)`: Remote/local AT command execution.
  * `HOSTMUX_FRAME_TYPE_AT_REPLY (4)`: Buffered responses to AT queries (e.g., chunked handling for `ATI5`).
* **COBS Encoding:** Eliminates framing sync issues by ensuring zero bytes only act as packet delimiters.

### 2. Companion Python UDP Bridge (`node_udp_bridge.py`)
A transparent, asynchronous Python bridge that maps UDP ports to specific radio NodeIDs, allowing Ground Control Stations (GCS) like QGroundControl or Mission Planner to talk directly to individual drones over a single base radio.

#### Port Mapping Schema (`--port-base B`, default `14550`):
* **Control / AT:** `B` (e.g., `:14550`)
* **Drone NodeID 1:** `B + 1` (e.g., `:14551`)
* **Drone NodeID N:** `B + N`

#### Usage Examples:
```bash
# Bridge a base radio on /dev/ttyUSB0 with two remote drone nodes
python3 node_udp_bridge.py --port /dev/ttyUSB0 --node 1 --node 2 --baud 57600

# Windows setup with custom base port and startup AT configuration
python3 node_udp_bridge.py --port COM7 --node 1 --node 2 --node 3 --port-base 14560 --init-at ATS18=4
