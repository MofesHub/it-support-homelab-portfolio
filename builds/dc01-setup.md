# DC01 Build — Windows Server 2022 Standup

**Role:** the company's main server / domain controller
**Date:** June 30, 2026

## Summary
Built DC01 from scratch in VMware Workstation Pro: created the VM, installed
Windows Server 2022 Standard (Desktop Experience), and verified a clean boot
to Server Manager. This is the baseline state before AD DS / DNS / DHCP
configuration.

## VM Specs
- 2 vCPU, 4 GB RAM, 60 GB disk (NVMe, split into multiple files)
- Network adapter: Custom (VMnet1) — host-only, isolated lab network
- Firmware: UEFI, Secure Boot disabled

## Build Notes
- Used "I will install the operating system later" during VM creation to
  avoid VMware's Easy Install automation — this skips the real Windows Server
  Setup screens, which defeats the purpose of a learning lab.
- Selected **Standard edition with Desktop Experience** over Datacenter —
  Standard is the realistic choice for a small-company domain controller;
  Datacenter is built for heavy virtualization hosts and unlimited VM
  licensing, which doesn't apply here.
- Confirmed the evaluation ISO was genuinely Server 2022 before installing —
  Microsoft's evaluation ISOs across versions share the same generic filename
  (`SERVER_EVAL_x64FRE_en-us.iso`), so the version isn't visible from the
  filename alone. Verified via the Setup screen itself.
- VM files stored at `C:\VirtualMachines` rather than inside a OneDrive-synced
  folder, to avoid OneDrive trying to sync a multi-GB disk file that's
  actively in use.

## Screenshots
![Edition picker](dc01-setup/screenshots/edition-picker.png)
*Selected Standard edition with Desktop Experience over Datacenter — realistic choice for a small-company DC.*

![Custom install](dc01-setup/screenshots/custom-install.png)
*Custom: Install Windows only (advanced).*

![Disk partitioning](dc01-setup/screenshots/disk-partitioning.png)
*Auto-created partitions on the 60 GB disk.*

![Server Manager dashboard](dc01-setup/screenshots/server-manager-dashboard.png)
*Clean post-install desktop, before any role configuration — this is the baseline state.*

## License Status Check — July 1–2, 2026
### Summary
Checked the Windows Server 2022 Evaluation license clock before building further on top of `DC01`. Discovered the evaluation had never actually completed online activation, then diagnosed and fixed a network misconfiguration that was blocking it.

### Findings
- `slmgr /dlv` showed **License Status: Initial grace period**, only 10 days remaining, with rearm count untouched at 6/6 — meaning the evaluation had never gone through real online activation.
- `slmgr /ato` failed with `0x80072EE7` (`DNS (Domain Name System)`/connectivity failure) — `DC01` had no route to Microsoft's activation servers, expected since it sits on host-only `VMnet1` with no internet path.
- Added a second `NIC (Network Interface Card)` to the VM, set to `NAT (Network Address Translation)`, for temporary internet access. Retried activation and got `0xC004E028` repeatedly.
- Diagnosed with `ping` and `nslookup` — both failed completely, ruling out "activation still processing" and pointing to a deeper connectivity gap.
- Root cause: the **primary network adapter had drifted onto NAT instead of `VMnet1`**, so `DC01`'s main adapter was never properly reaching a gateway on the lab network — this, not the activation service, is what broke activation from the start.

### Fix
- Reconfigured the primary **Network Adapter** back to **Custom: VMnet1**.
- Kept **Network Adapter 2** on **NAT**, dedicated to giving `DC01` temporary internet access for activation.

### Status
Pending final verification — retesting `ping` and activation with the corrected adapter config next session.

### Screenshots
![Initial grace period](dc01-setup/screenshots/license-initial-grace-period.png)
*`slmgr /dlv` showing Initial grace period, 10 days remaining, rearm counts untouched at 6/6.*
![DNS activation failure](dc01-setup/screenshots/activation-error-dns.png)
*`slmgr /ato` failing with 0x80072EE7 — no internet path from host-only VMnet1.*
![Activation pending error](dc01-setup/screenshots/activation-error-e028.png)
*Repeated 0xC004E028 after adding a NAT adapter — misleading symptom; real cause was the adapter 1 misconfiguration below.*
![Adapter misconfiguration found](dc01-setup/screenshots/network-adapter-nat-misconfig.png)
*VM Settings showing the primary Network Adapter incorrectly set to NAT instead of VMnet1.*

## Next Step
Promote DC01 to a domain controller via Active Directory Domain Services (AD DS).
