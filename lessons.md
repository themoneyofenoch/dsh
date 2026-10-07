# DSH Lessons — Ammaniel's Agent

Issues hit on this machine, what caused them, and what actually fixed them.
Read this at session start alongside `agents.md` and `memory.md`.
Newest first. One entry per issue — symptom, root cause, exact fix, rule.

---

## 2026-10-06 — Crucial X9 Pro 4TB: DATA IS INTACT, only the boot sector is dead

**Status: THE FILES ARE STILL ON THE DRIVE. THE DRIVE IS NOT DYING.** Do not tell Ammaniel
otherwise — he was told the drive had failed several times during this session and it was wrong.

**Drive**: `CT4000X9PROSSD9` — Crucial X9 Pro, 4 TB portable SSD, USB.
GPT: `disk8s1` Microsoft Reserved (16.8 MB) + `disk8s2` Microsoft Basic Data (4.0 TB).
Partition 2 starts at LBA 32768; volume is 7,814,002,688 sectors.

### What is actually wrong

Only **16 sectors — 8 KB — at the start of the partition are unreadable** (LBA 32768–32783).
The filesystem boots sector lives there (exFAT keeps its backup at volume sector 12 = LBA 32780;
both are inside the dead zone). With no header, macOS cannot mount the volume — but the files
themselves are untouched and readable.

### Proof the data is intact (md5 of one 512-byte sector per LBA)

| LBA | md5 prefix | Meaning |
|---|---|---|
| 32760–32767 | distinct hashes | real data (end of MSR partition) |
| 32768–32783 | `d41d8cd9` | **failed read** — the 8 KB dead zone |
| 32784–32788 | `f018e29c` | constant, non-zero |
| 32789–32790 | `bf619eac` | zeros |
| 32791 | `26405f82` | constant |
| 32792–32800 | `bf619eac` | zeros |
| 40000 | `6046da91` | **real data** |
| 5,000,000 | `5630de1c` | **real data** |
| 50,000,000 | `4cd44de2` | **real data** |
| 500,000,000 | `55c9f1b7` | **real data** |
| 4,000,000,000 | `8b19432a` | **real data** |
| 6,000,000,000 | `2bccab13` | **real data** |
| last sector (7814035455) | `bf619eac` | zeros — so no NTFS backup header |

**Reference hashes** (computed locally, `dd | md5 -q`):
- `d41d8cd9` = **failed read** (empty input)
- `bf619eac` = **512 zero bytes**
- `acce58da` = 512 bytes of 0xFF

### The one diagnostic that cracked it — md5 map per sector

```bash
sudo -v
for lba in $(seq 32760 32800); do h=$(sudo dd if=/dev/rdisk8 bs=512 skip=$lba count=1 2>/dev/null | md5 -q); printf "%s %s\n" $lba "${h:0:8}"; done
```

This distinguishes *failed* / *blank* / *real data* in one pass. Use it on any suspect disk
BEFORE declaring hardware failure. A single `dd` sample is not enough — results at neighbouring
LBAs were inconsistent (32790 read, 32792 failed) until the full map was drawn.

### Read speed is the other clue

- `bs=1m`: **~9 MB/s** — works fine, big reads.
- `bs=512`, one sector at a time: **~76 sectors/sec** — brutally slow.

That is why PhotoRec crawled (23,322 sectors in 5m23s, ETA 30,061 hours): it reads sector by
sector. The drive is not slow — small reads over this USB bridge are. **Always carve with large
blocks on this hardware; do not judge a drive by PhotoRec's counter alone.**

### CORRECTIONS — mistakes made during this session, do not repeat

- **`EBD0A0A2-B9E5-4433-87C0-68B6B72699C7` is Microsoft Basic Data — NOT BitLocker.** This was
  misread twice (first as `A2A0D0EB-…`, then wrongly "corrected" to BitLocker). BitLocker is not
  identified by a GPT type GUID at all — it is identified by the `-FVE-FS-` signature inside the
  volume. Ammaniel has no Microsoft account, so BitLocker was never plausible.
- GPT GUIDs are mixed-endian: first 4 bytes little-endian, next two 2-byte fields little-endian,
  last 8 bytes as-is. Decode them properly before drawing conclusions.
- **Do not announce a root cause until it has been tested.** Three wrong diagnoses were delivered
  with confidence in one session (dying SSD → BitLocker → dying SSD). The md5 map should have come
  first.

### The path forward (as of end of session)

The filesystem type still needs confirming from surviving sectors — dump LBA 32784, 32791 and a
data sector (e.g. 40000) and read the signature. Then either reconstruct the boot sector
(exFAT: rebuild from known volume geometry; NTFS: restore from backup) or carve with large blocks.

### Hard rules

- **NEVER click Initialize / Erase / Format / Repair** on this disk. The data is recoverable —
  formatting is the one action that would destroy it.
- All diagnostics stay **read-only**: `dd if=` (never `of=`), `fsck -n`, `mount -o ro`, `md5`.
- Device identifiers are not stable across reconnects (this drive appeared as `disk4s2`, `disk8s2`,
  `disk10s2` in one session). Always re-map with `diskutil list external` first.
