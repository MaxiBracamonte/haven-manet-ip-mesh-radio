# OpenWrt Build — Recommended Packages for Haven 2 / Pi 5

Target: `bcm27xx/bcm2712` (Raspberry Pi 5), OpenWrt 23.05.x, aarch64_cortex-a76

This page is for building the Haven 2 / Raspberry Pi 5 OpenWrt/Morse firmware yourself from [`buildwithparallel/openwrt-morse-rpi5`](https://github.com/buildwithparallel/openwrt-morse-rpi5). If you just want to flash a prebuilt image, use the [v0.3.0-alpha release](https://github.com/buildwithparallel/openwrt-morse-rpi5/releases/tag/v0.3.0-alpha).

## ADS-B to CoT

```
rtl-sdr
librtlsdr
dump1090
python3
python3-pip
python3-cryptography
adsbcot
```

## Reticulum / Mesh Networking

```
python3-rns
python3-lxmf
python3-nomadnet
python3-pyserial
python3-netifaces
```

## USB Serial — LoRa / MCU Sidecar Support

```
kmod-usb-acm
kmod-usb-serial
kmod-usb-serial-cp210x
kmod-usb-serial-ch341
kmod-usb-serial-ftdi
```

Tested devices:

| Device | USB ID | Driver |
|--------|--------|--------|
| Heltec V4 (ESP32-S3) | `303a:1001` | kmod-usb-acm |
| RAK4631 (nRF52840) | `239a:8029` | kmod-usb-acm |
| Heltec V1 (nRF52) | `239a:4405` | kmod-usb-acm |
| Muzi Works Base Duo | `239a:cafe` | kmod-usb-acm |
| Seeed Xiao (ESP32-S3) | `303a:1001` | kmod-usb-acm |
| Walter (ESP32-S3) | `303a:1001` | kmod-usb-acm |
| Null Hop Mesh Toad | `1a86:5512` | kmod-usb-serial-ch341 |

## WiFi Drivers

```
kmod-rt2800-usb
kmod-mt76x0u
kmod-rtl8821cu
```

Tested devices:

| Device | USB ID | Driver |
|--------|--------|--------|
| Panda Wireless N | `148f:5370` | kmod-rt2800-usb (already included) |
| MediaTek AC dongle | `0e8d:7610` | kmod-mt76x0u |
| Realtek AC dongle | `0bda:0811` | kmod-rtl8821cu |

## Network Debugging

```
tcpdump
iperf3
mtr
nmap
usbutils
```

## General Utilities

```
ca-certificates
curl
bash
nano
htop
screen
```
