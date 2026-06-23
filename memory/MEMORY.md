# Haven MANET Mesh Radio - Project Memory

## Network Topology
- **Gate (green)**: 192.168.0.66 (eth0), 10.41.0.3/16 (br-ahwlan), pw: havengreen
- **Heltec v2 node (heltec-4)**: 10.41.0.4/16, pw: heltec.org
- Other nodes: Heltec v2 in Tesla, Heltec v1 on desk (IPs unknown)
- Mesh: 802.11s HaLow (morse) + BATMAN-adv (BATMAN_V) on channel 27, mesh_id "haven"
- Gate runs batctl 2025.4 (OpenMANET/RPi), Heltec v2 runs batctl 2023.1 (OpenWrt 23.05.5) — compatible

## Key BATMAN-adv Config Pattern (working)
The wireless mesh interface must point to a `batadv_hardif` network, NOT directly to the bridge:
```
wireless.mesh_iface.network='batmesh'    # NOT 'ahwlan'
wireless.mesh_iface.mesh_fwding='0'
network.batmesh.proto='batadv_hardif'
network.batmesh.master='bat0'
network.bat0.proto='batadv'
network.ahwlan_dev.ports='bat0'          # bat0 in bridge, not wlan0
```

## Bug: Anonymous bridge device conflict
Heltec default firmware has an anonymous `network.@device[N]` for br-ahwlan with no ports.
This conflicts with the named `ahwlan_dev` and prevents bat0 from being added to the bridge.
**Fix**: Delete anonymous br-ahwlan devices before creating the named one. See configure-heltec.sh.

## Tesla Node
- Hostname: HT-MeshGate, static lease IP: 10.41.0.5, pw: heltec.org
- MACs: 06:36:ef:1d:83:f1, f2:5f:97:0b:bf:00 (changes across reboots)
- Has separate br-lan (10.42.0.0/24) for WiFi clients, NATs to ahwlan zone
- Reachable from local machine via ProxyCommand through gate
- Static DHCP lease configured on gate as dhcp.tesla

## Boot-time Bridge Check
- `haven-bridge-check.sh` — init script that verifies bat0 is in br-ahwlan on every boot
- Installed on: gate (192.168.0.66), Tesla node (10.41.0.118)
- Install: scp to /etc/init.d/haven-bridge-check, chmod +x, enable

## Gate HaLow HAT Hardware
- Seeed WM-WM6108 HAT on gate RPi can fail with `morse_spi spi0.0: cmd53_write/read failed (errno=-71)` on boot
- Cause: case standoffs bow HAT off GPIO header, causing intermittent SPI contact
- Fix: add 1mm standoff height; press HAT firmly onto header before closing case
- Recovery (if HAT is seated): `rmmod morse && modprobe morse` — no full power cycle needed
- Full details: see `hardware_gate_halow.md`

## Rules
- [Don't use Quick Config on Haven nodes](feedback_quick_config.md) — breaks batmesh0 assignment, kills BATMAN routing
- [No co-author lines in commits](feedback_no_coauthor.md) — omit Co-Authored-By trailer from all git commits

## Scripts
- `configure-heltec.sh` — configures BATMAN-adv on Heltec HaLow nodes for Haven mesh
- `setup-haven-gate.sh` — gate node setup (channel 27, batadv_hardif pattern)
- `setup-haven-point.sh` — point node setup (channel 27, batadv_hardif pattern)
- All scripts now: clean anonymous bridge devices, use explicit ports list, use batmesh hardif
