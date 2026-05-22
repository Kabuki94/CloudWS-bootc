# NEXT-RESEARCH — agenda for next scheduled-research pass

> Written by `scheduled-research-daily` on 2026-05-22 (UTC). Tomorrow's run should start here, then overwrite this file with its own next-day agenda.

---

## ACTION REQUIRED items (carry forward until resolved)

These are upstream signals that imply a build-breaking or security change to the project. **The research agent never applies these.** They are surfaced here for human review and follow-up.

1. **ACTION REQUIRED (now a FIVE-CVE local-root cluster, base image still unpatched): kernel LPE/escape exposure.** The MiOS base (`stable-nvidia-lts-20260511`) predates every fix below, and the only remediation lever — `ublue-os/ucore` PR #392 (F43→F44 base) — has been static since 2026-05-17 (open, no reviews). ucore main idle 15 days; GHCR bake stalled 11 days. The cluster as of 2026-05-22:
   - **`CVE-2026-31431` "Copy Fail"** — `algif_aead` AF_ALG root LPE, CVSS 7.8, **on CISA KEV (federal deadline 2026-05-15 passed), active exploitation confirmed.** Fixed 6.18.22 / 6.19.12 / 7.0. Red Hat RHSB-2026-002.
   - **`CVE-2026-31635` "DirtyDecrypt"** — RxGK (AFS-rxrpc) LPE, CVSS 7.5, **only on `CONFIG_RXGK` kernels (Fedora) → applies to MiOS.** Public PoC 2026-05-18. Fix in stable trees 7.0.9 / 6.18.32 / 6.12.90.
   - **`CVE-2026-46333` "ssh-keysign-pwn"** — ptrace exit-race info-disclosure → LPE; steals SSH host keys + `/etc/shadow`. Fixed 7.0.8 / 6.18.31 / 6.12.89 / 6.6.139 / 6.1.173.
   - **`CVE-2026-43500` + `CVE-2026-43284` + `CVE-2026-46300` "Dirty Frag" (Red Hat RHSB-2026-003)** — deterministic page-cache-write LPE via `splice` / `MSG_SPLICE_PAGES` on the xfrm-ESP / RxRPC paths. Public PoC; affects Fedora via RxRPC. (CVE-2026-43500 is NEW this window.)
   **Defense-in-depth the owner may consider pre-PR-392-merge:** `modprobe.blacklist=algif_aead algif_skcipher algif_hash algif_rng` (neutralizes Copy Fail AF_ALG); `modprobe.blacklist=rxrpc` (removes DirtyDecrypt RxGK + the RxRPC Dirty-Frag path, if AFS/rxrpc is unused — almost certainly is); `kernel.dmesg_restrict=1`. None of these are base-kernel config the project owns; they are kargs/modprobe.d the project *can* set.

2. **ACTION REQUIRED: Base image on a deleted upstream repo + cadence stalled 11 days + upstream main idle 15 days.** `Containerfile` still references `ghcr.io/ublue-os/ucore-hci:stable-nvidia` (`image-versions.yml` line 12 pins it). `bsherman/ucore-hci` is HTTP 404. `ublue-os/ucore-hci` GHCR latest is still `stable-nvidia-lts-20260511` (11 days old). `ublue-os/ucore` main: zero commits since 2026-05-07 (15 days idle). Migration target remains `ghcr.io/ublue-os/ucore:stable-nvidia-lts`. Requires hand-editing `renovate.json` `customManagers` regex `depName` + `image-versions.yml` line 12 `depName`. **Cannot be done by Renovate automerge.**

3. **ACTION REQUIRED: Pin cosign ≥ 3.0.6** wherever the project verifies signatures. GHSAs `GHSA-w6c6-c85g-mmv6` (= CVE-2026-39395, CVSS 4.3), `GHSA-wfqv-66vq-46rm`, `GHSA-whqx-f9j3-ch6m` fixed in 3.0.6 / 2.6.3; `CVE-2026-22703` (Rekor not bound to artifact) fixed in 2.6.2 / 3.0.4. Verify the cosign binary baked in + the `automation/42-cosign-policy.sh` flow; confirm CI passes `--bundle` (v3 requires it where v2 made it optional). cosign latest still v3.0.6 (2026-04-06) — no newer release.

