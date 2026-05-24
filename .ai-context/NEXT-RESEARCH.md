# NEXT-RESEARCH — agenda for next scheduled-research pass

> Written by `scheduled-research-daily` on 2026-05-24 (UTC). Tomorrow's run should start here, then overwrite this file with its own next-day agenda.

---

## ACTION REQUIRED items (carry forward until resolved)

These are upstream signals that imply a build-breaking or security change to the project. **The research agent never applies these.** They are surfaced here for human review and follow-up.

1. **ACTION REQUIRED (remediation lever MERGED, but image NOT yet rebuilt): kernel 5-CVE local-root cluster.** The MiOS base (`stable-nvidia-lts-20260511`) still predates every fix. **NEW 2026-05-24: `ublue-os/ucore` PR #392 MERGED today** (commit `412e7be`, approved @bsherman) — migrates the LTS image kernel flavor **longterm-6.12 → longterm-6.18 on the F44 base**. ucore main is active again. **BUT the fix only reaches MiOS once a rebuilt `ucore-hci:stable-nvidia-lts-YYYYMMDD` tag bakes on the merged base — UNCONFIRMED as of this pass.** Issue #385 (Copy Fail tracker) was NOT auto-closed. The rebuilt image must carry kernel **≥ 6.18.31** to close the whole cluster (≥6.18.23 DirtyDecrypt, ≥6.18.31 ssh-keysign-pwn); current longterm-6.18 = 6.18.33 (2026-05-23), so a fresh bake should satisfy. The cluster:
   - **`CVE-2026-31431` "Copy Fail"** — `algif_aead` AF_ALG root LPE, CVSS 7.8, **on CISA KEV (federal deadline 2026-05-15 passed), active exploitation.** Fixed 6.18.22 / 6.19.12 / 7.0. NVD reanalyzed 2026-05-21 (severity unchanged).
   - **`CVE-2026-31635` "DirtyDecrypt"** — RxGK LPE, CVSS 7.5, **only on `CONFIG_RXGK` kernels (Fedora) → applies to MiOS.** **Fixed 2026-04-25 upstream**, affected ranges 6.16.1–6.18.22 / 6.19.0–6.19.12. Public PoC 2026-05-18. **F44 `updates` ships `kernel-7.0.9-205.fc44` = fixed.**
   - **`CVE-2026-46333` "ssh-keysign-pwn"** — ptrace exit-race → LPE; steals SSH host keys + `/etc/shadow`. Fixed 7.0.8 / 6.18.31 / 6.12.89.
   - **`CVE-2026-43500` + `CVE-2026-43284` + `CVE-2026-46300` "Dirty Frag" (RHSB-2026-003)** — page-cache-write LPE on xfrm-ESP / RxRPC. Public PoC; affects Fedora via RxRPC. Fragnesia/-46300 = regression from the -43284 fix.
   **Defense-in-depth the owner may set while the rebuilt image is pending** (config the project *does* own — kargs/modprobe.d): `modprobe.blacklist=algif_aead algif_skcipher algif_hash algif_rng` (Copy Fail); `modprobe.blacklist=rxrpc` (DirtyDecrypt RxGK + Dirty-Frag RxRPC path, if AFS/rxrpc unused — almost certainly is); `kernel.dmesg_restrict=1`.

2. **ACTION REQUIRED: Base image on a deleted upstream repo.** `Containerfile` / `image-versions.yml` (line 12) still reference `ghcr.io/ublue-os/ucore-hci:stable-nvidia`; `bsherman/ucore-hci` is HTTP 404. Migration target remains `ghcr.io/ublue-os/ucore:stable-nvidia-lts`. **The PR #392 merge does NOT change the base-ref to adopt — only the kernel it carries.** Requires hand-editing `renovate.json` `customManagers` regex `depName` + `image-versions.yml` line 12 `depName`. Cannot be done by Renovate automerge.

3. **ACTION REQUIRED: Pin cosign ≥ 3.0.6** wherever the project verifies signatures. GHSAs (`GHSA-w6c6-c85g-mmv6` = CVE-2026-39395 CVSS 4.3, `GHSA-wfqv-66vq-46rm`, `GHSA-whqx-f9j3-ch6m`) fixed in 3.0.6 / 2.6.3; `CVE-2026-22703` (Rekor not bound to artifact) fixed in 2.6.2 / 3.0.4. cosign latest still **v3.0.6 (2026-04-06)** — no newer release, no new GHSA. Verify the baked cosign binary + `automation/42-cosign-policy.sh` + CI passes `--bundle` (v3 requires it where v2 made it optional).

