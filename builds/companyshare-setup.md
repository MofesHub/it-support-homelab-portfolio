# CompanyShare Build — Shared Drive, Group, and GPO Drive Mapping
**Role:** the company's shared department drive
**Date:** July 14, 2026

## Summary
Built a shared folder on `DC01 (the company's main server)` with security-group-based
access and a `GPO (Group Policy Object)`-driven drive mapping on
`WIN11-CLIENT`, to support the `Files & Permissions` category of tickets
(`TICKET-004`, `TICKET-005`).

## Share Creation
- Folder: `C:\Shares\CompanyShare` on `DC01`
- Shared as `CompanyShare`, share-level permissions set broad (`Everyone`:
  Change) — standard real-world practice, since the actual access gate is
  `NTFS (NT File System)` permissions, not the share layer.

## Security Group
- Created `Staff` (Global, Security) in Active Directory Users and
  Computers.
- Added jsmith as a member.

## NTFS Permissions
- On `C:\Shares\CompanyShare`, granted `Staff` Modify access.
- Removed the default `Users`/`Authenticated Users` entries so `Staff`
  membership is the only path to access — the two-layer model (share +
  `NTFS`, most restrictive wins) is intentional here to demonstrate that
  concept.

## GPO Drive Mapping
- Created `Drive Mapping - CompanyShare`, linked to the `OU
  (Organizational Unit)` containing `WIN11-CLIENT`'s user accounts.
- User Configuration > Preferences > Windows Settings > Drive Maps:
  `\\DC01\CompanyShare` mapped to `S:`, with item-level targeting scoped
  to the `Staff` security group.
- **Gotcha worth remembering:** `Group Policy Preferences (GPP)` items
  don't remove themselves by default when a user falls out of the
  targeting scope — the mapping just stops being actively maintained, but
  stays in place. Had to explicitly enable **"Remove this item when it is
  no longer applied"** on the Common tab for the mapping to actually
  disappear when `Staff` membership is removed, which is what a real
  access-loss scenario requires.
- Separately, this only works at all if the target user account is inside
  an `OU` in the first place — `GPO`s can't link to the default AD
  containers (like "Users"). This tripped up initial testing; see
  `TICKET-004` for the full diagnostic path.

## Verification
- Confirmed baseline: `S:` drive present and accessible for jsmith with
  `Staff` membership intact.
- Confirmed removal: `S:` drive correctly disappears after removing
  `Staff` membership and a fresh logon (not just `gpupdate /force`).
- Confirmed restoration: `S:` drive returns after re-adding `Staff`
  membership and a fresh logon.
