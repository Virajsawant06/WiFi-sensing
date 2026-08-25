# RuView Setup Audit, Hardware Diagnostic & Documentation Differences

This document records the exact findings from inspecting the environment, connected ESP32 hardware, documentation procedures in `Documentation.md`, and any discrepancies found during setup.

---

## 1. Connected Hardware Diagnostic (COM3)

Using `esptool` and Windows device enumeration, the connected microcontroller was scanned:

| Parameter | Value | Notes |
|---|---|---|
| **Serial Port** | `COM3` | Silicon Labs CP210x USB to UART Bridge |
| **Detected Chip** | **`ESP32-D0WD-V3 (revision v3.1)`** | Standard ESP32 (Xtensa LX6 dual-core) |
| **Flash Size** | **4 MB** | Manufacturer 68, Device 4016 |
| **Crystal Frequency** | 40 MHz | Standard |
| **MAC Address** | `1c:c3:ab:b3:eb:70` | Unique Node Hardware MAC |
| **Host IP (WiFi)** | `192.168.29.122` | Active IPv4 on `Wi-Fi 2` adapter |

---

## 2. Documentation Audit & Differences Found

### A. Pre-built Binaries vs. Connected Chip Architecture
* **Documentation Claim:** Section 2.1 states standard `ESP32` or `ESP32-S3` can be used, and firmware `README.md` suggests using pre-built binaries in `firmware/esp32-csi-node/release_bins/`.
* **Discrepancy:** All pre-compiled binaries in `release_bins/` (including `esp32-csi-node-4mb.bin`) are built specifically for **ESP32-S3** (`Chip ID: 9 (ESP32-S3)` / Xtensa LX7).
* **Impact:** Flashing `release_bins/` directly to the connected **standard ESP32** (`ESP32-D0WD-V3` / Xtensa LX6) will fail to boot due to CPU architecture incompatibility.
* **Resolution:** Standard ESP32 boards must be compiled using ESP-IDF targeting `esp32`:
  ```powershell
  # Using ESP-IDF / Docker build container
  idf.py set-target esp32
  idf.py build
  ```
  *(Or if using an ESP32-S3 dev board, the pre-built binaries can be flashed directly).*

---

### B. Server Execution: Docker vs. Native Rust Cargo
* **Documentation Claim:** Phase 4 & Phase 5 in `Documentation.md` require Docker Desktop, subnet CIDR updates (`172.18.0.0/16`), and a Python UDP loopback relay (`scripts/udp-relay.py`).
* **Technical Finding:** 
  * Running a native build directly on Windows (`cargo check/build`) triggers MSVC / windows-sys macro depth limitations (`STATUS_STACK_BUFFER_OVERRUN`).
  * Therefore, running the server inside the **Linux Docker container** (as described in `docker/Dockerfile.rust` and `docker-compose.yml`) or inside **WSL2** is strictly recommended for stable execution.

---

## 3. How to Enable Full Authority Mode in Antigravity

To stop Antigravity from prompting for *"Allow once / Allow always"* on terminal commands:

1. **Via Chat / Mode Toggle:**
   - Look at the bottom/top of the chat interface for the **Execution Mode** or **Permission Selector**.
   - Switch mode from **"Ask Before Running" / "Review"** to **"Always Proceed" / "Auto-Execute"**.

2. **Via IDE Settings:**
   - Open **Settings** (Click the Gear icon in the bottom left or press `Ctrl + ,`).
   - Search for `Tool Execution Policy` or `Agent Permissions`.
   - Set **Tool Execution Policy** to `always-proceed`.
   - Set **Terminal Sandbox** to allow direct command execution if prompted.

3. **Per-Project Customization (`.agents/`):**
   - In your workspace root, configurations in `.agents/` or IDE workspace preferences can enforce `always-proceed` for all pair-programming actions.

---

## 4. Summary & Verification

- **COM Port Verified:** `COM3` (ESP32-D0WD-V3 4MB).
- **Host IPv4 Verified:** `192.168.29.122`.
- **Code Integrity:** All source files preserved without unauthorized modifications.