4. **ACTION REQUIRED (RESOLVED-as-satisfied): NVIDIA kmod pin.** May 2026 bulletin `a_id/5821` floors satisfied by project pins (LTS 580.159.04 ≥ 580.159.03; feature 595.71.05 = R595 floor). New **dev-beta 595.44.09 (2026-05-22)** is NOT a project pin. No June bulletin. Only remaining action: confirm the rebuilt GHCR `ucore-hci` image bakes ≥ 580.159.03.

5. **ACTION REQUIRED (TIGHTEST HARD DEADLINE): Secure Boot 2023-CA shim refresh before 2026-06-26 (~33 days).** MS 2011 CA stops signing then. **F44 stable + updates + updates-testing ALL still ship only `shim-16.1-5` (2021-key) — `shim-16.1-6` (2023-key) is absent everywhere** (re-verified 2026-05-24 via mirrors.kernel.org `releases/` + `updates/` + `updates/testing/`). rhboot/shim upstream tip still `16.1` (2025-08-13) — the `-6` suffix is Fedora's package release. **Hard checkpoint: 2026-06-05** — if still 16.1-5 by then, MiOS needs a fallback (apply MS DBX update via `fwupdmgr` on target hardware).

6. **ACTION REQUIRED: Migrate from `ublue-os/bootc-image-builder-action` to `osbuild/bootc-image-builder-action`** if CI uses the former (verify in `.github/workflows/` — still not inspected; research-only scope). osbuild fork actively maintained; ublue-os fork maintenance-mode.

7. **ACTION REQUIRED: Fix `osautomation` → `osbuild` typo + zero digest in `image-versions.yml` line 21.** `osautomation` GitHub user has zero repos/packages; `image_builder_cli_digest` is all-zeros. Reference should be `ghcr.io/osbuild/image-builder-cli`. Trivial hand-fix.

8. **ACTION REQUIRED (actionable, positive): Bump the K3s pin to a 2026-05-20 GA tag.** K3s GA shipped 2026-05-20 (v1.36.1+k3s1 / v1.35.5+k3s1 / v1.34.8+k3s1 / v1.33.12+k3s1). These remediate **CVE-2026-33186** (grpc-go authz bypass, CVSS 9.1) via a `replace google.golang.org/grpc => v1.79.3` directive in go.mod (release notes are silent — the binary is patched regardless). Pick the GA tag matching the project's minor line.

9. **ACTION REQUIRED (validate before pinning): fapolicyd v1.5 (2026-05-20).** First minor bump off 1.4.x (transactional rule reload, `--check-rules`, per-rule hit counters). Since fapolicyd is a deny-by-default execution gate, **validate the trust DB / rules still load identically at image-build time before pinning v1.5.**

10. **ACTION REQUIRED (F45-paced, ~Oct 2026): Podman 6.0 GA slipped to Fedora 45.** No 6.0 RC tag cut (latest stable v5.8.2; Quadlet schema-delta docs still placeholder). Pre-flight Quadlet review still required (BoltDB→SQLite, slirp4netns→Pasta, cgroups v1 removal, netavark iptables→nftables).

