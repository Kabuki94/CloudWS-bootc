# NEXT-RESEARCH — agenda for next scheduled-research pass

> Written by `scheduled-research-daily` on 2026-05-26 (UTC). Tomorrow's run should start here, then overwrite this file with its own next-day agenda.

> **Repo-layout reminder:** this repo has NO `CLAUDE.md`, NO `docs/changelogs/`, NO top-level `CHANGELOG.md`. The SSOT is `INDEX.md` (+ `ENGINEERING.md`, `.cursorrules`, `.clinerules`). Hard invariants: USR-OVER-ETC, NO-MKDIR-IN-VAR, UNPRIVILEGED-QUADLETS (`User=`/`Group=`/`Delegate=yes`), BOOTC-NATIVE (`bootc container lint` passes), bash `set -euo pipefail`, kargs.d flat-array, NVIDIA VM-gating. Research-only scope: touch only `.ai-context/`.

---

## ACTION REQUIRED items (carry forward until resolved)

These are upstream signals that imply a build-breaking or security change to the project. **The research agent never applies these.** They are surfaced here for human review and follow-up.

1. **ACTION REQUIRED (remediation now DELIVERABLE — project-side adoption pending): kernel local-root CVE cluster.** **NEW 2026-05-26: the gate is CLOSED.** A fresh **`ucore-hci:stable-nvidia-lts-20260526`** bake exists (built 2026-05-26T04:19Z; manifest-probe `-20260526`/`-20260525` = HTTP 200, `-20260527` = 404) on the **post-#392-merge base** (image revision `e63e21f` = ucore commit #397, 2026-05-25; FCOS `44.20260419.3.1` stable; longterm-6.18 flavor = kernel **6.18.33** ≥ all cluster floors). **But MiOS pins the *non-LTS* rolling `ucore-hci:stable-nvidia` by digest, so the fix is not yet delivered.** The project owner must either (a) let Renovate bump the `:stable-nvidia` digest pin to a freshly-rebuilt image (verify the rolling tag is also re-baking), or (b) migrate the base-ref to `ghcr.io/ublue-os/ucore:stable-nvidia-lts` (see item 2). Issue #385 (Copy Fail tracker) is **still open** despite the bake. The cluster: **`CVE-2026-31431` "Copy Fail"** (algif_aead AF_ALG LPE, CVSS 7.8, on CISA KEV — federal deadline 2026-05-15 passed; fixed 6.18.22/6.19.12/7.0); **`CVE-2026-31635` "DirtyDecrypt"** (RxGK, CVSS 7.5, CONFIG_RXGK=Fedora; fixed 2026-04-25, ranges 6.16.1–6.18.22 / 6.19.0–6.19.12); **`CVE-2026-46333` "ssh-keysign-pwn"** (ptrace exit-race → LPE; fixed 7.0.8/6.18.31/6.12.89); **`CVE-2026-43500`+`CVE-2026-43284`+`CVE-2026-46300` "Dirty Frag" (RHSB-2026-003)** — RxRPC/ESP page-cache-write LPE, still "Ongoing/Important." Defense-in-depth the owner *could* set while adopting the bake (config the project owns — kargs/modprobe.d): `modprobe.blacklist=algif_aead algif_skcipher algif_hash algif_rng` (Copy Fail); `modprobe.blacklist=rxrpc` (DirtyDecrypt RxGK + Dirty-Frag RxRPC path, if AFS/rxrpc unused — almost certainly is); `kernel.dmesg_restrict=1`.

2. **ACTION REQUIRED: Base image on a deleted upstream repo + base-ref migration.** `Containerfile` / `image-versions.yml` (line 12) still reference `ghcr.io/ublue-os/ucore-hci:stable-nvidia`; `bsherman/ucore-hci` is HTTP 404. Migration target remains `ghcr.io/ublue-os/ucore:stable-nvidia-lts`. The PR #392 merge + the `-20260526` bake do NOT change the base-ref to adopt — only the kernel it carries. Requires hand-editing `renovate.json` `customManagers` regex `depName` + `image-versions.yml` line 12 `depName`. Cannot be done by Renovate automerge. **Adopting `:stable-nvidia-lts` is also the simplest way to pick up the kernel-CVE fix (item 1).**

