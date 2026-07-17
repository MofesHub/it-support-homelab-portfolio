# pfSense OpenVPN Remote-Access Server — Build Notes

## Purpose
Stand up an OpenVPN remote-access `VPN (Virtual Private Network)` server on
`PFSENSE` [the company's firewall, router, and VPN gateway] to enable
remote-access break/fix scenarios. Establishes the working baseline that
TICKET-008 later breaks and repairs.

## Server configuration
- Built via **VPN > OpenVPN > Wizards**, authentication backend **Local User Access**.
- Server mode: **Remote Access (`SSL (Secure Sockets Layer)`/`TLS (Transport
  Layer Security)` + User Auth)** — requires both a client certificate and
  username/password.
- `CA (Certificate Authority)`: `HOMELAB-VPN-CA`; server certificate: `HOMELAB-VPN-SERVER`.
- Interface **WAN**, protocol **UDP on IPv4**, port **1194**.
- Tunnel Network **10.8.0.0/24**; Local Network **192.168.199.0/24** (so tunnel
  clients reach the internal `LAN (Local Area Network)`).
- Split tunnel (Redirect Gateway off); `DNS (Domain Name System)` pushed as
  **192.168.199.10** (`DC01`).
- Firewall rules for WAN and OpenVPN interfaces auto-created by the wizard.

## Client
- `jsmith` created in **System > User Manager** with an attached internal
  certificate (`jsmith-vpn-cert`, `CN (Common Name)` `jsmith`) — required
  because the export tool only lists users that hold a certificate.
- Config exported via the **openvpn-client-export** package as an inline `.ovpn`
  (CA, client cert, and `TLS` key embedded in one file).
- **Host Name Resolution override:** export target set to the WAN address
  **192.168.154.131**, not the pfSense hostname — the remote client is the
  VMware host, which reaches pfSense across the VMware
  `NAT (Network Address Translation)` segment.

## Remote client model
The Windows host acts as the remote employee laptop: it sits on the VMware NAT
segment (WAN side) and dials pfSense's WAN. Because the host also holds a
`VMnet1` adapter inside the LAN, baseline verification requires temporarily
disabling `VMnet1` so the only path to the LAN is the tunnel — otherwise a ping
to `DC01` crosses the local wire and gives a false positive.

## Baseline verified
OpenVPN Connect connected (tunnel adapter `10.8.0.2`); with `VMnet1` disabled,
`ping 192.168.199.10` (`DC01`) succeeded through the tunnel. Screenshot:
`pfsense-openvpn-screenshots/vpn-baseline-connected-ping-through-tunnel.png`.

## Gotcha
Initial **openvpn-client-export** install returned "Another instance of
pfSense-upgrade is running" — a transient package-manager lock triggered by the
background update check. Cleared on retry; no reboot required.
