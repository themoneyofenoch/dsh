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

---

## 2026-10-06 — How we actually got the files back (the recovery path, step by step)

The mount never worked and never will. **This is the sequence that recovered the data.**
If a drive won't mount but you need the files, copy this order exactly.

### Step 1 — Map the damage with md5-per-sector (do NOT trust one dd)

```bash
sudo -v
for lba in $(seq 32760 32800); do h=$(sudo dd if=/dev/rdisk8 bs=512 skip=$lba count=1 2>/dev/null | md5 -q); printf "%s %s\n" $lba "${h:0:8}"; done
```

Reference hashes, computed locally so you can decode the output:
| hash | meaning |
|---|---|
| `d41d8cd9` | **failed read** (empty input) |
| `bf619eac` | 512 zero bytes |
| `acce58da` | 512 bytes of 0xFF |

Result here: LBA 32768–32783 = `d41d8cd9` (16 dead sectors = 8 KB = exactly the boot region),
everything around it real data. **One bad 8 KB on a 4 TB drive.**

### Step 2 — THE KEY TEST: read speed by block size

```bash
time sudo dd if=/dev/rdisk8 bs=512 count=2000 of=/dev/null        # small reads
time sudo dd if=/dev/rdisk8 bs=1m  count=2048 of=/dev/null        # 2 GB, large reads
```

**This is the whole story.** Same drive, same moment:

| block size | speed |
|---|---|
| 512 bytes | ~76 sectors/sec (~38 KB/s) — looks like a dying drive |
| 1 MB | **444 MB/s** — a perfectly healthy SSD |

`444102093 bytes/sec` = healthy hardware. The drive was never failing.

**RULE: never diagnose a drive as dead from small-read tools.** `PhotoRec`, `dd bs=512` and
`fsck` all read in tiny blocks and make a healthy drive look broken. Always run the `bs=1m`
test before saying "hardware failure" to Ammaniel. PhotoRec's "ETA 30,061 hours" was measuring
the *block size*, not the drive.

### Step 3 — Carve with PhotoRec, pointed at a destination with room

```bash
sudo photorec /dev/disk8
```
Menus: `Proceed` → `2 P MS Data` (the big partition, **not** `Unknown`) → `Other` → `[Whole]` →
destination must be **on a different disk** → `Y` for file types.

PhotoRec needs no filesystem header — it identifies files by internal signature, which is exactly
why it works when mounting cannot.

### Step 4 — Results (proof the method works)

Recovered on the first pass, ~10 minutes in: **16,133 files** — 6,074 jpg, 5,790 apple,
**1,860 cr3 (Canon RAW)**, 1,803 xmp, 340 webp, 145 png. Full-pass ETA ~13 h at 444 MB/s.

Turned out to be a photographer's drive. Nothing had to be sent to a lab.

### Step 5 — Watch the destination, it is the real limit

Files land as `recup_dir.N/` next to the destination you picked (`recup_dir.1`, `.2`, …).
Recovery ran ~20 GB every 4 minutes — the Mac's 208 GB free would have filled in ~45 minutes,
long before the 4 TB finished. **Check `df -h` before starting and decide the destination first.**
The source drive is never written to; only the destination fills.

### The dead ends — do not repeat

- `mount_exfat -o ro` on `disk8s2` and `disk10s2` → `Input/output error` (exfat.kext loads fine)
- `diskutil mount`, `mount readOnly` → `failed to mount`; `diskutil mountDisk` → false "success"
- `fsck_exfat -y` → `Could not read boot region`; `fsck_msdos -n` → `Could not read boot block`
- `dd` on `/dev/disk8s2` and `/dev/rdisk8s2` → `Input/output error`, **0 bytes**
- Alternate HFS+ volume header at LBA 7,814,035,454 → all zeros
- The boot region and its exFAT backup (volume sector 12) are both inside the dead 8 KB,
  so there is no header anywhere to restore — mounting is impossible, not merely difficult.

### Wrong turns made this session (recorded so they are not repeated)

1. "The SSD is failing" — wrong, it does 444 MB/s.
2. "It's BitLocker" — wrong; `EBD0A0A2-…` is **Microsoft Basic Data**, and Ammaniel has no
   Microsoft account. BitLocker is identified by a `-FVE-FS-` signature, not by a GPT GUID.
3. "It's HFS+" (from `ATTR` / `com.apple.provenance` / Finder-tag `bplist` found at 30 MB) —
   possible remnants, but never confirmed and never needed.
4. Telling him to pay a **$2,500 recovery lab** while the files were sitting there readable.
   That advice came from the bad speed reading in Step 2. **Get the `bs=1m` number first.**

**The rule that would have saved the whole afternoon:** confirm the drive's health with a
large-block read before forming any theory about what is wrong with it.

## 2026-10-19 — Netbrew "No such module 'Expo'" and the Xcode Cloud connect

**Symptoms:** Xcode showed `No such module 'Expo'` at `ios/Netbrew/AppDelegate.swift:1:17`, about 20 "module map file not found" errors and a "No Scheme" toolbar; the same project built fine from the command line.

