# Pi 5 Platform Notes (Haven 2)

Haven 2 runs on the Raspberry Pi 5 with USB-attached peripherals.

## Firmware

Haven 2 requires a Raspberry Pi 5-specific OpenWrt/Morse firmware image. Do **not** use the Haven 1 / Raspberry Pi 4 OpenMANET image on Pi 5 hardware.

Use one of these options:

- **Download:** prebuilt [v0.3.0-alpha release](https://github.com/buildwithparallel/openwrt-morse-rpi5/releases/tag/v0.3.0-alpha)
- **Build from source:** [`buildwithparallel/openwrt-morse-rpi5`](https://github.com/buildwithparallel/openwrt-morse-rpi5)

The Haven setup scripts in this repo run after the Pi 5 firmware is flashed and booted.

## Hardware

- **SBC**: Raspberry Pi 5 (4GB or 8GB)
- **HaLow Radio**: Morse Micro MM8108 (USB)
- **2.4GHz WiFi**: RT5370 USB adapter (Panda Wireless or similar)
- **USB Hub**: Recommended — the Pi 5 has 4 USB ports but HaLow + WiFi + peripherals (SDR, LoRa) can fill them quickly

### Optional Peripherals

- **RTL-SDR V3** — ADS-B aircraft tracking to TAK ([setup](../../integrations/atak/adsb-to-cot.md))
- **LoRa MCU** — Reticulum sidecar radio ([tested devices](tested-usb-devices.md))

## Known Issues

### Wireless config resets on reboot

The Morse Micro firmware includes a hotplug script (`/etc/hotplug.d/ieee80211/10-wifi-detect`) that runs `/sbin/wifi config` every time a radio PHY is detected, regenerating default wireless settings and overwriting customizations.

**Fix:** Edit the hotplug script so it only runs on first boot:

```sh
# /etc/hotplug.d/ieee80211/10-wifi-detect
#!/bin/sh

# Only run wifi config if no radios are configured yet.
[ "${ACTION}" = "add" ] && {
	uci -q get wireless.radio0 > /dev/null || /sbin/wifi config
}
```

### WiFi AP fails after USB topology change

Adding or removing USB devices (e.g., plugging in a hub) changes the USB bus enumeration. If the WiFi adapter moves to a different bus path, `radio1` may show `retry_setup_failed: true` and the AP won't broadcast.

**Fix:** `wifi up radio1` restarts the radio on its new path.

### RTL-SDR not detected via extension cable

The RTL-SDR V3 may not enumerate through cheap USB extension cables due to signal integrity or power issues.

**Fix:** Use a powered USB hub, or plug the SDR directly into the Pi and put the more tolerant WiFi adapter on the extension.

### HaLow HAT SPI failures (if using SPI HAT instead of USB)

See [Pi 4 notes](pi4-notes.md#halow-hat-spi-failures) — same issue applies if using the Seeed WM-WM6108 SPI HAT on a Pi 5.

## OpenWrt Build

See [openwrt-build-packages.md](openwrt-build-packages.md) for the recommended package list if you are building the Pi 5 firmware yourself.
