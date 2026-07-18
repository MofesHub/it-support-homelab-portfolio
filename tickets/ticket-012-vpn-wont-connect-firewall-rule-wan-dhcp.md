# TICKET-012 — VPN Won't Connect (Disabled Firewall Rule + WAN DHCP Lease Loss)

| Field | Detail |
|---|---|
| **Status** | Resolved |
| **Priority** | High |
| **Category** | Networking / Remote Access / Firewall |
| **Affected System** | `PFSENSE (the company's firewall, router, and VPN gateway)` — OpenVPN remote access service |
| **Reporter** | jsmith (simulated employee) |
| **Ticketing system** | Jira Service Management — HIS-10 |
| **Date Opened / Closed** | July 17, 2026 (same day) |

## Summary
`jsmith`, working remotely, reported being unable to connect to the company
`VPN (Virtual Private Network)` — connection attempts failed with a generic
timeout and no further detail. Diagnosis uncovered **two independent faults**
that had to be isolated separately: the `WAN (Wide Area Network)` firewall
rule permitting OpenVPN traffic had been disabled, and separately, `PFSENSE`'s
`WAN` interface had lost its `DHCP (Dynamic Host Configuration Protocol)`
lease following a VM suspend/resume, leaving it with no `IP (Internet
Protocol)` address at all. Both had to be found and fixed before the VPN
would connect again.

## Symptoms
- OpenVPN Connect client: "Connection Timeout — connection failed to
  establish within given time," both before and immediately after the first
  fix.
- No corresponding entries in `PFSENSE`'s OpenVPN log for any connection
  attempt — the traffic was never reaching the OpenVPN process.

## Diagnostic Steps
1. Checked **Status > System Logs > OpenVPN** for the connection attempt —
   found nothing logged, indicating the traffic was being dropped before
   reaching the OpenVPN service rather than being rejected by it (which
   would have logged a `TLS (Transport Layer Security)`/auth failure).
2. Checked **Firewall > Rules > WAN** and found the `OpenVPN HOMELAB-RA-VPN
   wizard` rule (port `1194`) disabled.
3. Cross-checked **Status > System Logs > Firewall** and found repeated
   `Default deny rule IPv4` entries matching the connection attempts,
   confirming the disabled rule was causing a silent drop at the firewall.
4. Re-enabled the rule and applied changes — but a retry still produced the
   same timeout, with **no new firewall deny entries** at all for the
   attempt. This absence, on its own, was the signal that a second, different
   fault was now in play.
5. Checked **Status > Dashboard** and found the `WAN` interface showing
   **n/a** for its `IP` address — link up, but no address assigned. This
   explained the second timeout: with no `WAN` address, the client's
   connection attempt had nowhere valid to land, so it never reached
   `PFSENSE` to be logged as blocked or accepted.

## Root Cause
Two unrelated faults overlapping in the same window:
1. **Deliberate:** the WAN firewall rule for OpenVPN (port 1194 `UDP (User
   Datagram Protocol)`) was disabled.
2. **Incidental:** `PFSENSE`'s `WAN` interface, configured for `DHCP`, failed
   to hold or renew its lease from the VMware `NAT (Network Address
   Translation)` service across a VM suspend/resume cycle earlier in the
   session, leaving `WAN` with no `IP` address.

## Resolution
1. Re-enabled the `OpenVPN HOMELAB-RA-VPN wizard` rule under **Firewall >
   Rules > WAN**, applied changes.
2. Re-saved the `WAN` interface's existing `DHCP` configuration under
   **Interfaces > WAN** to force a fresh lease request, which restored the
   `WAN` address to `192.168.154.131`.
3. Verified via **Status > Dashboard** that `WAN` showed a valid address
   again.
4. Retried the connection from OpenVPN Connect — client reported "Securely
   Connected!" with an active tunnel.

## Verification
OpenVPN Connect showed "Securely Connected!" with live throughput, alongside
`PFSENSE`'s Dashboard confirming `WAN` restored to `192.168.154.131` — both
visible in the same screenshot.

## Notes
The two faults were easy to conflate at first glance — both presented as an
identical client-side timeout with no distinguishing error text. The
distinguishing signal was the firewall log: the first timeout showed matching
`Default deny` entries at `:1194`, while the second timeout showed none at
all, which is what pointed away from the firewall and toward the interface
itself having no address to receive traffic on. A real-world parallel: never
assume a single fix has fully resolved an issue just because the underlying
cause looks addressed — re-verify after every change.