**Causes, in order:**
1. Xcode had **`Netbrew.xcodeproj`** open instead of **`Netbrew.xcworkspace`** — the Pods live only in the workspace, so Expo's module map was invisible.
2. A second stale window held the same project: `Couldn't load 'Netbrew.xcodeproj' because it is already opened from another project or workspace`.
3. A corrupt per-project build folder `~/Library/Developer/Xcode/DerivedData/Netbrew-dhmcvqjzfywxcrgyrbdacgoskwwp` held only `SymbolCache`, and Xcode's saved window state reopened the broken layout on every launch.

**Fix that worked:** quit Xcode (`quit saving no`), delete **all** `~/Library/Developer/Xcode/DerivedData/Netbrew-*`, delete `~/Library/Saved Application State/com.apple.dt.Xcode.savedState`, reopen `ios/Netbrew.xcworkspace` only. One window, scheme **Netbrew**, destination iPhone 18 Pro → it built. Running `pod install --repo-update` in `ios/` beforehand repaired the Pods project itself.

**Xcode Cloud traps:**
- The Cloud product was bound to the wrong GitHub repo (`ammaniel-knowledge`) because `/Users/nakfaai/Developer/projects` is itself a git repo, so Xcode computed the path as `facelink/ios/Netbrew.xcodeproj`; build 1 failed with `Project Netbrew.xcodeproj does not exist at facelink/ios/Netbrew.xcodeproj`. Each app folder must be its own clone with its own `origin`.
- Integrate → Create Workflow on a stale product fails with "This workflow has no actions" and then "Failed to create workflow — An unexpected error occurred (XcodeCloudKit … CommandHandlerError:5)"; deleting the old product (`DELETE /v1/ciProducts/<id>` → 204) and starting fresh is faster than repairing it.
- The Cloud "Select Product" sheet lists **Pod frameworks first** (`AppAuth`, `EXConstants`, `Expo`…). The app target sits at the bottom of that list — picking a framework creates a workflow for the wrong thing.

**What I would do differently:**
- Never diagnose an Xcode build error from the GUI while a command-line build of the same scheme succeeds — the difference is window/workspace state, not code.
- Verify before saying "it works": I said "✅ Netbrew builds in Xcode now" without seeing the build finish, and Ammaniel called it out.
- A sandboxed shell cannot send Apple Events (`osascript … quit` → `-10004 privilege violation`) and cannot write under `~/.dsh` (`Operation not permitted`). Anything that quits or cleans Xcode goes to an **unsandboxed Bot** — netbrew quit Xcode, wiped the DerivedData and reopened the workspace in one run.

---
## Kalkidan → Xcode Cloud, 2026-10-20 (the long one)

**What was broken, in order — each one hid the next:**
1. **Wrong repo in the workflow.** The product `App` (`295e69a9-f37f-4921-ab30-0c6073da4671`) had its Primary Repository on `latindance.git` and container path `apps/capacitor/ios/App/App.xcworkspace`, so builds 1–26 compiled Latindance code while the window showed Kalkidan. Apple's API refuses the change (`PATCH /v1/ciWorkflows/{id}` → 409 `ENTITY_ERROR.RELATIONSHIP.NOT_ALLOWED`; creating products/workflows → 405/409). Only the pencil in Xcode can re-point it.
2. **Repo was gitignoring `ios/`.** `ci_post_clone` passed, then the build died on `Build input file cannot be found: /Volumes/workspace/repository/ios/App/App/Base.lproj/Main.storyboard`. Storyboards, asset catalogs and `GoogleService-Info.plist` were never committed. Fix: force-add them (`git add -f`) — 10 files, commit `e750782` → first green build 30.
3. **No `corepack` on Apple's image any more.** `brew install node` succeeded (Homebrew Node 26.12.0) and the next line failed: `corepack: command not found` (`ci_post_clone.sh: line 11`). Node 26 dropped corepack. Fix: `npm install --global pnpm@11.15.1` + `export PATH="$(npm prefix --global)/bin:$PATH"` → commit `64a5ecd`; the build got past setup and then hit trap 2.
4. **Build green but never distributable.** The workflow had only a **Build** action. Cloud's "You're ready to set up this product for distribution" prompt is the switch: Set Up → Archive + TestFlight/App Store. Build 31 is the first run after it.

**Rules learned:**
- An Xcode Cloud product named "App" is not evidence of anything — three Capacitor apps use that name. Identify a product by the **commit** its builds carry, not the name.
- Green build ≠ finished. `Build` alone leaves the artefact on Apple's machine; distribution needs the Archive action.
- A build number in a repo can ride along silently (a bot once pushed 16 → 17 inside a signing fix). Re-read `CURRENT_PROJECT_VERSION` after every bot commit.
- `corepack`, `nvm`, `gem` are traps on Apple's image; keep `ci_post_clone.sh` to PATH exports + a plain package manager install.

**What I would do differently:**
- Download the **LOG_BUNDLE** on the first red build, not the fourth. `ci_post_clone.log` names the failing line in one look; the Xcode report only says "script failed".
- Check `.gitignore` for the app folder *before* blaming CI: two builds were spent re-writing a script that was already correct.
- Say "the product in your window is Latindance's" the moment the commit shas do not match the app — I said it late and it cost hours of him clicking the wrong window.