3. **ACTION REQUIRED: Pin cosign ≥ 3.0.6** wherever the project verifies signatures. GHSAs (`GHSA-w6c6-c85g-mmv6` = CVE-2026-39395 CVSS 4.3, `GHSA-wfqv-66vq-46rm`, `GHSA-whqx-f9j3-ch6m`) fixed in 3.0.6 / 2.6.3; `CVE-2026-22703` (Rekor not bound to artifact) fixed in 2.6.2 / 3.0.4. cosign latest still **v3.0.6 (2026-04-06)** — no newer release, no new GHSA (re-verified 2026-05-26). Verify the baked cosign binary + `automation/42-cosign-policy.sh` + CI passes `--bundle` (v3 requires it where v2 made it optional). **Note:** ucore commit #397 (the base the `-20260526` bake was built on) switched ucore's own CI to "cosign login instead of docker login" — unrelated to MiOS's verification path but a sign cosign tooling is in active use upstream.

4. **ACTION REQUIRED (TIGHTEST HARD DEADLINE): Secure Boot 2023-CA shim refresh before 2026-06-26 (~31 days).** MS 2011 CA stops signing then. **F44 stable + updates + updates-testing ALL still ship only `shim-16.1-5` (2021-key) — `shim-16.1-6` (2023-key) is absent everywhere** (re-verified 2026-05-26 via mirrors.kernel.org `releases/` + `updates/` + `updates/testing/`). rhboot/shim upstream tip still `16.1` (2025-08-13) — the `-6` suffix is Fedora's package release. **Hard checkpoint: 2026-06-05 (~10 days)** — if still 16.1-5 by then, MiOS needs a fallback (apply MS DBX update via `fwupdmgr` on target hardware). No movement in two passes.

5. **ACTION REQUIRED (RESOLVED-as-satisfied): NVIDIA kmod pin.** May 2026 bulletin `a_id/5821` floors satisfied by project pins (LTS 580.159.04 ≥ 580.159.03; feature 595.71.05 = R595 floor). Re-verified 2026-05-26: no new driver tag, no June bulletin. dev-beta 595.44.09 (2026-05-22) is NOT a project pin. Only remaining action: confirm the fresh `ucore-hci:stable-nvidia-lts-20260526` bake carries NVIDIA NVR ≥ 580.159.03 when the project adopts it.

6. **ACTION REQUIRED: Migrate from `ublue-os/bootc-image-builder-action` to `osbuild/bootc-image-builder-action`** if CI uses the former (verify in `.github/workflows/` — still not inspected; research-only scope). osbuild fork actively maintained; ublue-os fork maintenance-mode.

7. **ACTION REQUIRED: Fix `osautomation` → `osbuild` typo + zero digest in `image-versions.yml` line 21.** `osautomation` GitHub user has zero repos/packages; `image_builder_cli_digest` is all-zeros. Reference should be `ghcr.io/osbuild/image-builder-cli`. Trivial hand-fix.

8. **ACTION REQUIRED (actionable, positive): Bump the K3s pin to a 2026-05-20 GA tag.** K3s GA shipped 2026-05-20 (v1.36.1+k3s1 / v1.35.5+k3s1 / v1.34.8+k3s1 / v1.33.12+k3s1). These remediate **CVE-2026-33186** (grpc-go authz bypass, CVSS 9.1) via a `replace google.golang.org/grpc => v1.79.3` directive in go.mod (release notes are silent — the binary is patched regardless). Pick the GA tag matching the project's minor line.

9. **ACTION REQUIRED (validate before pinning): fapolicyd v1.5 (2026-05-20).** First minor bump off 1.4.x (transactional rule reload, `--check-rules`, per-rule hit counters). Since fapolicyd is a deny-by-default execution gate, **validate the trust DB / rules still load identically at image-build time before pinning v1.5.** No v1.5.x yet.

