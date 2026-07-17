# TICKET-011 — Host VMnet1 Adapter Conflicts with pfSense LAN Gateway Address

| Field | Detail |
|---|---|
| **Status** | Resolved |
| **Priority** | Medium |
| **Category** | Networking / Virtual Infrastructure |
| **Affected System** | `PFSENSE (the company's firewall, router, and VPN gateway)` LAN reachability from `WIN11-CLIENT (an employee's laptop I'm troubleshooting)` |
| **Reporter** | IT (self-generated during VPN build) |
| **Ticketing system** | Jira Service Management — HIS-9 |
| **Date Opened / Closed** | July 17, 2026 (same day) |

## Summary
While preparing to install a package on `PFSENSE`, the web `GUI (Graphical
User Interface)` became unreachable from `WIN11-CLIENT`, and a subsequent
reboot command appeared to hang indefinitely. Diagnosis found `PFSENSE` was
never actually down — the true fault was an `IP (Internet Protocol)` address
conflict: the **host machine's own VMware `VMnet1` virtual adapter** had
claimed `192.168.199.1`, the same address owned by `PFSENSE`'s
`LAN (Local Area Network)` interface. `WIN11-CLIENT` was resolving `.1` to the
host's adapter instead of `PFSENSE`, so pings and `GUI` requests hit a dead
end with no web server and no forwarding.

## Symptoms
- `https://192.168.199.1` timed out from `WIN11-CLIENT`'s browser
  (`ERR_CONNECTION_TIMED_OUT`).
- `ping 192.168.199.1` from `WIN11-CLIENT` timed out.
- A reboot triggered from the pfSense `GUI` appeared to never take effect —
  explained once it became clear the browser request never reached `PFSENSE`
  at all.
- `PFSENSE`'s own console showed a clean, fully-booted state the entire time
  (`Bootup complete`, both interfaces up on their correct addresses),
  ruling out an actual outage.

## Diagnostic Steps
1. Confirmed `WIN11-CLIENT`'s own networking was healthy: valid
   `192.168.199.50` address, correct mask, gateway, and a live
   `DHCP (Dynamic Host Configuration Protocol)` lease from `DC01`
   (`192.168.199.10`) — ruled out the client's adapter as the cause.
2. Noted the lease came from `DC01`, not `PFSENSE` — meaning client-to-`DC01`
   reachability said nothing about client-to-`PFSENSE` reachability.
3. Ran `ping 192.168.199.10` (succeeded) and `arp -a` on `WIN11-CLIENT` to
   compare same-subnet reachability against the `ARP (Address Resolution
   Protocol)` table.
4. In the `arp -a` output, found `192.168.199.1` resolved to
   `00-50-56-c0-00-01` — a VMware-reserved `MAC (Media Access Control)`
   address prefix, not the `00-0c-29` prefix used by the lab's VMs. This
   pointed to a **host-side** virtual adapter answering for `.1`, not
   `PFSENSE`.
5. Confirmed on the host: `ipconfig` showed **VMware Network Adapter
   VMnet1** holding `192.168.199.1` — a direct conflict with `PFSENSE`'s LAN
   interface.

## Root Cause
The host's `VMnet1` virtual adapter and `PFSENSE`'s LAN interface were both
configured to `192.168.199.1`. Windows on `WIN11-CLIENT` resolved `ARP`
requests for `.1` to whichever device answered — in this case the host
adapter, a dead end with no services running — rather than `PFSENSE`.

## Resolution
1. Opened VMware's **Virtual Network Editor** as Administrator, selected
   `VMnet1`, and disabled **"Use local DHCP service to distribute IP
   addresses to VMs"** (redundant now that `DC01` is the lab's `DHCP` server,
   and a second potential conflict source).
2. Set the host's `VMnet1` adapter to a non-conflicting static address:
   `netsh interface ip set address "VMware Network Adapter VMnet1" static
   192.168.199.2 255.255.255.0`.
3. Verified: `ping 192.168.199.1` from the host succeeded, confirming
   `PFSENSE` now owns `.1` cleanly.

## Verification
`WIN11-CLIENT` regained both `ping` and web `GUI` access to `PFSENSE` at
`192.168.199.1` immediately after the host adapter was moved to `.2`.

## Notes
No `PFSENSE` reboot ever actually occurred during this incident — the
console remained at the same boot state throughout, confirming the fault was
entirely a host-side addressing conflict rather than any instability on the
firewall itself.