11. **ACTION REQUIRED (migration possible, partial parity): `bootc-image-builder` → `image-builder-cli`.** v65 (2026-05-21) GA bootc subcommand, but public docs only enumerate qcow2 + bootc-installer ISO. BIB still has the wider matrix and remains active (no deprecation timeline). Viable for qcow2/ISO only until full parity (issue #506 = upstream tracker, still open).

12. **OPERATIONAL RISK (de-escalated, keep watching): git-receive-pack HTTP 503.** The 2026-05-19 and 2026-05-20 passes hit a persistent 503 on the proxy's git-receive-pack endpoint. **2026-05-24 update:** the 503-suffixed 2026-05-22 commits were in fact present on `origin/main` (the local tracking ref was just stale — a `git fetch` revealed them); the MCP per-file pushes had landed. So the remote was consistent. **Always `git fetch` before assuming the remote is behind.** Keep the per-file-MCP / single-commit-batching fallback in mind if a real 503 recurs.

---

## Priority topics for tomorrow's pass

Ordered by descending value. Rationale under each.

### P0 — Re-verify all ACTION REQUIRED items

Touch each upstream link to see if anything shifted in 24h. **Tightest: (a) Secure Boot shim-16.1-6 in F44 (~33 days to cutover, checkpoint 2026-06-05); (b) the rebuilt `ucore-hci` bake on the merged PR #392 base — the gate for the kernel-CVE fix reaching MiOS.** If any resolve, strike them and note resolution in `ai-journal.md`.

- shim-16.1-6 in F44 (Anubis-free): https://mirrors.kernel.org/fedora/releases/44/Everything/x86_64/os/Packages/s/ (+ `updates/44/...` + `updates/testing/44/...`)
- ucore-hci GHCR tags: https://github.com/ublue-os/ucore/pkgs/container/ucore-hci — **TIGHT WATCH for a new `stable-nvidia-lts-YYYYMMDD`**
- `ublue-os/ucore` main: https://github.com/ublue-os/ucore/commits/main
- issue #385: https://github.com/ublue-os/ucore/issues/385 (still open post-merge)
- cosign: https://github.com/sigstore/cosign/releases
- NVIDIA drivers: https://github.com/NVIDIA/open-gpu-kernel-modules/releases

### P1 — Secure Boot shim-16.1-6 in F44 (TIGHTEST hard-calendar deadline)

*Why:* 2026-06-26 cutover is ~33 days; hard checkpoint 2026-06-05. Verified-absent across all three F44 trees today.

*Specific questions:* fetch the three mirrors.kernel.org paths; grep `shim-`; has 16.1-6 landed in stable or updates-testing? Does Fedora's multi-signed shim auto-roll on bootc upgrade or require explicit `fwupdmgr`? Flag escalation if still 16.1-5 by 2026-06-05.

### P2 — Rebuilt `ucore-hci` bake on the merged PR #392 base + its kernel NVR

*Why:* PR #392 merged 2026-05-24 (longterm-6.12 → longterm-6.18 on F44). **The only remaining gate** for the 5-CVE kernel cluster fix reaching MiOS is a fresh `ucore-hci:stable-nvidia-lts` bake on the merged base. Issue #385 still open.

*Specific questions:* Has a new `stable-nvidia-lts-YYYYMMDD` tag appeared since `-20260511`? If yes, `skopeo inspect` / oci-inspect it — what kernel NVR does it bake? **Must be ≥ 6.18.31** to close DirtyDecrypt (≥6.18.23) + ssh-keysign-pwn (≥6.18.31). Did issue #385 close once a fixed image shipped? Any new ucore main commits? Has the KVMFR build-patch concern (§9.3, `MODULE_IMPORT_NS` string-literal + `vmalloc.h`) surfaced in the F44/6.18 build logs?

*Anchors:* ucore-hci GHCR tags, ucore main, issue #385 (above); https://github.com/ublue-os/ucore/pull/392 (merged).

### P3 — GNOME 50.2 ship (was tarballs-due 2026-05-23)

*Why:* Release expected ~week of 2026-05-25/26; `download.gnome.org/core/50/50.2/` 404'd today. Confirm it shipped and read 50.2 NEWS for any Mutter / gnome-remote-desktop / NVIDIA explicit-sync backport (expected bugfix-only — confirm).

*Anchor:* https://release.gnome.org/calendar/, https://download.gnome.org/core/50/50.2/, GNOME 50.2 NEWS once tagged.

### P4 — Pacemaker 3.0.2 final

*Why:* rc2 was 2026-05-11; now ~13 days. Projected final ~2026-05-28 — within the window.

*Anchor:* https://github.com/ClusterLabs/pacemaker/releases.

### P5 — composefs v1.1 tag + bootc native-backend GA

*Why:* main active (last commit PR #436, 2026-05-19) but still no tag in 16.5 months; bootc still flags the native composefs backend verbatim "Experimental … not yet suitable for production use." A v1.1 cut + bootc dropping the experimental framing is a significant on-disk-format event and gates the F45 sealed-image direction.

*Anchors:* https://github.com/composefs/composefs/releases, https://github.com/composefs/composefs/commits/main, https://bootc.dev/bootc/experimental-composefs.html.

### P6 — image-builder-cli parity + issue #506

*Why:* v65 (2026-05-21) added no new formats. Check v66+, BIB deprecation timeline, issue #506 (composefs+UKI sealed-image backend). osbuild.org/blog still 404 — retry.

*Anchors:* https://github.com/osbuild/image-builder-cli/releases, https://github.com/osbuild/image-builder-cli/issues/506, https://github.com/osbuild/bootc-image-builder/issues.

### P7 — Podman 6.0 RC tag watch (F45-paced)

*Why:* GA slip relieves pressure, but an RC tag will drop and Quadlet schema deltas remain undocumented. When an RC lands, the project gets concrete diff signal.

*Anchors:* https://github.com/containers/podman/releases, https://fedoraproject.org/wiki/Changes/Podman6.

### P8 — Looking Glass (master idle ~4 months) + Gamescope HDR (#2018 open)

*Why:* LG master has zero commits since 2026-01-17; Gamescope HDR fix `7d4e835` still master-only, #2018 still open with no maintainer ack. Both stalled-upstream watches — low churn, monthly cadence. **LG matters more now** — once the rebuilt ucore-hci 6.18 base lands, the KVMFR build patch (§9.3) becomes mandatory and rides on LG's single 2025-03-04 `module/` commit.

*Anchors:* https://github.com/gnif/LookingGlass/commits/master, https://github.com/ValveSoftware/gamescope/issues/2018, https://github.com/ValveSoftware/gamescope/tags.

### P9 — Routine version watches (low priority)

Mesa 26.0.8 / 26.1.2; libvirt 12.4.0 release; QEMU 11.0.x / 11.1; NVIDIA Container Toolkit post-1.19.1; WSL post-2.7.7 / 2.8.x; etcd 3.7.0 GA progress; Ceph 20.2.2 / Squid bulletins; CrowdSec post-1.7.8; fapolicyd post-1.5; systemd v261-rc progression (stable 260.x unaffected); Renovate cadence; bootc post-1.15.2; OSTree post-2026.1; ROCm post-7.2.3; Waydroid post-1.6.2; FreeIPA/SSSD.

---

## Upstream releases + CVE feeds to monitor

| Source | What to check |
| ------ | ------------- |
| https://mirrors.kernel.org/fedora/releases/44/Everything/x86_64/os/Packages/s/ | **shim-16.1-6 F44 promotion (Anubis-free) — TIGHTEST DEADLINE** (+ updates / updates/testing) |
| https://github.com/ublue-os/ucore/pkgs/container/ucore-hci | **new `stable-nvidia-lts` bake on merged PR #392 base — TIGHT WATCH** |
| https://github.com/ublue-os/ucore/commits/main | upstream activity (active again post-#392) |
| https://github.com/ublue-os/ucore/issues/385 | Copy Fail tracker (still open post-merge) |
| https://github.com/ublue-os/ucore/pull/392 | merged 2026-05-24 (longterm-6.18 / F44) |
| https://www.kernel.org/ | stable/longterm (7.0.10 / 6.18.33 / 6.12.91 / 6.6.141 / 6.1.174 as of 2026-05-23) |
| https://access.redhat.com/security/vulnerabilities/RHSB-2026-003 | Dirty Frag (CVE-2026-43284/-43500/-46300) |
| https://access.redhat.com/security/cve/cve-2026-31431 | Copy Fail — CISA KEV |
| https://github.com/sigstore/cosign/releases | post-3.0.6 |
| https://github.com/NVIDIA/open-gpu-kernel-modules/releases | post-580.159.04 / 595.71.05 / 595.44.09 |
| https://github.com/k3s-io/k3s/releases | post-GA (v1.36.1 / v1.35.5 / v1.34.8 shipped 2026-05-20) |
| https://github.com/ClusterLabs/pacemaker/releases | **3.0.2 final — TIGHT WATCH** (rc2 ~13 days old) |
| https://github.com/linux-application-whitelisting/fapolicyd/releases | post-1.5 |
| https://github.com/containers/podman/releases | v6.0 RC tag (none yet) |
| https://github.com/osbuild/image-builder-cli/releases | post-v65 |
| https://github.com/osbuild/image-builder-cli/issues/506 | composefs+UKI sealed-image backend |
| https://github.com/composefs/composefs/commits/main | v1.1 tag |
| https://github.com/bootc-dev/bootc/releases | post-v1.15.2 |
| https://github.com/ostreedev/ostree/releases | post-v2026.1 |
| https://github.com/etcd-io/etcd/releases | 3.7.0 GA progress (beta.0 = 2026-05-19) |
| https://ceph.io/en/news/blog/ | Tentacle patch / Squid bulletins |
| https://github.com/crowdsecurity/crowdsec/releases | post-1.7.8 |
| https://release.gnome.org/calendar/ | **GNOME 50.2 (due ~week of 05-25)**, 51.alpha 2026-06-27 |
| https://github.com/microsoft/WSL/releases | post-2.7.7 / 2.8.x |
| https://github.com/systemd/systemd/releases | v261-rc progression (stable 260.x) |
| https://github.com/renovatebot/renovate/releases | post-43.195.0 |
| https://docs.mesa3d.org/relnotes.html | post-26.1.1 / 26.0.7 |
| https://libvirt.org/news.html | 12.4.0 release |
| https://www.qemu.org/blog/ | post-11.0.0 |
| https://github.com/NVIDIA/nvidia-container-toolkit/releases | post-1.19.1 |
| https://github.com/gnif/LookingGlass/commits/master | B8 / cadence resume (idle ~4 months) |
| https://github.com/ValveSoftware/gamescope/tags | 3.16.24 / 3.17 + HDR fix 7d4e835 |

---

## Follow-up questions raised (resolved + unresolved)

**Resolved this pass:**
- **"Which F44 kernel NVR first carries the DirtyDecrypt fix?"** — F44 `updates` ships `kernel-7.0.9-205.fc44` (past the 6.16.1–6.18.22 / 6.19.0–6.19.12 affected range). Exact first-fixed NVR not pinpointed (bodhi Anubis-gated), but any 7.0.x F44 build is safe.
- **"Does PR #392 land before the 2026-06-05 checkpoint?"** — YES, merged 2026-05-24. Remaining gate is the `ucore-hci` rebuild (new P2 watch).

**New this pass:**
1. **Which exact F44 longterm-6.18 kernel NVR will the rebuilt `ucore-hci:stable-nvidia-lts` image carry?** Must be ≥ 6.18.31 to close the full cluster. Verify once the bake appears.
2. **Does the rebuilt 6.18 base break the KVMFR build** without the LookingGlass `MODULE_IMPORT_NS` string-literal patch (§9.3)? `automation/52-bake-kvmfr.sh` must apply it before the bake succeeds — code-inspection follow-up.
3. **Should the owner set `modprobe.blacklist=rxrpc` (+ `algif_aead` etc.)** as kargs/modprobe.d defense-in-depth while the rebuilt image is pending? Config the project owns. Project-owner decision.

**Carried forward (still unresolved — out of research-only scope):**
- Workflows still calling `ublue-os/bootc-image-builder-action`? (migrate to osbuild fork)
- cosign binary pinned to a digest in `automation/42-cosign-policy.sh`? (≥ 3.0.6)
- SELinux site modules landing in `/etc/selinux/targeted/active/modules/400/` (persists) vs `/usr/lib/selinux/` (wiped)?
- K3s HA mode vs single-node sqlite? (affects etcd-migration urgency)
- `automation/52-bake-kvmfr.sh` signs at image-build time (not first-boot)? **Now more urgent — the LTS base is moving to 6.18 and needs the kernel ≥6.13 KVMFR patch.**
- fapolicyd trust DB rebuilt at image-build (`fapolicyd-cli --update`) or via dnf plugin? Does v1.5's transactional rule-reload change the bake?
- GNOME 50 in `ucore-hci:stable-nvidia` yet?
- AMD iGPU used at all? (AMDGPU CVE cluster hits the 9950X3D iGPU unless `amdgpu` blacklisted)
- `mios-sysext-pack.sh` consume systemd 260's `/etc/systemd/systemd-sysext.conf`?
- Which NVIDIA driver line does the project pin? (595.71.x prod-feature vs 595.44.x dev-beta vs 580.x LTS — confirm rebuilt GHCR image bakes ≥ 580.159.03)
- Mesa line in the running image (25.x vs 26.x)?

---

## Priority-order rationale

P0 (reverify) first, as always. **P1 (Secure Boot) stays top** — it is the tightest *hard-calendar* deadline (~33 days, checkpoint 2026-06-05) and was verified-absent across all three F44 trees today. **P2 changed shape:** PR #392 is now MERGED (the long-running merge watch is closed), so the watch shifts to the **downstream `ucore-hci` rebuild + its kernel NVR** — the actual gate for the kernel-CVE fix reaching MiOS, and now human-trackable via GHCR tags. **P3 (GNOME 50.2)** is a near-term confirmation (release imminent). **P4 (Pacemaker 3.0.2 final)** is the closest "expected within days" event (~2026-05-28). The DirtyDecrypt-NVR and PR-#392-merge watches are **resolved** and drop off the funnel. **P5–P9** are slower monthly/routine checks; LG (P8) gains weight because the 6.18 base bump makes its KVMFR patch mandatory.

Anything not on this list can be skipped tomorrow unless an upstream release explicitly demands inclusion. **Tomorrow's run should overwrite this file with its own next-day agenda.**