10. **ACTION REQUIRED (F45-paced, ~Oct 2026): Podman 6.0 GA slipped to Fedora 45.** No 6.0 RC tag cut (latest stable v5.8.2; Quadlet schema-delta docs still placeholder). Pre-flight Quadlet review still required (BoltDB→SQLite, slirp4netns→Pasta, cgroups v1 removal, netavark iptables→nftables).

11. **ACTION REQUIRED (migration possible, partial parity): `bootc-image-builder` → `image-builder-cli`.** v66 (2026-05-25) added profiling + erofs/squashfs mount stages + skopeo container resolution — but **no bootc/composefs/UKI/sealed-image work**; public docs still enumerate only qcow2 + bootc-installer ISO. BIB still has the wider matrix and remains active (no deprecation timeline). Viable for qcow2/ISO only until full parity (issue #506 = upstream tracker, still open, no movement since 2026-04-29).

12. **OPERATIONAL RISK (de-escalated, keep watching): git-receive-pack HTTP 503 + stale tracking ref.** For the third consecutive pass, the local `origin/main` tracking ref was stale (showed an older commit) while the remote actually had the latest MCP-pushed commits. **Always `git fetch` before assuming the remote is behind.** Keep the per-file-MCP / single-commit-batching fallback in mind if a real 503 recurs on `git push`.

---

## Priority topics for tomorrow's pass

Ordered by descending value. Rationale under each.

### P0 — Re-verify all ACTION REQUIRED items

Touch each upstream link to see if anything shifted in 24h. **Tightest: (a) Secure Boot shim-16.1-6 in F44 (~31 days to cutover, checkpoint 2026-06-05 = ~10 days); (b) project-side adoption of the `-lts-20260526` bake — has the rolling `:stable-nvidia` tag re-baked, is there an open Renovate digest-bump PR, did issue #385 close?** If any resolve, strike them and note resolution in `ai-journal.md`.

- shim-16.1-6 in F44 (Anubis-free): https://mirrors.kernel.org/fedora/releases/44/Everything/x86_64/os/Packages/s/ (+ `updates/44/...` + `updates/testing/44/...`)
- ucore-hci GHCR tags (manifest-probe `-YYYYMMDD`; `tags/list` pagination is broken): https://github.com/ublue-os/ucore/pkgs/container/ucore-hci — **probe for `stable-nvidia-lts-20260527`+ and check the rolling `:stable-nvidia` digest**
- `ublue-os/ucore` main: https://github.com/ublue-os/ucore/commits/main
- issue #385: https://github.com/ublue-os/ucore/issues/385 (still open post-bake)
- cosign: https://github.com/sigstore/cosign/releases
- NVIDIA drivers: https://github.com/NVIDIA/open-gpu-kernel-modules/releases

### P1 — Secure Boot shim-16.1-6 in F44 (TIGHTEST hard-calendar deadline)

*Why:* 2026-06-26 cutover is ~31 days; hard checkpoint 2026-06-05 is **~10 days** out. Verified-absent across all three F44 trees for two passes running.

*Specific questions:* fetch the three mirrors.kernel.org paths; grep `shim-`; has 16.1-6 landed in stable or updates-testing? Does Fedora's multi-signed shim auto-roll on bootc upgrade or require explicit `fwupdmgr`? **Flag escalation if still 16.1-5 by 2026-06-05.**

### P2 — Project-side adoption of the `-lts-20260526` bake

*Why:* The upstream gate (a fresh ucore-hci bake on the merged base) closed 2026-05-26. The remaining gate is **project-side**: MiOS pins the non-LTS rolling `:stable-nvidia`, so the kernel-CVE fix only lands via a Renovate digest bump or the `:stable-nvidia-lts` base-ref migration.

*Specific questions:* Is the rolling `ucore-hci:stable-nvidia` tag also being re-baked daily again (probe its digest vs the one in `image-versions.yml`)? Has Renovate opened a digest-bump PR (cannot inspect cross-repo via MCP — research-only)? Did issue #385 close once the bake was verified? Probe for `stable-nvidia-lts-20260527`+ to confirm the daily cadence is sustained, not a one-off.

*Anchors:* ucore-hci GHCR tags, ucore main, issue #385 (above).

### P3 — GNOME 50.2 ship (overdue — tarball deadline 2026-05-23 passed)

*Why:* Release expected within days; `download.gnome.org/core/50/50.2/` still 404'd today and the releng path shows 50.1 latest. Confirm it shipped and read 50.2 NEWS for any Mutter / gnome-remote-desktop / NVIDIA explicit-sync / HDR backport (expected bugfix-only — confirm).

*Anchor:* https://download.gnome.org/teams/releng/, https://download.gnome.org/core/50/50.2/, https://release.gnome.org/calendar/, GNOME 50.2 NEWS once tagged.

### P4 — Pacemaker 3.0.2 final

*Why:* rc2 was 2026-05-11; now ~15 days. Projected final ~2026-05-28 — within the window.

*Anchor:* https://github.com/ClusterLabs/pacemaker/releases.

### P5 — QEMU 11.0.1 "QEMUtiny" CVE-ID + any 11.0.2/11.1

*Why:* 11.0.1 shipped 2026-05-25 (+10.2.3/10.0.10 backports) with no CVE-ID in tag metadata and no blog post; the CXL "QEMUtiny" framing came from oss-sec. Low MiOS exposure (CXL Type-3 emulation not in a GPU-passthrough topology), but confirm the CVE assignment and that no MiOS libvirt domain uses CXL.

*Anchors:* https://www.qemu.org/blog/, https://gitlab.com/qemu-project/qemu/-/tags, seclists.org oss-sec QEMUtiny thread.

### P6 — composefs v1.1 tag + bootc native-backend GA

*Why:* main active (last commit PR #436, 2026-05-19, hardlinked-whiteout fix) but still no tag in 16.5 months; bootc still flags the native composefs backend verbatim "Experimental … not yet suitable for production use." A v1.1 cut + bootc dropping the experimental framing is a significant on-disk-format event and gates the F45 sealed-image direction.

*Anchors:* https://github.com/composefs/composefs/releases, https://github.com/composefs/composefs/commits/main, https://bootc.dev/bootc/experimental-composefs.html.

### P7 — image-builder-cli parity + issue #506

*Why:* v66 (2026-05-25) added no new image formats and no bootc/composefs work. Check v67+, BIB deprecation timeline, issue #506 (composefs+UKI sealed-image backend). osbuild.org/blog still 404 — retry.

*Anchors:* https://github.com/osbuild/image-builder-cli/releases, https://github.com/osbuild/image-builder-cli/issues/506, https://github.com/osbuild/bootc-image-builder/issues.

### P8 — Podman 6.0 RC tag watch (F45-paced) + Looking Glass / Gamescope HDR

*Why:* Podman GA slip relieves pressure but an RC tag will eventually drop with Quadlet schema deltas. **LG matters more now** — once MiOS adopts the longterm-6.18 base, the KVMFR build patch (§9.3) becomes mandatory and rides on LG's single 2025-03-04 `module/` commit (master idle ~4 months). Gamescope HDR fix `7d4e835` still master-only; #2018 still open, no maintainer ack.

*Anchors:* https://github.com/containers/podman/releases, https://github.com/gnif/LookingGlass/commits/master/module, https://github.com/ValveSoftware/gamescope/tags, https://github.com/ValveSoftware/gamescope/issues/2018.

### P9 — Routine version watches (low priority)

Mesa 26.0.8 / 26.1.2; libvirt 12.4.0 release; ROCm post-7.2.3; NVIDIA Container Toolkit post-1.19.1; WSL post-2.7.7 / 2.8.x (GitHub releases page returned a stale snapshot — re-verify); etcd 3.7.0 GA progress (beta.0 = 2026-05-19); Ceph 20.2.2 / Squid bulletins (re-fetch ceph.io blog — last pass lower-confidence); CrowdSec post-1.7.8; fapolicyd post-1.5; systemd v261-rc progression (stable 260.x unaffected); Renovate cadence; bootc post-1.15.2; OSTree post-2026.1; Waydroid post-1.6.2; FreeIPA/SSSD; K3s post-GA patch.

---

## Upstream releases + CVE feeds to monitor

| Source | What to check |
| ------ | ------------- |
| https://mirrors.kernel.org/fedora/releases/44/Everything/x86_64/os/Packages/s/ | **shim-16.1-6 F44 promotion (Anubis-free) — TIGHTEST DEADLINE** (+ updates / updates/testing) |
| https://github.com/ublue-os/ucore/pkgs/container/ucore-hci | **rolling `:stable-nvidia` re-bake + new `-lts-YYYYMMDD` tags** (manifest-probe; `tags/list` pagination broken) |
| https://github.com/ublue-os/ucore/commits/main | upstream activity (active again — #397 on 2026-05-25) |
| https://github.com/ublue-os/ucore/issues/385 | Copy Fail tracker (still open post-bake) |
| https://www.kernel.org/ | stable/longterm (7.0.10 / 6.18.33 / 6.12.91 / 6.6.141 / 6.1.174; mainline 7.1-rc5 as of 2026-05-24) |
| https://access.redhat.com/security/vulnerabilities/RHSB-2026-003 | Dirty Frag (CVE-2026-43284/-43500/-46300) — still "Ongoing/Important" |
| https://access.redhat.com/security/cve/cve-2026-31431 | Copy Fail — CISA KEV |
| https://github.com/sigstore/cosign/releases | post-3.0.6 |
| https://github.com/NVIDIA/open-gpu-kernel-modules/releases | post-580.159.04 / 595.71.05 / 595.44.09; June bulletin watch |
| https://github.com/k3s-io/k3s/releases | post-GA patch (v1.36.1 / v1.35.5 / v1.34.8 / v1.33.12 shipped 2026-05-20) |
| https://github.com/ClusterLabs/pacemaker/releases | **3.0.2 final — TIGHT WATCH** (rc2 ~15 days old) |
| https://gitlab.com/qemu-project/qemu/-/tags | **QEMU 11.0.1 (2026-05-25) — QEMUtiny CVE-ID watch**; post-11.0.1 / 11.1 |
| https://www.qemu.org/blog/ | 11.0.1 announcement (not yet posted) |
| https://github.com/linux-application-whitelisting/fapolicyd/releases | post-1.5 |
| https://github.com/containers/podman/releases | v6.0 RC tag (none yet) |
| https://github.com/osbuild/image-builder-cli/releases | post-v66 |
| https://github.com/osbuild/image-builder-cli/issues/506 | composefs+UKI sealed-image backend |
| https://github.com/composefs/composefs/commits/main | v1.1 tag |
| https://github.com/bootc-dev/bootc/releases | post-v1.15.2 |
| https://github.com/ostreedev/ostree/releases | post-v2026.1 |
| https://github.com/etcd-io/etcd/releases | 3.7.0 GA progress (beta.0 = 2026-05-19) |
| https://ceph.io/en/news/blog/ | Tentacle patch / Squid bulletins (re-fetch — lower confidence last pass) |
| https://github.com/crowdsecurity/crowdsec/releases | post-1.7.8 |
| https://release.gnome.org/calendar/ | **GNOME 50.2 (overdue)**, 50.3 + 51.alpha both 2026-06-27 |
| https://github.com/microsoft/WSL/releases | post-2.7.7 / 2.8.x (page returned stale snapshot — re-verify) |
| https://github.com/systemd/systemd/releases | v261-rc progression (stable 260.x) |
| https://github.com/renovatebot/renovate/releases | post-43.195.8 |
| https://docs.mesa3d.org/relnotes.html | post-26.1.1 / 26.0.7 |
| https://libvirt.org/news.html | 12.4.0 release |
| https://github.com/NVIDIA/nvidia-container-toolkit/releases | post-1.19.1 |
| https://github.com/gnif/LookingGlass/commits/master/module | KVMFR `module/` commit (idle ~4 months) / B8 |
| https://github.com/ValveSoftware/gamescope/tags | post-3.16.24 / 3.17 + HDR fix 7d4e835 |

---

## Follow-up questions raised (resolved + unresolved)

**Resolved this pass:**
- **"Has a new `ucore-hci:stable-nvidia-lts` bake appeared on the merged PR #392 base?"** — YES: `-20260526`, built 2026-05-26T04:19Z on image revision `e63e21f` (ucore #397, 2026-05-25). The P2 gate is closed.
- **"Which kernel NVR — must be ≥6.18.31?"** — the longterm-6.18 flavor = 6.18.33, past all cluster floors. No explicit NVR label; confirmed by the longterm line.

**New this pass:**
1. **Is the rolling `ucore-hci:stable-nvidia` tag (the one MiOS actually pins) also freshly re-baked**, and is there an open Renovate digest-bump PR for `image-versions.yml`? This is the actual delivery path for the kernel-CVE fix.
2. **QEMU 11.0.1 "QEMUtiny" CVE-ID assignment** — confirm the CVE number and that no MiOS libvirt domain emulates CXL Type-3 devices (it shouldn't).
3. **Does issue #385 close now that a fixed bake exists?** Maintainers gate-close on verification, not on existence.

**Carried forward (still unresolved — out of research-only scope):**
- Workflows still calling `ublue-os/bootc-image-builder-action`? (migrate to osbuild fork)
- cosign binary pinned to a digest in `automation/42-cosign-policy.sh`? (≥ 3.0.6)
- SELinux site modules landing in `/etc/selinux/targeted/active/modules/400/` (persists) vs `/usr/lib/selinux/` (wiped)?
- K3s HA mode vs single-node sqlite? (affects etcd-migration urgency)
- `automation/52-bake-kvmfr.sh` applies the LG `MODULE_IMPORT_NS` string-literal patch + signs at image-build time (not first-boot)? **Now imminent — adopting the longterm-6.18 LTS base needs the kernel ≥6.13 KVMFR patch, and LG's `module/` has had no new commits in ~4 months.**
- fapolicyd trust DB rebuilt at image-build (`fapolicyd-cli --update`) or via dnf plugin? Does v1.5's transactional rule-reload change the bake?
- GNOME 50 in `ucore-hci:stable-nvidia` yet?
- AMD iGPU used at all? (AMDGPU CVE cluster hits the 9950X3D iGPU unless `amdgpu` blacklisted)
- `mios-sysext-pack.sh` consume systemd 260's `/etc/systemd/systemd-sysext.conf`?
- Which NVIDIA driver line does the project pin? (595.71.x prod-feature vs 595.44.x dev-beta vs 580.x LTS — confirm the `-20260526` bake carries ≥ 580.159.03)
- Mesa line in the running image (25.x vs 26.x)?

---

## Priority-order rationale

P0 (reverify) first, as always. **P1 (Secure Boot) stays top** — it is the tightest *hard-calendar* deadline (~31 days, checkpoint 2026-06-05 = ~10 days) and has been verified-absent across all three F44 trees for two passes running with zero movement. **P2 changed shape again:** the upstream bake gate is now CLOSED (the long-running ucore-hci bake watch is resolved), so the watch shifts to **project-side adoption** — the rolling `:stable-nvidia` digest, a Renovate bump, and the issue #385 close. **P3 (GNOME 50.2)** is overdue and imminent. **P4 (Pacemaker 3.0.2 final)** is the closest "expected within days" event (~2026-05-28). **P5 (QEMU QEMUtiny CVE-ID)** is a near-term confirmation on a fresh point release. **P6–P9** are slower monthly/routine checks; LG (P8) gains weight because the project will need its KVMFR patch the moment it adopts the 6.18 base.

Anything not on this list can be skipped tomorrow unless an upstream release explicitly demands inclusion. **Tomorrow's run should overwrite this file with its own next-day agenda.**
