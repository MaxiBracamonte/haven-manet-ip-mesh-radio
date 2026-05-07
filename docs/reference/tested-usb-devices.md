# Tested USB Devices

USB peripherals tested on Haven nodes (Pi 4 and Pi 5) with OpenMANET / OpenWrt 23.05.x.

## LoRa / MCU Sidecars

These boards connect as USB serial devices for use with Reticulum (RNodeInterface) or other serial protocols.

| Device | USB ID | Chip | Driver |
|--------|--------|------|--------|
| Heltec LoRa V4 | `303a:1001` | ESP32-S3 (native USB) | `kmod-usb-acm` |
| Seeed Xiao (ESP32-S3) | `303a:1001` | ESP32-S3 (native USB) | `kmod-usb-acm` |
| Walter (ESP32-S3) | `303a:1001` | ESP32-S3 (native USB) | `kmod-usb-acm` |
| RAK4631 (WisBlock) | `239a:8029` | nRF52840 (native USB) | `kmod-usb-acm` |
| Heltec HT-n5262 (V1) | `239a:4405` | nRF52 (native USB) | `kmod-usb-acm` |
| Muzi Works Base Duo | `239a:cafe` | nRF52840 / RP2040 | `kmod-usb-acm` |
| Null Hop Mesh Toad | `1a86:5512` | CH340/CH341 | `kmod-usb-serial-ch341` |

## WiFi Adapters

| Device | USB ID | Driver |
|--------|--------|--------|
| Panda Wireless N | `148f:5370` | `kmod-rt2800-usb` |
| MediaTek 802.11ac | `0e8d:7610` | `kmod-mt76x0u` |
| Realtek 802.11ac | `0bda:0811` | `kmod-rtl8821cu` |

## SDR

| Device | USB ID | Driver |
|--------|--------|--------|
| RTL-SDR Blog V3 | `0bda:2838` | None (userspace `librtlsdr` + `libusb`) |

> **Note:** The RTL-SDR V3 may not enumerate through cheap USB extension cables. Use a powered USB hub or plug directly into the Pi.

## Required Kernel Modules

To support all tested devices, include these in the OpenWrt build:

```
# USB serial (LoRa / MCU sidecars)
kmod-usb-acm
kmod-usb-serial
kmod-usb-serial-ch341
kmod-usb-serial-cp210x
kmod-usb-serial-ftdi

# WiFi
kmod-rt2800-usb
kmod-mt76x0u
kmod-rtl8821cu
```

CP210x and FTDI are included for coverage of common boards not yet tested (Heltec V2/V3, T-Beam, generic LoRa modules).
