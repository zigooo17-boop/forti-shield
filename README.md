# 🛡️ FortiShield

**A multi-branch enterprise network secured with FortiGate firewalls, built and tested in PNETLab.**

FortiShield simulates a real-world enterprise: a **Main Branch** with a segmented core (Management / Data / Voice VLANs and a DMZ) and **three remote branches** connected through the internet with **IPsec site-to-site VPN**, centralized DHCP, captive-portal authentication, and traffic logging.

![Network Topology](assets/screenshots/01-topology.png)

---

## 📋 Table of Contents

- [Overview](#-overview)
- [Topology](#-topology)
- [IP Addressing](#-ip-addressing)
- [Features](#-features)
- [Lab Environment](#-lab-environment)
- [Implementation Notes](#-implementation-notes)
- [Verification & Testing](#-verification--testing)
- [Screenshots](#-screenshots)
- [Repository Structure](#-repository-structure)
- [Future Improvements](#-future-improvements)
- [Author](#-author)

---

## 🔎 Overview

| Item | Details |
|------|---------|
| **Project name** | FortiShield |
| **Platform** | PNETLab |
| **Firewalls** | 4 × FortiGate-VM64-KVM (FortiOS v7.0.3) |
| **Sites** | 1 Main Branch + 3 Remote Branches + DMZ |
| **Connectivity** | Internet / ISP cloud + IPsec site-to-site VPN |
| **Clients** | Virtual PCs (VPCS) and a Windows 7 host |

## 🗺️ Topology

The design is built around four FortiGate firewalls:

- **Fortinet_Main_Branch** – headquarters. Connects to the internet, the ISP cloud, the DMZ, and the internal core switch.
- **Fortinet_Branch1 / Branch2 / Branch3** – remote sites, each with its own LAN segments and internet-facing interfaces used to reach the main branch.

### Main Branch internal design

| Segment | VLAN | Gateway |
|---------|------|---------|
| Management | 10 | 10.10.10.1 |
| Data | 20 | 10.10.20.1 |
| Voice | 30 | 10.10.30.1 |
| DMZ | – | 192.168.50.1 (servers: 192.168.50.10, 192.168.50.20) |

A **Core Switch** connects the MGMT, Data, and Voice access switches. The **DMZ** is isolated on its own firewall interface (`port3`).

> All PCs obtain their addresses from the **DHCP server running on the FortiGate**.

## 🔢 IP Addressing

### Remote branch LANs

| Site | Subnet | Hosts |
|------|--------|-------|
| Branch 1 | 172.16.10.0/24 (gateway 172.16.10.1) | .10, .11, .50 |
| Branch 1 | 172.16.11.0/24 | .10, .11, .12 |
| Branch 2 | 172.16.12.0/24 (gateway 172.16.12.1) | .10, .11 |
| Branch 2 | 172.16.13.0/24 (gateway 172.16.13.1) | .10, .11, .12 |
| Branch 3 | 172.16.14.0/24 (gateway 172.16.14.1) | .10, .11 |
| Branch 3 | 172.16.15.0/24 (gateway 172.16.15.1) | .10, .11, .12 |

### Firewall uplinks (as labeled in the topology)

| Firewall | Internet link | ISP-cloud link |
|----------|---------------|----------------|
| Main Branch | `port1` – 192.168.107.200 | `port2` – 192.168.10.10 |
| Branch 1 | `port4` – 192.168.107.100 | `port1` – 192.168.10.11 |
| Branch 2 | `port4` – 192.168.107.150 | `port1` – 192.168.107.20 |
| Branch 3 | `port4` – 192.168.107.250 | `port1` – 192.168.107.30 |

## ✨ Features

- **Multi-site design** – central headquarters with three remote branches
- **IPsec site-to-site VPN** created with the FortiOS IPsec Wizard (*Site to Site – FortiGate* template, pre-shared key authentication, Dead Peer Detection set to *On Demand*)
- **Auto-generated VPN objects** – phase 1/phase 2 interfaces, local/remote address groups, static and blackhole routes, and bidirectional firewall policies
- **VLAN segmentation** at the main branch (Management, Data, Voice)
- **DMZ zone** isolating the servers
- **Centralized DHCP** served by the FortiGate
- **Captive-portal authentication** for Branch 1 users
- **Internet access policy** (`port2 → port4`) for branch clients
- **Traffic logging** with *Log & Report → Forward Traffic*

## 🧰 Lab Environment

| Component | Details |
|-----------|---------|
| Lab platform | PNETLab |
| Firewall OS | FortiOS 7.0.3 (FortiGate-VM64-KVM) |
| Switching | Virtual switches (Core, MGMT, Data, Voice, and branch access switches) |
| End devices | VPCS nodes, Windows 7 VM |
| Host tools | Chrome, PuTTY, Kiwi Syslog Server, TFTPD32 |

## ⚙️ Implementation Notes

1. **Interfaces & addressing** are configured on each FortiGate (internet, ISP, LAN, DMZ).
2. **DHCP servers** run on the LAN interfaces so clients receive IP, gateway, and DNS automatically.
3. **IPsec tunnels** (for example `VPN_BR1_MAIN`) are created with the *IPsec Wizard*, which generates the address groups, routes, and policies.
4. **Internet policy** `port2 → port4` provides outbound access for branch clients.
5. **Authentication** is enforced through the FortiGate captive portal.

## ✅ Verification & Testing

- **Routing between subnets**: a host in Branch 2 (`172.16.13.10`) reaches a host in the neighboring subnet (`172.16.12.10`) and the internet (`8.8.8.8`).
- **Internet access**: the Windows client browses external sites, and the sessions appear in *Forward Traffic* with policy ID and byte counters.
- **Authentication**: the captive portal (`172.16.10.1:1000`) prompts for credentials.
- **VPN setup**: the wizard summary confirms the tunnel, address groups, routes, and policies were created.

## 📸 Screenshots

### Network topology
![Topology](assets/screenshots/01-topology.png)

### IPsec VPN
| Tunnel settings | Wizard: policy & routing |
|---|---|
| ![IPsec tunnel](assets/screenshots/02-ipsec-tunnel-settings.png) | ![VPN wizard policy](assets/screenshots/03-vpn-wizard-policy-routing.png) |

![VPN wizard summary](assets/screenshots/04-vpn-wizard-summary.png)

### Monitoring & authentication
| Forward traffic logs | Captive portal |
|---|---|
| ![Forward traffic](assets/screenshots/05-forward-traffic-logs.png) | ![Captive portal](assets/screenshots/06-captive-portal-auth.png) |

### Connectivity tests
![Ping test](assets/screenshots/07-connectivity-ping-test.png)

| Client web test (YouTube) | Client web test (Facebook) |
|---|---|
| ![YouTube test](assets/screenshots/08-client-web-test.png) | ![Facebook test](assets/screenshots/09-client-facebook-test.png) |

### Windows host with VPN client
![VPN client](assets/screenshots/10-vpn-client.png)

## 📁 Repository Structure

```
forti-shield/
├── README.md
├── .gitignore
└── assets/
    └── screenshots/      # lab screenshots used in this README
```

## 🚀 Future Improvements

- Publish sanitized FortiGate configuration files for each firewall
- Add Web Filter, Application Control, and IPS profiles
- Add SD-WAN with redundant links
- Add High Availability (HA) at the main branch
- Centralized logging with FortiAnalyzer / Syslog

## 👤 Author

**Your Name**
GitHub: [Yahia mohamed](https://github.com/zigooo17-boop)

---

> ⚠️ Lab environment for learning and demonstration only. Never publish real pre-shared keys or production configurations.
