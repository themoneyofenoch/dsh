# DSH Lessons — Ammaniel's Agent

Issues hit on this machine, what caused them, and what actually fixed them.
Read this at session start alongside `agents.md` and `memory.md`.
Newest first. One entry per issue — symptom, root cause, exact fix, rule.

---

## 2026-10-06 — Crucial X9 Pro 4TB (exFAT) won't mount on macOS

**Drive**: `CT4000X9PROSSD9` — Crucial X9 Pro, 4 TB portable SSD, USB, external, fixed media.
GPT layout: `disk8s1` Microsoft Reserved (16.8 MB) + `disk8s2` Microsoft Basic Data (4.0 TB).

**Symptom**: Finder shows nothing. macOS dialog *"The disk you attached was not readable by this computer"*
(Eject / Initialize… / Ignore). `diskutil info /dev/disk8s2` → `Mounted: No`, no volume name,
`Volume Total Space: 0 B`. `/Volumes/` contains only `Macintosh HD`.

**What it is NOT**:
- **Not BitLocker.** GPT partition-type GUID read at offset 0x80 was
  `A2A0D0EB-E5B9-3344-87C0-68B6B72699C7` (Microsoft Basic Data).
  BitLocker would be `EBD0A0A2-B9E5-4433-87C0-68B6B72699C7` — absent.
- **Not a missing driver.** macOS reads exFAT and NTFS (read-only) natively.

**The actual fault**: the partition's data area is unreadable over USB.
- `sudo fsck_msdos -n /dev/disk8s2` → `could not read boot block (Input/output error)`
- `sudo dd if=/dev/disk8s2 bs=512 count=1` → `Input/output error`, 0 bytes transferred
- BUT the partition table itself read fine: `sudo dd if=/dev/disk8 bs=512 skip=2 count=2 | xxd`
  returned both GPT entries. First sectors OK, data area dead → enclosure/bridge/cable or failing SSD.

**The fix that worked before** (found in `~/.zsh_history:232`):
```bash
sudo mount_exfat -o ro /dev/disk4s2 /Volumes/X9Pro
```
The drive was exFAT, not NTFS. **The device number changes between boots** (was `disk4s2` then;
now `disk8s2`) — always re-check with `diskutil list external` before reusing an old command.
`-o ro` = strictly read-only, cannot damage the volume.

**Diagnostics used (all read-only)**:
```bash
diskutil list external
diskutil info /dev/disk8s2
sudo dd if=/dev/disk8 bs=512 skip=2 count=2 2>/dev/null | xxd | head -20   # GPT entries + type GUIDs
sudo fsck_msdos -n /dev/disk8s2                                            # -n = no repair
sudo dd if=/dev/disk8s2 bs=512 count=1 2>&1 | xxd | head -8                # partition boot area
sudo mkdir -p /Volumes/X9Pro && sudo mount_exfat -o ro /dev/disk8s2 /Volumes/X9Pro
```

**Rules**:
- **NEVER click Initialize / Erase / Format / Repair** on an unreadable disk — that is the only
  action that destroys the data. `Ignore` and `Eject` are safe.
- Every diagnostic is **read-only**: `dd if=` (never `of=`), `fsck -n`, `mount -o ro`.
- If the data matters and reads keep failing: **stop reading the drive** — each failed read wears a
  failing SSD. Move to a Windows PC (exFAT/NTFS are native there) with `chkdsk` in read-only mode.
- Try the **factory cable** and a **direct Mac port** (no hub/dock) before assuming the SSD is dead —
  charge-only USB-C cables produce exactly this I/O error.

**Note**: device identifiers (`disk8`) are not stable across reconnects. Always map fresh with
`diskutil list external` before acting on a remembered path.
