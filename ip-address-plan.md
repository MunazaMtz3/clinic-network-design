# Network Addressing Plan — Northside Family Clinic

## VLAN & Subnet Summary

| VLAN | Name                  | Subnet           | Gateway       | Purpose                                      |
|------|-----------------------|------------------|---------------|----------------------------------------------|
| 10   | Clinical-Staff        | 10.10.10.0/24    | 10.10.10.1    | EHR workstations, clinical laptops           |
| 20   | Administrative        | 10.10.20.0/24    | 10.10.20.1    | Billing, scheduling, front desk              |
| 30   | Guest-Patient-WiFi    | 10.10.30.0/24    | 10.10.30.1    | Patient/visitor internet only — isolated     |
| 40   | Medical-Devices-IoMT  | 10.10.40.0/24    | 10.10.40.1    | Imaging, monitoring devices, printers        |
| 99   | Management            | 10.10.99.0/24    | 10.10.99.1    | Network device management (SSH only)         |

---

## Static Device Assignments

| Device              | VLAN | IP Address    | Role                        |
|---------------------|------|---------------|-----------------------------|
| Clinic-Router-01    | 99   | 10.10.99.1    | Core router / gateway       |
| Core-Switch-01      | 99   | 10.10.99.2    | Core switch                 |
| EHR-Server-01       | 10   | 10.10.10.10   | EHR application server      |
| File-Server-01      | 10   | 10.10.10.11   | Clinical document storage   |
| Workstation-Clinical-01 | 10 | 10.10.10.20 | Clinical workstation        |
| Workstation-Clinical-02 | 10 | 10.10.10.21 | Clinical workstation        |
| Billing-PC-01       | 20   | 10.10.20.20   | Billing workstation         |
| Front-Desk-PC-01    | 20   | 10.10.20.21   | Scheduling workstation      |
| Wireless-AP-01      | 30   | 10.10.30.2    | Patient/guest Wi-Fi AP      |
| Imaging-Device-01   | 40   | 10.10.40.20   | Digital X-ray system        |
| Monitor-Device-01   | 40   | 10.10.40.21   | Vital signs monitor         |

---

## DHCP Scope Definitions

### VLAN 10 — Clinical Staff
- Range: 10.10.10.50 – 10.10.10.150
- Default Gateway: 10.10.10.1
- DNS: 10.10.10.10 (internal), 8.8.8.8 (fallback)
- Lease: 8 hours

### VLAN 20 — Administrative
- Range: 10.10.20.50 – 10.10.20.100
- Default Gateway: 10.10.20.1
- DNS: 10.10.10.10 (internal), 8.8.8.8 (fallback)
- Lease: 8 hours

### VLAN 30 — Guest / Patient Wi-Fi
- Range: 10.10.30.50 – 10.10.30.200
- Default Gateway: 10.10.30.1
- DNS: 8.8.8.8, 8.8.4.4 (public only — no internal DNS)
- Lease: 1 hour

### VLAN 40 — Medical Devices
- Static assignments only — no DHCP
- Prevents unexpected IP changes on clinical devices

---

## WAN / ISP

| Item              | Value             |
|-------------------|-------------------|
| ISP-assigned IP   | 203.0.113.2/30    |
| ISP Gateway       | 203.0.113.1       |
| NAT               | PAT (overload) on GigabitEthernet0/1 |
| Internet access   | VLAN 10 and 20 only (via NAT ACL)    |
