# TICKET-013 — Can't Load Websites (DNS Resolution Failure via LAN Firewall Rule)

| Field | Detail |
|---|---|
| **Status** | Resolved |
| **Priority** | High |
| **Category** | Networking / DNS / Firewall |
| **Affected System** | `WIN11-CLIENT (an employee's laptop I'm troubleshooting)` and `DC01 (the company's main server)` — external name resolution |
| **Reporter** | jsmith (simulated employee) |
| **Ticketing system** | Jira Service Management — HIS-11 |
| **Date Opened / Closed** | July 18, 2026 (same day) |

## Summary
`jsmith` reported that websites would not load, despite general internet
connectivity appearing to work. Diagnosis traced the fault to a firewall rule
on `PFSENSE (the company's firewall, router, and VPN gateway)`'s `LAN (Local
Area Network)` interface blocking outbound `DNS (Domain Name System)` traffic
(port 53), preventing `DC01` — the internal DNS server — from resolving any
external hostname.

## Symptoms
- Websites failed to load in the browser on `WIN11-CLIENT`.
- `ping 8.8.8.8` from `WIN11-CLIENT` succeeded with 0% packet loss, ruling out
  a general connectivity or routing failure.
- `nslookup google.com` from `WIN11-CLIENT` timed out completely
  ("DNS request timed out").
- `nslookup google.com` run directly on `DC01` also failed — isolating the
  fault to somewhere between `DC01` and the internet, rather than a
  client-side DNS misconfiguration on `WIN11-CLIENT`.

## Diagnostic Steps
1. Confirmed raw connectivity was healthy (`ping 8.8.8.8` succeeded),
   narrowing the fault specifically to name resolution.
2. Confirmed `nslookup` failed identically on both `WIN11-CLIENT` and `DC01`
   itself — ruling out a client-specific DNS setting and pointing at the path
   between `DC01` and the internet (`PFSENSE`).
3. Checked `PFSENSE`'s firewall logs for `DC01`'s DNS queries. Initial checks
   against **Status > System Logs > Firewall > Normal View** on the WAN
   interface showed no matching entries — a false lead, since the block rule
   lived on the LAN interface, not WAN.
4. Confirmed via **Firewall > Rules > LAN** that a rule blocking port 53
   `TCP (Transmission Control Protocol)`/`UDP (User Datagram Protocol)`
   traffic was present and enabled.
5. The rule initially produced no log entries even after confirming it was
   active — traced to the rule's **"Log packets that are handled by this
   rule"** option being unchecked by default for custom rules (unlike
   `PFSENSE`'s implicit default-deny rule, which always logs). Enabled
   logging on the rule.
6. With logging enabled, **Status > System Logs > Firewall > Dynamic View**
   showed live, repeated blocked entries: source `192.168.199.10` (`DC01`),
   destination port 53, interface `LAN`, matching `DC01`'s recursive resolver
   retrying multiple upstream DNS servers — all blocked identically.

## Root Cause
A firewall rule on `PFSENSE`'s `LAN` interface was blocking all outbound
`DNS` traffic (port 53) originating from the internal network, preventing
`DC01`'s recursive DNS resolver from reaching any external DNS server.

## Resolution
Deleted the blocking rule from **Firewall > Rules > LAN** and applied
changes. Verified `nslookup google.com` resolved successfully on
`WIN11-CLIENT` immediately afterward.

## Verification
`nslookup google.com` returned a valid result on `WIN11-CLIENT` post-fix,
confirming external name resolution restored.

## Notes
An early diagnostic misstep is worth recording honestly: the block rule was
initially created on `PFSENSE`'s `WAN` interface rather than `LAN`. Since
`DC01`'s outbound DNS queries *enter* `PFSENSE` via LAN and are only routed
out through WAN afterward, a WAN-side rule has no effect on that traffic —
pfSense filters based on the interface where a packet enters the firewall,
not where it exits. The rule had to be recreated on the LAN interface to
actually reproduce the fault. This is a useful real-world lesson: firewall
rule placement (LAN vs. WAN) determines what traffic a rule actually catches,
and getting it wrong can look identical to "the fix isn't working" until the
interface itself is questioned.