4. **ACTION REQUIRED (RESOLVED-as-satisfied — pins already cover the May bulletin): NVIDIA kmod pin.** The **NVIDIA May 2026 bulletin `a_id/5821` is published** (2026-05-19; 13 CVEs, top `CVE-2026-24187` CVSS 8.8 Linux UAF). Linux floors: R595 = 595.71.05, R580 RTX/Quadro/NVS = 580.159.03, R580 GeForce = 580.126.09, R535 = 535.309.01. **Project pins (LTS 580.159.04 ≥ 580.159.03; feature 595.71.05 = R595 floor) already satisfy — no bump forced.** Only remaining action: confirm the GHCR `ucore-hci` image actually bakes ≥ 580.159.03 (lts-20260511 ≈ 580.159.04, so it should). Jan 2026 advisory a_id/5747 still the older floor.

5. **ACTION REQUIRED (TIGHTEST HARD DEADLINE): Secure Boot 2023-CA shim refresh before 2026-06-26 (~35 days).** MS 2011 CA stops signing on that date. **F44 stable + updates + updates-testing ALL still ship only `shim-16.1-5` (2021-key) — `shim-16.1-6` (2023-key) is absent everywhere in F44** (verified 2026-05-22 via mirrors.kernel.org `releases/` + `updates/` + `updates/testing/` — the Anubis-free check path). `shim-16.1-6` remains rawhide-only. **Hard checkpoint: 2026-06-05** — if 16.1-6 still hasn't landed in F44 by then, MiOS needs a fallback. Apply Microsoft DBX update via `fwupdmgr` on target hardware.

6. **ACTION REQUIRED: Migrate from `ublue-os/bootc-image-builder-action` to `osbuild/bootc-image-builder-action`** if CI uses the former (verify in `.github/workflows/` — still not inspected; research-only scope). osbuild fork is actively maintained; ublue-os fork is maintenance-mode.

7. **ACTION REQUIRED: Fix `osautomation` → `osbuild` typo + zero digest in `image-versions.yml` line 21.** Confirmed: `osautomation` GitHub user has zero repos/packages; the `image_builder_cli_digest` is all-zeros. Reference should be `ghcr.io/osbuild/image-builder-cli`. Trivial hand-fix.

8. **ACTION REQUIRED (NEW — now actionable, positive): Bump the K3s pin to a 2026-05-20 GA tag.** K3s GA shipped 2026-05-20 (v1.36.1+k3s1 / v1.35.5+k3s1 / v1.34.8+k3s1). These remediate **CVE-2026-33186** (grpc-go authz bypass, CVSS 9.1) via a `replace google.golang.org/grpc => v1.79.3` directive in go.mod (release notes are silent — the fix is in the binary regardless). Pick the GA tag matching the project's minor line.

9. **ACTION REQUIRED (NEW — validate before pinning): fapolicyd v1.5 (2026-05-20)** is the first minor bump off 1.4.x (transactional rule reload, `--check-rules`, per-rule hit counters). Since fapolicyd is a deny-by-default execution gate, **validate that the trust DB / rules still load identically at image-build time before pinning v1.5** — a reload-behavior change could break the bake.

10. **ACTION REQUIRED (F45-paced, ~5 months): Podman 6.0 GA slipped to Fedora 45.** No 6.0 RC tag cut (latest stable v5.8.2). Pre-flight Quadlet review still required (BoltDB→SQLite, slirp4netns→Pasta, cgroups v1 removal, netavark iptables→nftables) but F45-paced (Oct 2026).

11. **ACTION REQUIRED (migration now possible, partial parity): `bootc-image-builder` → `image-builder-cli`.** image-builder-cli v65 (2026-05-21) GA bootc subcommand, but public docs only enumerate qcow2 + bootc-installer ISO — raw/ami/vmdk/vhd/gce undocumented. BIB still has the wider matrix. Viable for qcow2/ISO only until full parity (issue #506 = upstream tracker, still open).

12. **OPERATIONAL RISK (meta — flag for the human owner): recurring git-receive-pack HTTP 503.** The 2026-05-19 and 2026-05-20 passes both hit a persistent 503 on the proxy's git-receive-pack endpoint; the 2026-05-20 pass **lost its journal + knowledge-doc edits** (only the NEXT-RESEARCH commit landed). If this push path keeps 503-ing, the scheduled job needs a more robust push/fallback strategy (e.g. per-file MCP `create_or_update_file` as the 2026-05-19 pass used, or batching all three files into one commit so a partial-push can't split the trio).

