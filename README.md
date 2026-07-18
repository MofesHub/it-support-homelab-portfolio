# IT Support Home Lab Portfolio

A self-built lab simulating a small company's IT environment — `DC01` (domain
controller), `WIN11-CLIENT` (employee laptop), `PFSENSE` (firewall/router), `UBUNTU-WAZUH` (security monitoring), plus a Microsoft 365 cloud tenant.

I deliberately break common help-desk issues, log them as tickets, diagnose
and resolve them on a structured path, and document the full process here.

## Tickets Resolved
| #   | Ticket                          | Category                        | Status         |
| --- | -------------------------------- | -------------------------------- | -------------- |
| 000 | [DC01 license activation failure](tickets/ticket-000-dc01-license-activation-failure.md) | Server / Licensing / Networking | ✅ Resolved     |
| 001 | [Account lockout](tickets/ticket-001-jsmith-account-lockout.md) | Identity & Access | ✅ Resolved |
| 002 | [jsmith MFA sign-in failure](tickets/ticket-002-jsmith-mfa-signin-failure.md) | Identity & Access | ✅ Resolved |
| 003 | [WIN11-CLIENT high CPU — remote support](tickets/ticket-003-win11-client-high-cpu-remote-support.md) | Endpoint Performance / Desktop Support | ✅ Resolved |
| 004 | [CompanyShare unreachable — SMB/GPO diagnostic](tickets/ticket-004-companyshare-smb-gpo-diagnostic.md) | Files & Permissions / Group Policy | ✅ Resolved |
| 005 | [jsmith loses S: drive access — group membership](tickets/ticket-005-jsmith-drive-access-group-membership.md) | Files & Permissions | ✅ Resolved |
| 006 | [jsmith deleted file — Volume Shadow Copy recovery](tickets/ticket-006-jsmith-deleted-file-shadow-copy-recovery.md) | Backup & Recovery | ✅ Resolved |
| 009 | [jsmith can't print — Print Spooler stopped](tickets/ticket-009-jsmith-cant-print-spooler-stopped.md) | Print | ✅ Resolved |
| 010 | [DC01 no internet after gateway cutover](tickets/ticket-010-dc01-no-internet-after-gateway-cutover.md) | Networking / Routing | ✅ Resolved |
| 011 | [VMnet1 host adapter IP conflict with pfSense LAN gateway](tickets/ticket-011-vmnet1-pfsense-ip-conflict.md) | Networking / Virtual Infrastructure | ✅ Resolved |
| 012 | [VPN won't connect — firewall rule + WAN DHCP lease loss](tickets/ticket-012-vpn-wont-connect-firewall-rule-wan-dhcp.md) | Networking / Remote Access / Firewall | ✅ Resolved |
| 013 | [Can't load websites — DNS resolution failure via LAN firewall rule](tickets/ticket-013-dns-resolution-failure-lan-firewall-rule.md) | Networking / DNS / Firewall | ✅ Resolved |


## Lab Architecture
See [docs/lab-architecture.md](docs/lab-architecture.md)

## Infrastructure Build Log
| Component                  | What It Does                        | Status         | Details |
| --------------------------- | ------------------------------------ | -------------- | ------- |
| DC01 — Windows Server 2022 | Domain controller: AD DS, DNS, DHCP | ✅ Complete     | [builds/dc01-setup.md](builds/dc01-setup.md) · [builds/dc01-addc.md](builds/dc01-addc.md) · [builds/dc01-dhcp.md](builds/dc01-dhcp.md) |
| WIN11-CLIENT — Windows 11  | Employee laptop, domain-joined      | ✅ Complete    | [builds/win11-client-setup.md](builds/win11-client-setup.md) |
| Entra ID tenant             | Cloud identity, Conditional Access, MFA | ✅ Complete | [builds/entra-id-tenant-setup.md](builds/entra-id-tenant-setup.md) |
| CompanyShare                | Shared drive: Staff group, NTFS permissions, GPO drive mapping | ✅ Complete | [builds/companyshare-setup.md](builds/companyshare-setup.md) |
| PFSENSE                    | Firewall / router / VPN gateway     | ✅ Complete | [builds/pfsense-setup.md](builds/pfsense-setup.md) · [builds/pfsense-openvpn-setup.md](builds/pfsense-openvpn-setup.md) |
| UBUNTU-WAZUH               | Security monitoring server          | ⬜ Not started | — |
