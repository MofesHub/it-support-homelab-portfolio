# Lab Architecture

## Host Environment
- Host machine: Windows PC, 16 GB RAM, running `VMware Workstation Pro` (free personal use license)
- VMs run in "scenes" — only the machines needed for the current ticket are powered on at once

## Virtual Machines

### DC01 — Windows Server 2022
Role: the company's main server / domain controller
- `AD (Active Directory)` — identity and user management
- `DNS (Domain Name System)` — internal name resolution
- `DHCP (Dynamic Host Configuration Protocol)` — IP address assignment for clients

### WIN11-CLIENT — Windows 11
Role: an employee's laptop, joined to the DC01 domain. Used to simulate and troubleshoot end-user issues — login problems, network connectivity, file/printer access, performance.

### PFSENSE
Role: the company's firewall, router, and VPN gateway. Sits at the network edge, routes traffic between the internal lab network and the internet, and will host VPN configuration for remote-access scenarios.

### UBUNTU-WAZUH — Ubuntu Server running Wazuh
Role: the company's security monitoring server. Collects logs and alerts from DC01 and WIN11-CLIENT — this is the SOC/security analyst skill track.

## Network Layout

VM network adapter: **Host-only (VMnet1)** — VMware's default isolated network, shared with the host, not bridged to the internet.

Internet
   |
PFSENSE (firewall / router / VPN gateway)
   |
Internal LAN (192.168.X.0/24)
   |-- DC01 (domain controller: AD / DNS / DHCP)
   |-- WIN11-CLIENT (employee laptop)
   |-- UBUNTU-WAZUH (security monitoring)

*(IP ranges are placeholder until pfSense and DC01 are configured — update this section once static IPs are assigned.)*

## Cloud Stack
- `M365 (Microsoft 365)` Developer tenant — `Entra ID` (cloud identity), `Intune` (device management), `Exchange` (email)
- Ticketing system — Freshservice or ServiceNow, used to log and track every ticket end-to-end
- `RMM (Remote Monitoring and Management)` trial — NinjaOne or Atera, used to simulate remote device management
