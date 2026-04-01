# Clinic Network Design — Cisco Packet Tracer Lab

A simulated small clinic network built in Cisco Packet Tracer, designed with HIPAA network segmentation requirements in mind. Includes full device configurations, an IP addressing plan, and documentation of key security design decisions.

---

## Network Overview

This lab simulates the network infrastructure of a small multi-department medical clinic — **Northside Family Clinic** — serving clinical staff, administrative/billing staff, patients (guest Wi-Fi), and networked medical devices.

The design prioritizes:
- **VLAN segmentation** — isolating clinical, administrative, guest, and medical device traffic
- **Access control** — preventing guest/patient traffic from reaching clinical systems
- **Least-privilege management** — SSH-only device access on a dedicated management VLAN
- **Realistic addressing** — structured IP plan with static assignments for critical devices

---

## Topology

```
                    [ Internet / ISP ]
                           |
                    203.0.113.2/30
                           |
                  [ Clinic-Router-01 ]
                   Cisco 2911
                   Inter-VLAN routing
                   NAT / ACL enforcement
                           |
                   802.1Q trunk
                           |
                  [ Core-Switch-01 ]
                   Cisco 2960
          __________|_________________________
         |         |          |         |         |
    VLAN 10    VLAN 20    VLAN 30    VLAN 40    VLAN 99
   Clinical    Admin      Guest      IoMT       Mgmt
  10.10.10/24 10.10.20/24 10.10.30/24 10.10.40/24 10.10.99/24
```

---

## VLAN Design

| VLAN | Name | Subnet | Purpose |
|------|------|--------|---------|
| 10 | Clinical-Staff | 10.10.10.0/24 | EHR workstations, clinical laptops, servers |
| 20 | Administrative | 10.10.20.0/24 | Billing, scheduling, front desk |
| 30 | Guest-Patient-WiFi | 10.10.30.0/24 | Patient/visitor internet — fully isolated |
| 40 | Medical-Devices-IoMT | 10.10.40.0/24 | Imaging, vital sign monitors — static IPs only |
| 99 | Management | 10.10.99.0/24 | Network device management via SSH |

---

## Security Design Decisions

### 1. VLAN Segmentation (HIPAA §164.312(a)(1))
All traffic types are isolated into dedicated VLANs. A billing workstation on VLAN 20 cannot directly communicate with a clinical workstation on VLAN 10 without traversing the router — where ACLs are enforced.

### 2. Guest Network Isolation (ACL)
An extended ACL (`BLOCK_GUEST_TO_CLINICAL`) applied inbound on the guest subinterface explicitly denies VLAN 30 traffic from reaching VLANs 10, 20, or 40. Guest devices can only reach the internet via NAT.

```
ip access-list extended BLOCK_GUEST_TO_CLINICAL
 deny ip 10.10.30.0 0.0.0.255 10.10.10.0 0.0.0.255
 deny ip 10.10.30.0 0.0.0.255 10.10.20.0 0.0.0.255
 deny ip 10.10.30.0 0.0.0.255 10.10.40.0 0.0.0.255
 permit ip any any
```

### 3. Medical Device Isolation (VLAN 40 — Static IPs Only)
IoMT devices (imaging systems, vital sign monitors) are on a dedicated VLAN with no DHCP — static assignments only. This prevents an unknown device from obtaining an IP and masquerading as clinical equipment.

### 4. NAT Restriction
Only VLANs 10 and 20 are permitted through the NAT ACL — clinical and administrative traffic reaches the internet. Guest traffic is handled separately. VLAN 40 (medical devices) has no internet access by design.

### 5. SSH-Only Management (VLAN 99)
All device management (router, switch) occurs exclusively over SSH on the dedicated management VLAN. Telnet is explicitly disabled. Devices are only reachable from VLAN 99.

### 6. PortFast on Access Ports
STP PortFast is enabled on all end-device access ports to eliminate the 30-second STP convergence delay on workstation ports, without enabling PortFast on trunk or uplink ports.

---

## Files

```
clinic-network-design/
├── configs/
│   ├── Clinic-Router-01.txt    # Full router CLI config
│   └── Core-Switch-01.txt      # Full switch CLI config
├── docs/
│   └── ip-address-plan.md      # Full IP/VLAN/DHCP addressing plan
└── README.md
```

> **Packet Tracer file**: Open Cisco Packet Tracer, recreate the topology using the configs and IP plan in this repo. A `.pkt` file is not included as it requires Packet Tracer to open.

---

## How to Recreate in Packet Tracer

1. Add a **Cisco 2911 router** and a **Cisco 2960 switch**
2. Connect them with a crossover (or straight-through) cable on Gig0/0 ↔ Gig0/1
3. Paste `Clinic-Router-01.txt` into the router CLI (enable → config t → paste)
4. Paste `Core-Switch-01.txt` into the switch CLI
5. Add end devices (PCs, servers, access point) and assign them to the correct VLANs per the IP address plan
6. Test with `ping` across VLANs to verify routing and ACL enforcement

---

## Technologies Used

- **Cisco Packet Tracer** — network simulation
- **Cisco IOS** — router and switch CLI configuration
- **TCP/IP** — Layer 3 addressing and routing
- **VLANs (802.1Q)** — Layer 2 traffic segmentation
- **Router-on-a-Stick** — inter-VLAN routing via subinterfaces
- **ACLs (Extended)** — access control between VLANs
- **NAT/PAT** — internet access with address translation
- **DHCP** — dynamic addressing per VLAN scope
- **SSH v2** — secure management access
- **STP PortFast** — optimized convergence on access ports

---

## HIPAA Relevance

This design reflects real requirements for networked healthcare environments:

- **Network segmentation** protects ePHI by limiting which devices can communicate with clinical systems
- **Guest isolation** ensures patients using clinic Wi-Fi cannot access internal resources
- **IoMT segmentation** protects medical devices from lateral movement attacks
- **Management VLAN** limits attack surface for device administration
- **ACL enforcement at the router** provides a central policy point for inter-VLAN traffic control
