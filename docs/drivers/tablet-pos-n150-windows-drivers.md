# Windows Tablet/POS Driver Package (Intel Processor N150)

A complete driver collection for **Intel Alder Lake-N N150** based tablet/POS devices, covering chipset, graphics, audio, camera, wireless, sensors, and peripherals.

## Platform

- **Processor:** Intel Alder Lake-N N150
- **OS Support:** Windows 10 / Windows 11 (64-bit)

## Driver List

| # | Category | Driver | Version | Installer |
|---|----------|--------|---------|-----------|
| 1 | Chipset | Intel Chipset Device Software | 10.1.19222.8341 | `SetupChipset.exe` |
| 2 | HID | Intel HID Event Filter Driver | 2.2.1.386 | `Setup.exe` |
| 3 | CSME | Intel Converged Security & Management Engine | 16.50.0.1175 v2 | `SetupME.exe` |
| 4 | GNA | Intel Gaussian & Neural Accelerator | 3.0.0.1457 | pnputil |
| 5 | Dynamic Tuning | Intel Dynamic Tuning Technology (DTT + IPF) | 9.0.11000.27915 | `.exe` per sub-package |
| 6 | Serial IO | Intel Serial IO Driver | 30.100.2229.4 v2 | `SetupSerialIO.exe` |
| 7 | ISH | Intel Integrated Sensor Hub Kit | 5.4.2.4587 v2 | `SetupISS.exe` |
| 8 | Graphics | Intel Graphics Driver (DCH) | 32.0.101.6080 | `Installer.exe` |
| 9 | G-Sensor | DA217 / MXC6655 Accelerometer | — | pnputil |
| 10 | Audio | Realtek ALC269VC3 UAD | 9718.1 | `InstallPackage.bat` |
| 11 | Camera | Intel MIPI Camera (GC5035 / GC8034) | — | pnputil |
| 12 | Wi-Fi | Intel Wi-Fi 6 AX101 | 22.240.0 | pnputil |
| 13 | Bluetooth | Intel Bluetooth USB | — | pnputil |
| 14 | SD Card | Alcor Micro USB SD Card Reader | — | pnputil |
| 15 | Fingerprint | FocalTech FPSTP | 5.3.66.5433 | pnputil |

## Installation

> **If a `.bat` file exists in the folder, run it as Administrator to install. Otherwise, run the `.exe` installer as Administrator.**

### Batch install (recommended)

Each driver folder contains an `install Driver.bat` that runs:

```bat
pnputil /add-driver *.inf /subdirs /install
```

Right-click the `.bat` file → **Run as administrator**.

### Manual install

For folders with `.exe` installers (Chipset, Graphics, CSME, Serial IO, etc.), right-click the executable → **Run as administrator**.

### Install order

Follow the numbered folder order (1 → 15) to ensure correct dependency resolution:

1. **Chipset** — must be installed first
2. **HID → CSME → GNA → Dynamic Tuning → Serial IO → ISH** — core platform drivers
3. **Graphics** — GPU driver (requires ≥ 4 GB RAM)
4. **G-Sensor → Audio → Camera → Wi-Fi → Bluetooth → SD → Fingerprint** — peripherals

## Graphics Driver Notes

- Version 32.0.101.6080 (October 2024)
- Supports Windows 10 22H2, Windows 11 21H2–24H2
- Includes Vulkan runtime, OpenCL ICD, oneVPL, Intel Graphics Command Center
- Minimum system memory: 4 GB

## Audio Driver Notes

- Realtek ALC269VC3 Universal Audio Driver (UAD), WHQL certified
- Hardware ID: `10EC133E`
- Includes APO (Win10/Win11), ASIO, HSA, and service components

## Camera Driver Notes

- Intel ICamera framework with ISP
- Sensors: Galaxycore GC5035 (front) and GC8034 (rear)

## Fingerprint Driver Notes

- FocalTech FPSTP, WHQL certified for Windows 24H2 x64
- Integrates with Windows Biometric Framework (WBF)

## Resources

- Source: [SUNTEK Cloud Share](https://github.com/SUNTEK-Official/suntek-docs/releases)
- Password: `suntek`


## Verified Release Downloads

Download EV-signed driver bundles from [SUNTEK Support](https://www.suntekai.com) or [GitHub Releases](https://github.com/SUNTEK-Official/suntek-docs/releases) and verify checksums using `SHA256SUMS.txt`.
