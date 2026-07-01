# IT Support Home Lab Portfolio

A self-built lab simulating a small company's IT environment — `DC01` (domain
controller), `WIN11-CLIENT` (employee laptop), `PFSENSE` (firewall/router),
`UBUNTU-WAZUH` (security monitoring), plus a Microsoft 365 cloud tenant.

I deliberately break common help-desk issues, log them as tickets, diagnose
and resolve them on a structured path, and document the full process here.

## Lab Architecture
See [docs/lab-architecture.md](docs/lab-architecture.md)

## Infrastructure Build Log
| Component | What It Does | Status | Details |
|---|---|---|---|
| DC01 — Windows Server 2022 | Domain controller: AD DS, DNS, DHCP | ✅ Complete | [builds/dc01-setup.md](builds/dc01-setup.md) · [builds/dc01-addc.md](builds/dc01-addc.md) · [builds/dc01-dhcp.md](builds/dc01-dhcp.md) |
| WIN11-CLIENT — Windows 11 | Employee laptop, domain-joined | ⬜ Not started | — |
| PFSENSE | Firewall / router / VPN gateway | ⬜ Not started | — |
| UBUNTU-WAZUH | Security monitoring server | ⬜ Not started | — |

## Tickets Resolved
| # | Ticket | Category | Status |
|---|--------|----------|--------|
| 001 | Account lockout | Identity & Access | ⬜ In Progress |
