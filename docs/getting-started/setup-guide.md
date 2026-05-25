# Haven Setup Guide

Step-by-step instructions for setting up a Haven mesh network.

**[Haven Guide](https://buildwithparallel.com/products/haven)** - Video tutorials, schematics, 3D printable enclosures, Discord community, and direct support.

> **Prerequisite:** All scripts assume each node is flashed with a fresh/recent version of [OpenMANET](https://openmanet.org/).
>
> **Fresh install:** Flash OpenMANET onto each node's microSD card using Raspberry Pi Imager, then insert the card and power on.
>
> **Card not wiping cleanly or re-flash looks half-done?** In Raspberry Pi Imager, use the **Erase** option first (in **Choose OS** it is often under **Raspberry Pi OS (other)** or a **Misc** / **utility** section, depending on version; some builds label it as formatting the card). That fully blanks the microSD, then run **Choose OS** → your OpenMANET file / image and **Write** as usual.
>
> **Pro tip (Raspberry Pi 4):** If **Erase** then OpenMANET still misbehaves, some installs respond to writing **Raspberry Pi OS (vanilla)** to the card first, then using Imager again to **write the OpenMANET image on top** (overwriting the Pi install). The intermediate full Pi OS image can force a clean partition layout on stubborn cards. Booting Pi OS once in between is optional; either way, finish with OpenMANET as the last write.
>
> **Upgrading an existing install:** Open LuCI → System → Backup / Flash Firmware → upload the OpenMANET image. **Uncheck "Keep settings"** for a clean slate.

## Step 1: Set Up the Gate Node (green)

This is the node that shares internet with the rest of the mesh.

1. **Flash OpenMANET** onto a microSD card and insert it into the gate node
2. **Connect the gate to your computer** via Ethernet and power it on
3. **Open LuCI** — go to `http://10.41.254.1` (or the IP shown on the node's boot screen) in your browser
4. **Run through the OpenMANET initial wizard** — this configures the basic mesh gate settings (hostname, mesh channel, password, etc.). You can **disable the 2.4GHz and 5GHz WiFi radios** in the wizard — the Haven setup script reconfigures them from scratch anyway
5. **Plug the gate into your upstream router** via Ethernet
6. **Find the gate's new IP** in your router's device/DHCP table
7. **Open a terminal on the gate** — pick one:
   - **Browser:** go to `http://<gate-ip>` → **Services → Terminal**
   - **SSH:** `ssh root@<gate-ip>` from your computer
8. **Run the Haven setup script** (the node now has internet via the router):
   ```bash
   wget -O /tmp/setup.sh https://raw.githubusercontent.com/buildwithparallel/haven-manet-ip-mesh-radio/main/scripts/node-setup/setup-haven-gate.sh && sh /tmp/setup.sh
   ```
9. The node **reboots automatically** when the script finishes — wait ~60 seconds

### Gate Smoke Test

1. On your phone, connect to the **`green`** WiFi (password: `greengreen`)
2. Open a browser and confirm you have internet
3. Run a speed test — this is your baseline; point nodes will be slower due to HaLow backhaul

## Step 2: Add Point Nodes (blue)

Point nodes extend the mesh — no internet connection needed on the point during setup.

1. **Flash OpenMANET** onto a microSD card and insert it into the point node
2. **Connect the point to your computer** via Ethernet and power it on
3. **Open LuCI** — go to `http://10.41.254.1` in your browser
4. **Run through the OpenMANET initial wizard** — select mesh point mode. You can disable the 2.4GHz and 5GHz WiFi radios in the wizard; the Haven setup script reconfigures them
5. After the wizard finishes, **keep the Ethernet connected** and check your router or network settings for the point’s new IP address
6. **Open LuCI at the new IP** → **Services → Terminal**
7. In the terminal, run:
   ```sh
   cat > /tmp/setup.sh
   ```
8. On your laptop, open the [raw point setup script](https://raw.githubusercontent.com/buildwithparallel/haven-manet-ip-mesh-radio/main/scripts/node-setup/setup-haven-point.sh) in a browser, **select all → copy**, then paste it into the terminal
9. Press **Ctrl+D** to finish, then run:
   ```sh
   sh /tmp/setup.sh
   ```
10. The node **reboots automatically** when the script finishes — wait ~60 seconds

### Point Smoke Test

1. On your phone, connect to the **`blue`** WiFi (password: `blueblue`)
2. Open a browser and confirm you have internet
3. Bandwidth here is limited by the HaLow backhaul to the gate — expect lower speeds than on `green` directly, which is normal

### Adding More Nodes

For each additional point node:
1. Edit `setup-haven-point.sh` with unique `HOSTNAME` and `MESH_IP`
2. Keep `MESH_ID`, `MESH_KEY`, `HALOW_CHANNEL` the same as gate
3. Run script and reboot

### Verify the Mesh

1. Connect your phone or laptop to **`green`** WiFi (password: `greengreen`)
2. Check your network settings — the gateway IP shown is the gate; open a terminal there via **`http://<gate-ip>` → Services → Terminal** or SSH
3. Run on the gate to find blue's IP:
   ```bash
   ip neigh show dev br-ahwlan
   ```
4. Ping blue first, then browse to its LuCI:
   ```bash
   ping <blue-ip>
   ```
5. Open **`http://<blue-ip>`** — if blue's LuCI loads, your mesh is working

For deeper diagnostics, run these on any node:
```bash
iwinfo wlan0 info     # HaLow link quality
batctl n              # BATMAN-adv neighbors
```

<img src="../../assets/mesh-verify.png" alt="Mesh verification from point node" width="500">

> After setup, use LuCI's web interface to change passwords, WiFi SSIDs, and other settings on each node. See [Finding & Accessing Nodes](../reference/finding-nodes.md) to reach each node.
>
> **Something not working?** Run the diagnostic script on the problem node:
> ```bash
> wget -O- https://raw.githubusercontent.com/buildwithparallel/haven-manet-ip-mesh-radio/main/scripts/node-ops/haven-diag.sh | sh
> ```

### Connect Your Device

After setup, connect your computer, phone, or tablet to the Haven network:

1. **Join the node's WiFi** — `green` (2.4GHz) or `green-5ghz` for the gate; `blue` for a point (2.4GHz USB Panda only — onboard radios are disabled when Panda is present)
   - Gate: `green` / `greengreen` or `green-5ghz` / `green-5ghz`
   - Point: `blue` / `blueblue` (change in `WIFI_2G4_*` in the point script, or in LuCI)
2. **Verify your device got an IP** — you should receive an address in the `10.41.x.x` range
   - **Mac/Linux:** `ifconfig` or `ip addr` — look for `10.41.x.x` on your WiFi interface
   - **Windows:** `ipconfig` — look for `10.41.x.x` under your Wi-Fi adapter
   - **Phone:** Settings → WiFi → tap the connected network to see your IP
3. **Access the node's web interface** — browse to `http://<node-mesh-ip>`
   - Find the mesh IP by running `uci get network.ahwlan.ipaddr` on the node, or check its boot screen

> **Can't see the WiFi network?** See [Troubleshooting → WiFi AP Not Broadcasting](../runbooks/troubleshooting.md#checklist-4--wifi-ap-not-broadcasting).
>
> **Alternative:** If the gate node is plugged into your home router, you can also connect your computer to your **regular home WiFi** and access the gate at the IP shown in your router's device list — no need to switch WiFi networks.

## Step 3: Install Reticulum (optional)

**Recommended (simplest):** do **not** put Reticulum (RNS) on your Haven nodes. Install a Reticulum **app on each device** you care about — e.g. [Sideband](https://github.com/markqvist/Sideband) (iOS/Android) or [MeshChat](https://github.com/liamcottle/reticulum-meshchat) (desktop). Connect each device to the **Haven mesh WiFi**; in the app, turn on **AutoInterface**. The mesh is just a `10.41.x.x` network; your clients find each other the same as on any LAN. This is the path we document first in the [Reticulum README](../../integrations/reticulum/README.md).

**On-node RNS (advanced, optional):** only if you need `rnsd` on a node — e.g. the [ATAK/CoT bridge](../../integrations/atak/README.md), always-on RNS on hardware, or the [demo `rns_*.py` scripts](../../scripts/README.md#reticulum-demo-scripts). Then run on **each** such node:

```bash
wget -O /tmp/setup-reticulum.sh https://raw.githubusercontent.com/buildwithparallel/haven-manet-ip-mesh-radio/main/scripts/optional/setup-reticulum.sh
sh /tmp/setup-reticulum.sh
/etc/init.d/rnsd enable && /etc/init.d/rnsd start
```

See the [Reticulum README](../../integrations/reticulum/README.md) for when to use clients vs nodes, and for on-node config when you do install.

## Step 4: Send Reticulum Messages (Optional)

If you are using only **Sideband, MeshChat,** etc. (Reticulum on your **devices**), you do not need this step on the **nodes** — you already send messages in the app. Skip to [Step 5](#step-5-install-the-atak-bridge-optional) or you are done.

The [demo scripts in `scripts/tools/`](../../scripts/README.md#reticulum-demo-scripts) are for when **RNS is running on a node** — they need [Step 3](#step-3-install-reticulum-optional) with on-node RNS, then:

See [scripts/README.md](../../scripts/README.md) for `rns_status.py`, `rns_send.py`, and `rns_receive.py` usage and example output.

## Step 5: Install the ATAK Bridge (Optional)

Bridges ATAK/CivTAK traffic over Reticulum. The bridge runs **on each node** and expects `rnsd` — you need **on-node** RNS from [Step 3](#step-3-install-reticulum-optional) first (client-only Reticulum is *not* enough for this).

```bash
wget -O /tmp/setup-cot-bridge.sh https://raw.githubusercontent.com/buildwithparallel/haven-manet-ip-mesh-radio/main/scripts/optional/setup-cot-bridge.sh
sh /tmp/setup-cot-bridge.sh
/etc/init.d/cot_bridge enable && /etc/init.d/cot_bridge start
```

See [ATAK README](../../integrations/atak/README.md) for peering, live dashboards, and troubleshooting.

### Verify ATAK Bridge
```bash
tail -f /tmp/bridge.log
```

---

## Configuring Heltec HaLow Nodes (BATMAN-adv)

The `configure-heltec.sh` script sets up [Heltec HaLow](https://heltec.org/project/ht-hd01/) nodes running OpenWrt to join the Haven mesh using BATMAN-adv routing over 802.11s.

<img src="../../assets/heltec-1.JPG" alt="Heltec HaLow node" width="500">

This is an alternative to the standard point node setup — use it when your node is a Heltec v2 HaLow board rather than a Raspberry Pi with a HaLow HAT.

**What it does:**

1. Binds the HaLow mesh radio to `bat0` via a `batadv_hardif` interface
2. Disables 802.11s forwarding (BATMAN-adv handles routing instead)
3. Creates `bat0` with BATMAN_V in client gateway mode
4. Bridges `bat0` into `br-ahwlan` with a static mesh IP
5. Connects the local WiFi AP to the mesh bridge so clients get internet
6. Removes conflicting anonymous bridge devices from the default firmware

**Usage:**

1. SSH into the Heltec node: `ssh root@10.42.0.1`
2. Edit the configuration variables at the top of the script (`HOSTNAME`, `MESH_IP`, etc.) for your node
3. Paste the script into the terminal and run it
4. Reboot: `reboot`

```bash
# Configuration variables to set per-node:
HOSTNAME="heltec-4"
MESH_IP="10.41.0.4"
MESH_NETMASK="255.255.0.0"
GATEWAY_IP="10.41.0.3"
```

After reboot, the node is reachable at `MESH_IP` on the mesh network.

> **Note:** The Heltec default firmware ships with an anonymous bridge device for `br-ahwlan` that conflicts with BATMAN-adv. The script automatically detects and removes these before creating the correct bridge configuration.

---

## Default Credentials

All nodes use `root` as the username.

| Node | Password | WiFi SSID | WiFi Password |
|------|----------|-----------|---------------|
| Gate (green) | `havengreen` | `green-5ghz` | `green-5ghz` |
| Gate (green) 2.4GHz | — | `green` | `greengreen` |
| Point (blue) | `havenblue` | `blue` (2.4GHz USB; `setup-haven-point.sh`) | `blueblue` |
| Heltec | `heltec.org` | varies | varies |

---

## Configuration Reference

### Gate Node Defaults (green)

| Setting | Default | Description |
|---------|---------|-------------|
| `HOSTNAME` | green | Node hostname |
| `ROOT_PASSWORD` | havengreen | SSH/LuCI password |
| `MESH_ID` | haven | Mesh network name |
| `MESH_KEY` | havenmesh | Mesh encryption key |
| `MESH_IP` | 10.41.0.1 | Initial node IP (openmanetd may reassign) |
| `HALOW_CHANNEL` | 28 | HaLow channel (916 MHz) |
| `HALOW_HTMODE` | HT80 | Channel width (8 MHz) |

### Point Node Defaults (blue)

| Setting | Default | Description |
|---------|---------|-------------|
| `HOSTNAME` | blue | Node hostname |
| `ROOT_PASSWORD` | havenblue | SSH/LuCI password |
| `MESH_IP` | 10.41.0.2 | Initial node IP (openmanetd may reassign) |
| `GATEWAY_IP` | 10.41.0.1 | Initial gate node IP (openmanetd may reassign) |

> **Note:** OpenMANET's address reservation system manages mesh IPs on all nodes after setup. The defaults above are initial values — the final IPs may differ. Run `uci get network.ahwlan.ipaddr` on any node to find its current mesh IP, or check the boot screen on a connected monitor. To discover the IPs of all mesh nodes from any node, run:
> ```
> strings /etc/openmanetd/openmanetd.db
> ```
> This prints each node's MAC address, hostname, and current mesh IP.
