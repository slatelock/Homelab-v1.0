# Network Design

## VLANs

| Network | VLAN ID | Subnet | Purpose |
| --- | --- | --- | --- |
| Main | 1 | 192.168.1.0/24 | Trusted household devices and PCs |
| IoT | 3 | 192.168.3.0/24 | Smart home devices and doorbell |
| Cameras | 4 | 192.168.4.0/24 | PoE cameras and NVR |
| Guest | 5 | 192.168.5.0/24 | Visitor Wi-Fi, fully isolated |

Switch ports are profiled per VLAN. Camera ports are pre-assigned to the Cameras VLAN, and each camera gets a DHCP reservation as it goes live.

## Firewall zones

UniFi zone-based firewall. Default posture: lower-trust zones can't start connections into higher-trust zones.

| Policy | Action | Why |
| --- | --- | --- |
| Main → IoT | Allow | Control smart devices from phones and PCs |
| Main → Cameras | Allow | View cameras and manage the NVR locally |
| IoT / Cameras → Main | Return traffic only | A compromised device can't reach personal devices |
| IoT ↔ Cameras | Block | No lateral movement between untrusted zones |
| Cameras → Internet | Block | Video stays local; no vendor cloud, no phoning home |
| VPN → Cameras | Allow | Remote viewing only through WireGuard |
| Guest → internal networks | Block | Guests get internet and nothing else |

Zone boundaries were tested from the Main, IoT and Guest networks.

## Wi-Fi

| SSID | Bands | Notes |
| --- | --- | --- |
| Main | 2.4 / 5 / 6 GHz | Primary household network |
| IoT | 2.4 GHz only | Most IoT devices are 2.4 GHz-only, and this avoids band-steering issues |
| Guest | 2.4 / 5 GHz | Password-protected, client isolation on |

Guests join by scanning a 3D-printed QR code with the network name and password baked in. No typing, no asking. The Guest VLAN is zone-isolated from everything else, and client isolation stops guest devices from seeing each other.

## RF plan

- **2.4 GHz:** locked to channels 1, 6 and 11, the only non-overlapping channels.
- **5 GHz:** DFS channels excluded, which avoids radar-triggered channel changes and the dropouts they cause. The AP runs on the clean 149–165 block at 80 MHz.
- **6 GHz:** 320 MHz width, for Wi-Fi 7 clients.

## Other hardening

- **mDNS proxy** scoped to Main and IoT only, so casting and device discovery work across those two networks without leaking into Cameras or Guest.
- **IDS/IPS** (intrusion detection/prevention) in notify-and-block mode, scoped to the IoT, Cameras and Guest networks.
- **UPnP off**, so no device can open ports on its own.