---

## Priority topics for tomorrow's pass

Ordered by descending value. Rationale under each.

### P0 — Re-verify all ACTION REQUIRED items

Touch each upstream link to see if anything shifted in 24h. **Tightest: (a) Secure Boot shim-16.1-6 in F44 (~35 days to cutover, checkpoint 2026-06-05); (b) PR #392 merge status — gates the only kernel-CVE remediation path for a now-5-CVE local-root cluster.** Re-check ucore main / `ucore-hci` bake cadence — any resumption unblocks actions #1 and #2.

- `ublue-os/ucore` PR #392: https://github.com/ublue-os/ucore/pull/392 — **TIGHT WATCH**
- `ublue-os/ucore` issue #385: https://github.com/ublue-os/ucore/issues/385
- `ublue-os/ucore` main (idle 15 days): https://github.com/ublue-os/ucore/commits/main
- ucore-hci GHCR tags: https://github.com/ublue-os/ucore/pkgs/container/ucore-hci
- shim-16.1-6 in F44 (Anubis-free): https://mirrors.kernel.org/fedora/releases/44/Everything/x86_64/os/Packages/s/ (+ `https://mirrors.kernel.org/fedora/updates/44/Everything/x86_64/Packages/s/` + `.../updates/testing/44/...`)
- cosign: https://github.com/sigstore/cosign/releases
- NVIDIA drivers: https://github.com/NVIDIA/open-gpu-kernel-modules/releases

If any resolve, **strike them from the ACTION REQUIRED list** and note resolution in `ai-journal.md`.

### P1 — Secure Boot shim-16.1-6 in F44 (TIGHTEST hard-calendar deadline)

*Why:* 2026-06-26 cutover is ~35 days; hard checkpoint 2026-06-05. Verified-absent today across all three F44 trees.

*Specific questions:* fetch the three mirrors.kernel.org paths above; grep `shim-`; has 16.1-6 landed in stable or updates-testing? Does Fedora's multi-signed shim auto-roll on bootc upgrade or require explicit `fwupdmgr`? Flag escalation if still 16.1-5 by 2026-06-05.

### P2 — PR #392 merge status + kernel package version in any new ucore-hci tag

*Why:* Still the only visible remediation path for the 5-CVE kernel local-root cluster (Copy Fail + DirtyDecrypt + ssh-keysign-pwn + Dirty Frag). PR static since 2026-05-17; main idle 15 days.

*Specific questions:* Has PR #392 merged / issue #385 closed? If merged, has a new `stable-nvidia-lts-YYYYMMDD` tag landed? Skopeo/oci-inspect it: what kernel NVR does it bake? (Must be ≥ 6.18.22 for Copy Fail; ≥ 6.18.31 for ssh-keysign-pwn; DirtyDecrypt + Dirty Frag present in any kernel built after their ~early-May merges.) Any new ucore main commits?

*Anchors:* PR #392, ucore-hci GHCR tags, issue #385 (above).

### P3 — GNOME 50.2 ship (was due 2026-05-23)

*Why:* Scheduled for 2026-05-23 — should have shipped by tomorrow's pass. Confirm it shipped and whether it carries any Wayland / NVIDIA explicit-sync / HDR fix relevant to MiOS GRD/Gamescope (the major explicit-sync/HDR work landed in 50.0, so 50.2 is likely bugfix-only — confirm).

*Anchor:* https://release.gnome.org/calendar/, GNOME 50.2 NEWS once tagged.

### P4 — Pacemaker 3.0.2 final

*Why:* rc2 was 2026-05-11; now 11 days. rc1→rc2 gap was ~17 days; projected final ~2026-05-28 — within the next-few-days window.

*Anchor:* https://github.com/ClusterLabs/pacemaker/releases.

### P5 — DirtyDecrypt (CVE-2026-31635) Fedora kernel NVR

*Why:* `CONFIG_RXGK` makes this Fedora-specific; need to know which exact F44 kernel build first carries the fix to confirm the eventual F44-base bump actually closes it. bodhi is Anubis-gated — try mirrors.kernel.org kernel changelog, kernel.org, or a non-gated Fedora kernel-NVR source.

*Anchors:* https://www.kernel.org/, https://mirrors.kernel.org/fedora/updates/44/Everything/x86_64/Packages/k/ (kernel NVRs), https://thehackernews.com/2026/05/dirtydecrypt-poc-released-for-linux.html.

