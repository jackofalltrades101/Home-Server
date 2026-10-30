 # Home Server

## Architecture

```
 Internet (ISP ONT)
                              │  ethernet · DHCP v4 + DHCPv6-PD /56
                              ▼
        ┌──────────────── OPNsense (VM) ── edge router / firewall ──────────────┐
        │   WAN: DHCP v4/v6             LAN: XXX.XXX.XX.0/24                    │
        └───────────────────────────────┬───────────────────────────────────────┘
                                         │  LAN bridge
        ┌────────────────────────────────┼──────────────────────────────────────────┐
        │  PROXMOX HOST   i7-8700K · 16 GB · NVMe + 4 TB                            │
        │   ├─ VM  docker-immich   → Immich private photo cloud (4 TB passthrough)  │
        │   ├─ VM  opnsense        → the router above                               │
        │   └─ VM  docker-apps     → public web apps · dynamic RAM                  │
        │         ├─ Fillmap  (Next.js SSR)  + slim Supabase                        │
        │         ├─ XXXXXX   (SPA + Node)   + slim Supabase                        │
        │         └─ cloudflared ── outbound-only tunnel ─┐                         │
        └──────────────────────────────────────────────────┼────────────────────────┘
                                                           │
   NetBird mesh (WireGuard) · remote admin, 0 ports     ▼
                                                    Cloudflare edge
                                                 CDN · Tunnel · Workers
                                                            │
                                      fillmap.app  ·  XXXXXX.XXX   ← public
```

## 1. Pi zero 2w KVM

Needed a way to remote into a headless server. Had a Pi zero running P4wnP1 A.L.O.A. on hand.

Components: USB HID keyboard + USB-Ethernet both over the gadget USB port.

Plan: Put pi into station mode with IP forwarding + nftables. Mac → Pi (Wi-Fi) → USB → server.

Issues:

- The documented SSH for P4wnP1 A.L.O.A. was incorrect. (root/toor)
  - Mounted the image's ext4 root offline with debugfs to read shadow, root was locked
  - P4wnP1 A.L.O.A. is built on Kali so I attempted to use the default "Kali" user and that worked.
- Clock was off by 7 hours.
- Wi-Fi power save was dropping packets

## 2. Proxmox home server

Headless setup with the above Pi KVM followed by SSH configuration & hardware health checks.

Set up the following services:

- Netbird VPN: Used WireGuard mesh Host as routing peer so all the VMs would be reachable over netbird, set up privilege policies etc.
- Immich: Photo & video library backup. VM 2
- Supabase / node.js stack for hosting web services locally (FillMap.App). VM 3

## 3. Home router

Promoted server to home router.

- OPNsense VM. VM 1
- Bridged NICs
- AP backup over USB
- Set up automatic cold boot recovery
- End to end IPv6.

## 4. Cloudflare tunnel

- Passed all outbound traffic through cloudflare

## 5. Red Team

- Full external scope of public IP, TLS, headers, & database access control.
