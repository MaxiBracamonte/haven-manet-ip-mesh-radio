# ADS-B to CoT

Display aircraft in TAK — ADS-B feed to TAK gateway.

ADSBCOT is available for OpenMANET via `opkg`, but it is not installed by default. Once installed and enabled, you can forward aircraft tracks into the Team Awareness Kit (TAK) ecosystem with minimal effort. The integration is intentionally opinionated: it assumes an RTL-SDR-based receiver and multicasts the resulting Cursor on Target (CoT) data across the mesh for TAK clients to consume.

For deeper details, refer to the [official ADSBCOT documentation](https://adsbcot.readthedocs.io/).

![RTL-SDR dongle connected to a compact ADS-B antenna](../../assets/rtl-sdr-adsb-dongle.png)

## Features

- Converts ADS-B messages to CoT format for TAK clients
- Preserves aircraft track, course, speed vectors, and metadata
- Compatible with ATAK, TAKX, WinTAK, and iTAK
- Supports multiple ADS-B data aggregators and CoT receivers
- Accepts over-the-air RF ADS-B via SDR hardware
- Runs on Python 3.7+ across Windows and Linux

## Getting Started on OpenMANET

### 1. Install ADSBCOT

Install via LuCI (**System > Software > Update lists** > search `adsbtocot` > Install) or CLI:

```bash
opkg update
opkg install adsbtocot
```

If using an older `adsbtocot` package revision that does not pull crypto dependencies automatically:

```bash
opkg install python3-cryptography
```

### 2. Connect the SDR

Plug an RTL-SDR dongle into your Raspberry Pi (USB 3 preferred).

![Field ADS-B receiver setup with antenna, RTL-SDR, and Haven node](../../assets/rtl-sdr-adsb-field-setup.png)

### 3. Enable and Start Services

ADS-B to CoT relies on two OpenWrt services:

| Service | Role |
|---------|------|
| `dump1090` | Collects ADS-B frames from the SDR |
| `adsbcot` | Converts ADS-B feed to CoT |

Enable via the OpenWrt GUI (**System > Startup**) or CLI:

```bash
/etc/init.d/dump1090 enable
/etc/init.d/dump1090 start
/etc/init.d/adsbcot enable
/etc/init.d/adsbcot start
```

### 4. Verify Service State

```bash
which adsbcot
pgrep -af "adsbcot|dump1090"
logread -e adsbcot | tail -n 40
```

### 5. Confirm in TAK

Open ATAK, WinTAK, or iTAK and verify aircraft CoT markers are appearing on the map.

## Requirements

- RTL-SDR dongle (USB 3 recommended)
- OpenMANET node (Raspberry Pi gate or point)
- TAK client (ATAK, WinTAK, iTAK, or TAKX) connected to the Haven node's WiFi

## Troubleshooting

### No Aircraft Appearing in TAK

1. **SDR not detected** — check `lsusb` for the RTL-SDR device
2. **dump1090 not running** — `pgrep -af dump1090`; check `logread -e dump1090`
3. **adsbcot not running** — `pgrep -af adsbcot`; check `logread -e adsbcot`
4. **No aircraft in range** — verify dump1090 is receiving frames: `wget -qO- http://localhost:8080/data.json 2>/dev/null | head`
5. **TAK client not on mesh WiFi** — phone/tablet must be connected to the Haven node's AP