### P6 — composefs v1.1 tag + bootc native-backend GA

*Why:* main active but still no tag in 16.5 months; bootc still flags the native composefs backend "experimental." A v1.1 cut + bootc dropping the experimental framing is a significant on-disk-format event and gates the F45 sealed-image direction.

*Anchors:* https://github.com/composefs/composefs/releases, https://github.com/composefs/composefs/commits/main, https://bootc.dev/bootc/experimental-composefs.html.

### P7 — image-builder-cli parity + issue #506

*Why:* v65 (2026-05-21) added no new formats. Check v66+, BIB deprecation timeline, issue #506 (composefs+UKI sealed-image backend). osbuild.org/blog still 404 — retry.

*Anchors:* https://github.com/osbuild/image-builder-cli/releases, https://github.com/osbuild/image-builder-cli/issues/506, https://github.com/osbuild/bootc-image-builder/issues.

### P8 — Podman 6.0 RC tag watch (F45-paced)

*Why:* GA slip relieves pressure, but an RC tag will drop and Quadlet schema deltas remain undocumented. When an RC lands, the project gets concrete diff signal.

*Anchors:* https://github.com/containers/podman/releases, https://fedoraproject.org/wiki/Changes/Podman6.

### P9 — Looking Glass (master idle ~4 months) + Gamescope HDR (#2018 open)

*Why:* LG master has zero commits since 2026-01-17; Gamescope HDR fix `7d4e835` still master-only, #2018 still open with no maintainer ack. Both are stalled-upstream watches — low churn, monthly cadence.

*Anchors:* https://github.com/gnif/LookingGlass/commits/master, https://github.com/ValveSoftware/gamescope/issues/2018, https://github.com/ValveSoftware/gamescope/tags.

### P10 — Routine version watches (low priority)

Mesa 26.0.8 / 26.1.2; libvirt 12.4.0 release; QEMU 11.0.x; NVIDIA Container Toolkit post-1.19.1; WSL post-2.7.7 / 2.8.x; etcd 3.7.0 GA progress; Ceph 20.2.2 / Squid bulletins; CrowdSec post-1.7.8; systemd post-260.1; Renovate cadence; bootc post-1.15.2; OSTree post-2026.1.

---

## Upstream releases + CVE feeds to monitor

| Source | What to check |
| ------ | ------------- |
| https://github.com/ublue-os/ucore/pull/392 | **PR #392 merge status — TIGHT WATCH** |
| https://github.com/ublue-os/ucore/commits/main | upstream activity (idle 15 days) |
| https://github.com/ublue-os/ucore/pkgs/container/ucore-hci | daily-bake cadence (11 days stale) — TIGHT WATCH |
| https://mirrors.kernel.org/fedora/releases/44/Everything/x86_64/os/Packages/s/ | **shim-16.1-6 F44 promotion (Anubis-free) — TIGHTEST DEADLINE** (+ updates / updates/testing) |
| https://www.kernel.org/ | stable/longterm (7.0.9 / 6.18.32 / 6.12.90 / 6.6.140 / 6.1.173 as of 2026-05-17) |
| https://access.redhat.com/security/vulnerabilities/RHSB-2026-003 | **Dirty Frag (CVE-2026-43284/-43500/-46300)** |
| https://thehackernews.com/2026/05/dirtydecrypt-poc-released-for-linux.html | **DirtyDecrypt CVE-2026-31635** Fedora NVR |
| https://access.redhat.com/security/cve/cve-2026-31431 | Copy Fail — CISA KEV |
| https://github.com/sigstore/cosign/releases | post-3.0.6 |
| https://nvidia.custhelp.com/app/answers/detail/a_id/5821 | May 2026 bulletin (published; 403/Anubis — use GamingOnLinux mirror) |
| https://github.com/NVIDIA/open-gpu-kernel-modules/releases | post-580.159.04 / 595.71.05 / 595.44.08 |
| https://github.com/k3s-io/k3s/releases | post-GA (v1.36.1 / v1.35.5 / v1.34.8 shipped 2026-05-20) |
| https://github.com/ClusterLabs/pacemaker/releases | **3.0.2 final — TIGHT WATCH** (rc2 11 days old) |
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
| https://release.gnome.org/calendar/ | **GNOME 50.2 (due 2026-05-23)**, 51.alpha 2026-06-27 |
| https://github.com/microsoft/WSL/releases | post-2.7.7 / 2.8.x |
| https://github.com/systemd/systemd/releases | post-260.1 |
| https://github.com/renovatebot/renovate/releases | post-43.192.0 |
| https://docs.mesa3d.org/relnotes.html | post-26.1.1 / 26.0.7 |
| https://libvirt.org/news.html | 12.4.0 release |
| https://www.qemu.org/blog/ | post-11.0.0 |
| https://github.com/NVIDIA/nvidia-container-toolkit/releases | post-1.19.1 |
| https://github.com/gnif/LookingGlass/commits/master | B8 / cadence resume (idle ~4 months) |
| https://github.com/ValveSoftware/gamescope/tags | 3.16.24 / 3.17 + HDR fix 7d4e835 |

---

## Follow-up questions raised (resolved + unresolved)

**Resolved this pass:**
- **CVE-2026-33186 grpc-go callout** — RESOLVED: K3s 2026-05-20 GA links grpc-go v1.79.3 via a `replace` directive (notes silent). Owner should bump the K3s pin to a GA tag (action item #8).
- **NVIDIA May 2026 bulletin (cadence-due)** — RESOLVED: a_id/5821 published; project pins satisfy all floors.

**New this pass:**
1. **Which Fedora F44 kernel NVR first carries the DirtyDecrypt (`CVE-2026-31635`) fix?** bodhi Anubis-gated — find via mirrors.kernel.org kernel changelog or kernel.org.
2. **Does fapolicyd v1.5's transactional-rule-reload change affect the image-build-time trust-DB bake?** Validate before pinning.
3. **Should the owner set `modprobe.blacklist=rxrpc` (+ `algif_aead` etc.) as kargs/modprobe.d defense-in-depth** while PR #392 is stalled? This is config the project *does* own (unlike the base kernel). Project-owner decision.

**Carried forward (still unresolved — out of research-only scope):**
- Workflows still calling `ublue-os/bootc-image-builder-action`? (migrate to osbuild fork)
- cosign binary pinned to a digest in `automation/42-cosign-policy.sh`? (≥ 3.0.6)
- SELinux site modules landing in `/etc/selinux/targeted/active/modules/400/` (persists) vs `/usr/lib/selinux/` (wiped)?
- K3s HA mode vs single-node sqlite? (affects etcd-migration urgency)
- `automation/52-bake-kvmfr.sh` signs at image-build time (not first-boot)? Once base bumps to 6.18, must apply the kernel ≥6.13 KVMFR patch (`MODULE_IMPORT_NS` string-literal form + `vmalloc.h`).
- fapolicyd trust DB rebuilt at image-build (`fapolicyd-cli --update`) or via dnf plugin?
- GNOME 50 in `ucore-hci:stable-nvidia` yet?
- AMD iGPU used at all? (AMDGPU CVE cluster hits the 9950X3D iGPU unless `amdgpu` blacklisted)
- `mios-sysext-pack.sh` consume systemd 260's `/etc/systemd/systemd-sysext.conf`?
- Which NVIDIA driver line does the project pin? (595.71.x prod-feature vs 595.44.x dev-beta vs 580.x LTS — confirm GHCR image bakes ≥ 580.159.03)
- Mesa line in the running image (25.x vs 26.x)?

---

## Priority-order rationale

P0 (reverify) first, as always. **P1 (Secure Boot) is now promoted above PR #392** because it is the tightest *hard-calendar* deadline (~35 days, checkpoint 2026-06-05) and was verified-absent across all three F44 trees today — the clock is real and the project must act before the base bump may even be relevant. **P2 (PR #392)** remains the only lever for the 5-CVE kernel cluster but is human-uncontrollable and has been static for 5 days, so re-checking is cheap but unlikely to move. **P3 (GNOME 50.2)** is a near-term confirmation (was due 2026-05-23). **P4 (Pacemaker 3.0.2 final)** is the closest "expected within days" event. **P5 (DirtyDecrypt NVR)** sharpens whether the eventual F44 bump closes the cluster. The K3s-CVE and NVIDIA-bulletin watches are **resolved** and drop off the funnel (now owner-action items #8 and #4). **P6–P10** are slower monthly/routine checks.

Anything not on this list can be skipped tomorrow unless an upstream release explicitly demands inclusion. **Tomorrow's run should overwrite this file with its own next-day agenda.**
