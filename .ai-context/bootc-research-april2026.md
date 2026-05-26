# CloudWS-bootc / MiOS — Upstream Research (live document)

> **Status:** Bootstrapped 2026-05-11 by `scheduled-research-daily`. Live document — refined daily. Latest pass 2026-05-26.
> **Project version pinned at:** v0.1.4 (`VERSION` file).
> **Base image:** `ghcr.io/ublue-os/ucore-hci:stable-nvidia` (digest-pinned via Renovate in `image-versions.yml`).
> **Scope:** This doc tracks upstream state and project-relevant deltas. Refine in place; mark revisions inline with `(updated YYYY-MM-DD: <reason>)`. Add new findings as dated subsections under the relevant topic. Remove proven-false entries and note removal in the journal.

---

## Top-priority action items (current — 2026-05-26)

These are flagged in `NEXT-RESEARCH.md` as `ACTION REQUIRED`. They are surfaced here for visibility but **never applied by the research agent itself.**

1. **Base image is on an archived/deleted repo + cadence is now 8 days stale + upstream main idle 12 days.** `bsherman/ucore-hci` returns HTTP 404 (updated 2026-05-16: was "archived" — now deleted or made private). Dev consolidated under `ublue-os/ucore`; `ghcr.io/ublue-os/ucore-hci:stable-nvidia-lts` container is actively rebuilt by that org. New canonical tag for the NVIDIA-LTS profile MiOS targets remains `ghcr.io/ublue-os/ucore:stable-nvidia-lts` (pre-signed `kmod-nvidia-open` 580 LTS). Renovate cannot self-correct a depName change — must be edited by hand. **Watch `ublue-os/ucore` issue #385 and PR #392** — kernel bump pending to address kernel Copy Fail CVE (see item 8). (updated 2026-05-19: **daily-rebuild cadence still stalled** — latest tag still `stable-nvidia-lts-20260511` (digest `sha256:a764379b…`, 8 days old). `ublue-os/ucore` main has had **zero commits since 2026-05-07** (12 days idle). **NEW THIS PASS:** PR #392 "fix: resolve build failures for Fedora 44" opened 2026-05-17T15:57Z by dylanmtaylor — flips ucore's `FEDORA_VERSION` 43 → 44 plus mergerfs / install-ucore tweaks. **This is the implicit kernel-bump path** (F44 base brings a newer kernel package set). PR still open, unmerged, unreviewed as of 2026-05-19. Issue #385 itself was updated 2026-05-17T12:54Z and now has 7 comments (bodies not retrievable via WebFetch — JS-rendered).) (updated 2026-05-22: **still zero forward motion on every lever.** PR #392 unchanged — still open, no reviews/approvals, last activity the two 2026-05-17 force-pushes. Issue #385 still open. `ublue-os/ucore` main **now idle 15 days** (last commit `Fix 404 link to cosign README (#382)`, 2026-05-07). `ucore-hci` GHCR daily-bake **now stalled 11 days** — newest tag still `stable-nvidia-lts-20260511`. The base image remains pre-fix for the entire kernel local-root cluster, which has **grown** this window — see item 8.) (updated 2026-05-24: **MAJOR — PR #392 MERGED 2026-05-24.** Approved by @bsherman, all 73 checks passed (commit `412e7be`); `ublue-os/ucore` **main is active again** (mergerfs v2.42.0 bump #390 on 2026-05-23, then the #392 merge today — the 15-day idle gap is broken). The PR migrates the **LTS kernel flavor from longterm-6.12 → longterm-6.18 on the F44 base** + mergerfs F44 RPMs + nfs-utils build fix. **This is the kernel-CVE remediation lever finally moving at the source.** Caveats: (a) **issue #385 was NOT auto-closed** — still open; (b) **no new `ucore-hci` GHCR bake confirmed yet** (GHCR tag enumeration unavailable via WebFetch — manual `skopeo`/registry check needed); (c) longterm-6.18 is itself inside the DirtyDecrypt affected range (6.16.1–6.18.22), so the new image must land **≥ 6.18.23 (DirtyDecrypt) and ≥ 6.18.31 (ssh-keysign-pwn)** to close the cluster — current longterm-6.18 is **6.18.33** (2026-05-23), so a fresh bake should satisfy. The migration target `ghcr.io/ublue-os/ucore:stable-nvidia-lts` and the Renovate `depName` hand-fix are unchanged — the merge does not change the base-ref the project must adopt, only the kernel it carries.) (updated 2026-05-26: **GATE CLOSED — a fresh `ucore-hci:stable-nvidia-lts-20260526` bake now exists** (registry-manifest probe: `-20260526` and `-20260525` return HTTP 200; `-20260527` 404 — so `-20260526` is newest, built **2026-05-26T04:19Z**). Its image config `org.opencontainers.image.revision = e63e21f` = ucore commit **#397 "use cosign login instead of docker login" (2026-05-25)** — i.e. built **on the post-#392-merge base**. FCOS base `version=44.20260419.3.1`, stream `stable`. The 11-to-15-day daily-bake stall is broken. The longterm-6.18 LTS flavor tracks kernel.org 6.18.x = **6.18.33** (≥ the 6.18.31 ssh-keysign-pwn floor), so the bake closes the kernel local-root cluster (see item 8 / §6.5). **What remains for the project (still ACTION REQUIRED, not auto-applied):** the project pins the *non-LTS* rolling `ucore-hci:stable-nvidia` by digest, so remediation reaches MiOS only when (a) Renovate bumps that digest pin to a freshly-rebuilt `:stable-nvidia` (verify the rolling tag is also re-baking — only the `-lts` dated tags were probed this pass), or (b) the project migrates the base-ref to `ghcr.io/ublue-os/ucore:stable-nvidia-lts` per the standing recommendation. Issue #385 is **still open** despite the bake.)
2. **cosign verification bypass — patch immediately.** `CVE-2026-22703` (Rekor entry not bound to artifact) fixed in **cosign 2.6.2 / 3.0.4**. Multiple cosign GHSAs (`GHSA-w6c6-c85g-mmv6`, `GHSA-wfqv-66vq-46rm`, `GHSA-whqx-f9j3-ch6m`) all fixed in **cosign ≥ 3.0.6 / 2.6.3**. Any signature-verification gate using older cosign is bypassable. (updated 2026-05-16: removed mis-attribution of `CVE-2026-31431` to cosign — that CVE is the Linux kernel "Copy Fail" LPE, see item 7.) (updated 2026-05-19: `GHSA-w6c6-c85g-mmv6` now has a CVE assignment — **CVE-2026-39395** (CVSS 4.3 Moderate, `verify-blob-attestation` false positive); already fixed in v3.0.6, no remediation change. No new GHSAs; latest cosign release unchanged at v3.0.6 (2026-04-06).)
3. **NVIDIA driver pin — bump LTS floor.** Pin `kmod-nvidia-open` ≥ **580.159.04** (LTS — 2026-05-14) or ≥ **595.71.05** (feature — 2026-04-28). (updated 2026-05-16: LTS floor lifted from 580.126.20 → 580.159.04; new branch release post the Jan 2026 advisory. The Jan 2026 NVIDIA bulletin advisory `a_id/5747` covers CVEs `CVE-2025-33219`, `CVE-2025-23277`, `CVE-2025-23280` — kernel-module LPE/RCE — and is fully patched in 580.126.09+.) (updated 2026-05-22 — **DOWNGRADED, pins already satisfy**: the **NVIDIA May 2026 bulletin (`a_id/5821`) is now published** (2026-05-19, updated 2026-05-21) — 13 CVEs (12 Linux), top **CVE-2026-24187** CVSS **8.8** Linux UAF. Linux fix floors: R595 = 595.71.05, R580 RTX/Quadro/NVS = 580.159.03, R580 GeForce = 580.126.09, R535 = 535.309.01 (R570 listed EOL/no-fix by GamingOnLinux — not a project pin, immaterial). **The project's pins already satisfy these** — LTS 580.159.04 ≥ 580.159.03, feature 595.71.05 = R595 floor. **No bump forced.** Action remaining: confirm the GHCR `ucore-hci` image actually bakes ≥ 580.159.03. See §8.1.) (updated 2026-05-24: new **dev-beta 595.44.09 (2026-05-22)** supersedes 595.44.08 — Vulkan/bugfix pre-release, **not** a project pin. LTS 580.159.04 / production-feature 595.71.05 both unchanged; pins still satisfy all May-bulletin floors. No June bulletin exists.) (updated 2026-05-26: re-verified — **no new driver tag** (LTS 580.159.04 / feature 595.71.05 / dev-beta 595.44.09 all unchanged) and **still no June 2026 bulletin** (latest remains May a_id/5821). Pins satisfy all floors; no action. The fresh `ucore-hci:stable-nvidia-lts-20260526` bake (item 1) should carry ≥580.159.x — confirm the baked NVIDIA NVR when the project adopts it.)
4. **Secure Boot 2011-CA expiry — 2026-06-26 (~5 weeks out, URGENT).** Microsoft stops signing with the 2011 CA in late June 2026. (updated 2026-05-16: **Fedora 44 final still ships `shim-16.1-5` which is 2021-key signed only** — Rawhide has `shim-16.1-6` with 2023-key signing but it has not landed in F44 stable yet. Track `bodhi.fedoraproject.org` for shim-16.1-6+ updates-testing → stable promotion.) MiOS image must pick up Fedora's **2023-CA-signed shim** before that date or new installs onto firmware that has updated `db` will fail. Already-running systems continue to boot. (updated 2026-05-22: **still `shim-16.1-5` in F44 — verified directly via the Anubis-free mirrors.kernel.org tree.** `curl` of `releases/44/.../os/Packages/s/`, `updates/44/...`, and `updates/testing/44/...` shows `shim-x64-16.1-5` / `shim-ia32-16.1-5` in releases and **no shim package at all in updates or updates-testing** — i.e. 16.1-6 has not landed anywhere in F44. Cutover now **~35 days out**. mirrors.kernel.org is the working check path; bodhi/koji/dl.fedoraproject.org all remain Anubis-gated. Hard checkpoint 2026-06-05.) (updated 2026-05-24: **re-verified — still absent.** mirrors.kernel.org F44 `releases/` still has only `shim-x64/ia32-16.1-5` (+ `shim-unsigned-x64/ia32-16.1-1`); `updates/` and `updates/testing/` still carry **no shim package at all**. rhboot/shim upstream tip still `16.1` (2025-08-13). Cutover now **~33 days** out; hard checkpoint 2026-06-05 unchanged.) (updated 2026-05-26: **re-verified — still absent, no movement.** mirrors.kernel.org F44 `releases/.../os/Packages/s/` still has only `shim-x64-16.1-5` / `shim-ia32-16.1-5` (+ `shim-unsigned-x64/ia32-16.1-1`); `updates/` and `updates/testing/` still carry **no shim package at all**. rhboot/shim upstream tip still tag `16.1` (2025-08-13). Cutover now **~31 days** out; hard checkpoint 2026-06-05 now **~10 days** out — this is the tightest unresolved risk and has not budged in the 2-day window.)
5. **RTX 50-series VFIO passthrough is broken.** Reset bug acknowledged by NVIDIA, no fix shipped (verified 2026-05-16; 595.71.05 LTS / 595.x feature did NOT ship a fix; no 600-series driver yet; kernel 6.18 ships no Blackwell-specific FLR/IOMMU 1:1 patches). RTX 4090 (project's target) is unaffected, but any roadmap upgrade should be deferred.
6. **`image-builder-cli` v64 (2026-05-13) — `bootc` subcommand now GA** (updated 2026-05-16). PR #510 dropped "bootc is experimental". Canonical container is `ghcr.io/osbuild/image-builder-cli:latest`. **Confirms `ghcr.io/osautomation/...` reference in `image-versions.yml` is a typo for `osbuild`** — the `osautomation` GitHub user exists with zero public repos / no GHCR packages. **MiOS can now plan migration from `bootc-image-builder` to `image-builder-cli` as the unified tool** — but BIB remains a separate active repo, so this is "now possible" not "forced". (updated 2026-05-22: **v65 shipped 2026-05-21** — deps bump to `images` 0.266.0 + cache-dir handling for non-root users; no new format coverage. Issue #506 (composefs+UKI sealed-image bootc backend) still open, no movement since 2026-04-29. `osbuild.org/blog/` still 404. See §1.3.)
7. **Podman 6.0 GA — TARGET SLIPPED to Fedora 45** (updated 2026-05-18: was "imminent — week of 2026-05-25"). The Fedora Change Proposal at `fedoraproject.org/wiki/Changes/Podman6` is now tagged `ChangeAcceptedF45` (last edited 2026-03-11) — original F44 plan was abandoned before this research started. **No 6.0 RC tag has been cut as of 2026-05-18** — upstream latest is still `v5.8.2` (2026-04-14). Test Days closed 2026-05-15 but no post-test-day report has been published. **Net effect:** the project has more breathing room than the May-25 deadline implied; the Quadlet pre-flight review is still needed but is now F45-paced (Oct 2026). Breaking removals (unchanged): **BoltDB** (→ SQLite), **slirp4netns** (→ Pasta), **cgroups v1** entirely. netavark switches **iptables → nftables** default. Quadlet 6.0 adds `.artifact` file type, `AppArmor=` key, `HttpProxy=`, `StopTimeout=`, multi-doc `.kube`, templated vol/net deps. **Pre-flight review still required before bumping Podman version label** — any Quadlet relying on slirp4netns/cgroup-v1/BoltDB on-disk state will break.
8. **Linux kernel CVE cluster — CVE-2026-31431 IS NOW CISA KEV PAST DUE (deadline 2026-05-15 missed).** (updated 2026-05-19: NVD entry for CVE-2026-31431 last-modified 2026-05-18, confirms inclusion in CISA Known Exploited Vulnerabilities with federal remediation deadline 2026-05-15 — that deadline is now **4 days in the past**. The MiOS base image (`stable-nvidia-lts-20260511`) is **built before the fix landed** and is therefore vulnerable. Federal-tier severity flag.) (updated 2026-05-22: **the local-root cluster has grown to FIVE named vectors.** Copy Fail unchanged (still KEV, active exploitation per Tenable). NEW (these were surfaced in the 2026-05-20 `NEXT-RESEARCH` but never folded into this doc): **`CVE-2026-31635` "DirtyDecrypt"** (RxGK LPE, CVSS 7.5, only on `CONFIG_RXGK` kernels = Fedora/Arch — public PoC 2026-05-18), **`CVE-2026-46333` "ssh-keysign-pwn"** (ptrace exit-race info-disclosure → LPE; steals SSH host keys + `/etc/shadow`), and **`CVE-2026-43500`** (new RxRPC page-cache-write LPE) now grouped with CVE-2026-43284 + CVE-2026-46300 under Red Hat **RHSB-2026-003 "Dirty Frag"**. All five detailed in §6.5. **The base image (`stable-nvidia-lts-20260511`) predates every fix.** kernel.org floors: Copy Fail ≥ 6.18.22 / 6.19.12 / 7.0; ssh-keysign-pwn ≥ 7.0.8 / 6.18.31 / 6.12.89; DirtyDecrypt + Dirty Frag present in current stable trees 7.0.9 / 6.18.32 / 6.12.90 (2026-05-17).) Affects host kernel on AMD 9950X3D + RTX 4090. Fix windows: NVD lists vulnerable kernels ≤6.18.21 / ≤6.19.11 / ≤6.12.84; fixed in **6.18.22 / 6.19.12 / 7.0** (cherry-picks have been landing on the stable trees — see §6.5 for the wider set). **Kernel 6.18.28 (2026-05-08) and 6.18.30 (2026-05-14)** ship the backports; on LTS-6.12, **6.12.88** ships them. Remediation arrives only when `ucore-hci` PR #392 (F44 base) merges and a new image is cut. (updated 2026-05-24: **PR #392 MERGED today** — the F44/longterm-6.18 remediation path is now in ucore main (see item 1). DirtyDecrypt (CVE-2026-31635) fix date **confirmed 2026-04-25 upstream**, affected ranges **6.16.1–6.18.22 / 6.19.0–6.19.12**; **F44 `updates` now ships kernel `7.0.9-205.fc44`** (past all affected ranges = fixed). kernel.org stable/longterm all bumped **2026-05-23**: 7.0.10 / 6.18.33 / 6.12.91 / 6.6.141 / 6.1.174. CVE-2026-31431 NVD reanalyzed 2026-05-21, severity unchanged (7.8, still KEV). **Net: remediation is unblocked at the source but NOT yet delivered to MiOS** — awaits a new `ucore-hci` bake on the merged base. Fragnesia/CVE-2026-46300 confirmed to be a **regression from the CVE-2026-43284 fix** (dropped `SKBFL_SHARED_FRAG` on the espintcp path), not a separate root cause.) (updated 2026-05-26: **REMEDIATION NOW BAKED at the source.** A fresh `ucore-hci:stable-nvidia-lts-20260526` image built 2026-05-26T04:19Z on the merged-#392 base (longterm-6.18 = 6.18.33 ≥ all cluster floors) — see item 1 + §2. The fix is therefore *deliverable* to MiOS now (via a Renovate digest bump or the `:stable-nvidia-lts` base-ref migration); it is not yet *delivered* until the project adopts the rebuilt image. **No new high-severity kernel local-root CVE disclosed 2026-05-24→26** — the named cluster (Copy Fail / DirtyDecrypt / ssh-keysign-pwn / Dirty-Frag) is stable; CVE-2026-31431 still 7.8 / CISA KEV (NVD reanalysis 2026-05-21); RHSB-2026-003 still "Ongoing/Important." kernel.org mainline advanced to **7.1-rc5**; stable/longterm unchanged at 7.0.10 / 6.18.33 / 6.12.91 / 6.6.141 / 6.1.174. F44 `updates` kernel unchanged at `7.0.9-205.fc44`.)
   - `CVE-2026-31431` — "Copy Fail" root LPE (Microsoft advisory 2026-05-01).
   - `CVE-2026-43398` — AMDGPU user-queue wait ioctl OOM DoS (NVD 2026-05-08).
   - `CVE-2026-43400` — AMDGPU `amdgpu_userq_signal_ioctl` OOM DoS, missing bounds check.
   - `CVE-2026-43318` — AMDGPU DMA-BUF sync GPU page faults (fixed 6.15.3 / 6.14.10 / 6.12.20).
   - `CVE-2026-43305` — AMDGPU DC `dmplane_atomic_check` deadlock on error path (backport in 6.13.12 / 6.12.20).
   - `CVE-2026-43298` — AMDGPU VCN 2.5 VF teardown UAF (SR-IOV).
   - `CVE-2026-43237` — AMDGPU stale DMA fences → kernel panic.
   - `CVE-2026-43320` — AMD display DSC eDP (Azure Linux advisory mirror).
   - `CVE-2026-43300` — DRM panel NULL deref (mainline 2026-05-08).
   - `CVE-2026-43287` — DRM property-blob memcg accounting.
   - `CVE-2026-43284` — "Dirty Frag" kernel LPE via ESP/RxRPC (2026-05-08).
   - `CVE-2026-46300` — "Fragnesia" networking (AlmaLinux advisory 2026-05-13).
   - **AMD iGPU exposure on 9950X3D:** the AMDGPU cluster directly hits the 9950X3D iGPU unless `amdgpu` is blacklisted. MiOS uses NVIDIA dGPU for display but iGPU is still bound by default. **Consider `modprobe.d` blacklist as defense-in-depth.**
   - **Mitigation:** Track ublue-os/ucore issue #385 for kernel rev; cherry-pick fixes are upstream-merged on the kernel-6.18 stable branch and on longterm-6.12. **MiOS image rebuild needed after `ucore-hci` kernel bump lands** — see item 1 about the stalled daily-rebuild cadence.

---

## Table of contents

| # | Topic group | Last updated |
| - | ----------- | ------------ |
| 1 | bootc + bootc-image-builder + image-builder-cli | 2026-05-26 |
| 2 | ucore-hci / Universal Blue base | 2026-05-26 |
| 3 | Fedora bootc / FCOS / composefs / OSTree | 2026-05-19 |
| 4 | Podman + Quadlet + rechunk | 2026-05-19 |
| 5 | K3s + Ceph + Pacemaker/Corosync | 2026-05-24 |
| 6 | CrowdSec + fapolicyd + usbguard + SELinux + kernel CVEs | 2026-05-26 |
| 7 | cosign/Sigstore + Secure Boot/MOK | 2026-05-26 |
| 8 | NVIDIA kmods + Container Toolkit / CDI | 2026-05-24 |
| 9 | VFIO/IOMMU + Looking Glass + KVMFR + QEMU + libvirt | 2026-05-26 |
| 10 | Gamescope + Waydroid + Mesa/ROCm | 2026-05-26 |
| 11 | FreeIPA/SSSD + GNOME + WSL2 | 2026-05-26 |
| 12 | kargs.d + Renovate + systemd-sysext + tmpfiles + bootc lifecycle | 2026-05-26 |

---

## 1. bootc + bootc-image-builder + image-builder-cli

### 1.1 bootc (containers/bootc → `bootc-dev/bootc`)
*Recorded 2026-05-11.*

- **Latest:** `v1.15.2` (2026-05-01).
- **Recent line:**
  - `v1.15.2` — `discoverable-partitions` install knob, container `sigpolicy` config knob, ZFS dataset fixes, riscv64 + s390x improvements.
  - `v1.15.1` (Apr 14) — `--karg-delete` CLI flag, Intel VROC install support, IPC namespace fixes.
  - `v1.15.0` (Mar 31) — tag-aware upgrades, `usroverlay --readonly`, composefs verity fixes, pre-flight disk-space checks.
  - `v1.14.x` (Mar 11–12) — experimental `bootc container export --format=tar`, `/usr` overlay status display.
- **Lint rules in current versions:** `nonempty-run-tmp`, `var-tmpfiles` (validates both `/etc/tmpfiles.d` and `/usr/lib/tmpfiles.d`), `kargs.d` syntax validation, single-kernel check in `/usr/lib/modules`. `Containerfile` already runs `bootc container lint` as the final step — these should pass cleanly.
- **`bootc image` subcommands:** `bootc container inspect` (v1.12) and experimental `container export --format=tar` (v1.14.1) added. No native `bootc image list/copy/push` — those remain podman territory.
- **No breaking changes** to TOML formats since v1.11. No CVEs.
- Source: `https://github.com/bootc-dev/bootc/releases`, `https://bootc.dev/bootc/building/kernel-arguments.html`.

### 1.2 bootc-image-builder (BIB)
*Recorded 2026-05-11.*

- **Image:** `quay.io/centos-bootc/bootc-image-builder:latest` (also `:rhel-9`, `:rhel-10`). No formal GitHub releases — versioning lives in container tags.
- **State:** Active. Partition layout customization (`/`, `/boot` min size, extra `/var`-mounted partitions) is mature. Format coverage: `qcow2`, `raw`, `iso` (Anaconda), `ami`, `vmdk`, `vhd`, `gce`. SBOM via osbuild integration.
- **Direction:** Upstream framing: BIB and `image-builder-cli` will **merge** into a unified tool. Treat BIB as the production tool today; not the long-term home.
- Source: `https://github.com/osbuild/bootc-image-builder`, `https://osbuild.org/docs/bootc/`.

### 1.3 image-builder-cli
*Recorded 2026-05-11.* *(updated 2026-05-16: v64 dropped "bootc is experimental" — bootc subcommand now GA; `osautomation` confirmed as typo for `osbuild`.)* *(updated 2026-05-19: no v65 yet; v64 still latest 6 days on. Issue #506 "bootc: Add support for the composefs native backend and sealed images with a UKI" (2026-04-29) is the upstream gating item for sealed-image / Atomic Desktops parity. **osbuild.org/blog/ now returns HTTP 404** — blog index appears moved/broken; flag for next pass.)* *(updated 2026-05-22: **v65 shipped 2026-05-21** — routine: deps bump to `images` 0.266.0 + cache-dir handling for non-root users; **no new format coverage** (still qcow2 + bootc-installer ISO documented). Issue #506 (composefs+UKI sealed-image bootc backend) still open, no movement since 2026-04-29. osbuild.org/blog/ still 404.)* *(updated 2026-05-26: **v66 shipped 2026-05-25** — profiling options (#516), osbuild/images bump 0.266.0 → 0.267.0 (#524), ELN packit (#521). The images bump brings new osbuild stages to mount erofs/squashfs compressed filesystems (#2348) and `container: resolve containers using skopeo` (#2346) — **no bootc/composefs/UKI/sealed-image work**. Issue #506 still open, no new activity. BIB still active (latest issues #1220/#1221, early May), no deprecation timeline. osbuild.org/blog/ still 404.)*

- **Upstream:** `github.com/osbuild/image-builder-cli`. **Latest: v64 (2026-05-13)** — PR #510 "drop 'bootc is experimental'", so the `bootc` subcommand is no longer experimental.
- **Status:** **`image-builder-cli` bootc subcommand is GA but format coverage is not yet at parity with BIB.** Canonical container is `ghcr.io/osbuild/image-builder-cli:latest`. Invocation: `podman run --privileged ghcr.io/osbuild/image-builder-cli build --distro fedora-43 --bootc-ref ... --bootc-build-ref ...`. (updated 2026-05-18: the public usage docs only enumerate **qcow2 + bootc-installer ISO** patterns; **`raw`, `ami`, `vmdk`, `vhd`, `gce` are not documented**. BIB still has the wider format matrix. Soften any claim of "BIB replacement" — until full parity is documented, image-builder-cli is a viable alternative for qcow2/ISO workflows only.) (updated 2026-05-19: BIB shows 76 open issues, no deprecation timeline announced; recent issues #1220 / #1221 (May 2026) confirm BIB remains the production path.)
- **BIB still active.** `osbuild/bootc-image-builder` remains a separate repo with open issues (e.g. #1190 "Bootcfile, a proposal"). No formal unification milestone/RFC has been published; the projects coexist with overlap.
- **`osautomation` typo confirmed.** GitHub user `osautomation` exists with **zero public repos and no GHCR packages**. The `ghcr.io/osautomation/image-builder-cli` reference in `image-versions.yml` is a typo for `ghcr.io/osbuild/image-builder-cli`. Source: https://github.com/osautomation. **Flagged ACTION REQUIRED for hand-fix.**
- Source: https://github.com/osbuild/image-builder-cli/releases, https://osbuild.org/docs/developer-guide/projects/image-builder/usage/.

---

## 2. ucore-hci / Universal Blue
*Recorded 2026-05-11.* *(updated 2026-05-16: `bsherman/ucore-hci` upstream now 404 / deleted-or-private; ublue-os/ucore-hci container rebuilt 2026-05-11; tracking issues #362 and #385 added.)* *(updated 2026-05-18: ublue-os/ucore-hci daily-rebuild cadence appears stalled — latest tag is still `stable-nvidia-lts-20260511` from 7 days ago; issues #385 and #362 still open with no new activity in the window.)* *(updated 2026-05-19: cadence still stalled (now 8 days); **upstream main is also stalled** — zero commits since 2026-05-07 (12 days idle). **NEW THIS PASS: PR #392** opened 2026-05-17 to migrate ucore's base from F43 → F44 (FEDORA_VERSION bump in Justfile + mergerfs / install-ucore tweaks). This is the implicit path to a fresh kernel — F44 carries a kernel package set newer than F43's 6.19.x branch. PR still unmerged. Issue #385 has 7 comments now (was 0 yesterday) — bodies not retrievable via WebFetch.)* *(updated 2026-05-22: **all stalls deepened, nothing merged.** `ucore-hci` newest tag still `stable-nvidia-lts-20260511` — cadence now **11 days** broken. `ublue-os/ucore` main still at the 2026-05-07 commit — now **15 days idle**. PR #392 still open, no reviews/approvals, last touched 2026-05-17. Issue #385 still open. The base image remains pre-fix for the now-5-CVE kernel local-root cluster (§6.5).)* *(updated 2026-05-24: **PR #392 MERGED 2026-05-24** (commit `412e7be`, approved @bsherman, 73 checks green) — **migrates the LTS image kernel flavor longterm-6.12 → longterm-6.18 on the F44 base**, plus mergerfs F44 RPMs + nfs-utils build fix. This **resolves the "F362 longterm-6.12 → longterm-6.18" migration** as a side effect (issue #362 effectively superseded). ucore **main is active again** (#390 mergerfs v2.42.0 on 2026-05-23, then the #392 merge). **Issue #385 (Copy Fail kernel bump) was NOT auto-closed — still open.** **No new `ucore-hci` GHCR bake confirmed yet** — needs a manual `skopeo`/registry check; the bake on the merged base is the gate for the kernel-CVE fix reaching MiOS.)* *(updated 2026-05-26: **fresh bake CONFIRMED — `ucore-hci:stable-nvidia-lts-20260526`** (registry-manifest probe: `-20260526` + `-20260525` = HTTP 200, `-20260527` = 404; newest built 2026-05-26T04:19Z). Image config `revision = e63e21f` = ucore commit #397 (2026-05-25, "use cosign login instead of docker login") → built on the post-#392-merge base; FCOS `44.20260419.3.1` stable; longterm-6.18 flavor = 6.18.33. The daily-bake stall is broken; ucore main active (#397 on 2026-05-25). **Issue #385 still open** despite the bake. Note: only the dated `stable-nvidia-lts-YYYYMMDD` tags were probed — the rolling `:stable-nvidia` tag the project actually pins was not re-checked this pass.)*

- **Project base image today:** `ghcr.io/ublue-os/ucore-hci:stable-nvidia` (Containerfile line 19 / `image-versions.yml`).
- **Upstream reality:**
  - `bsherman/ucore-hci` repo **returns HTTP 404** as of 2026-05-16 — was previously archived, now appears deleted or made private. All development consolidated into mainline `ublue-os/ucore`.
  - `ublue-os/ucore-hci` GHCR container had been actively rebuilt by the ublue-os org. Tag `stable-nvidia-lts-20260511` was published 2026-05-11; **no newer tag has appeared in 7 days as of 2026-05-18** — daily-build cadence stalled. Still the migration target, but verify a fresh tag lands before relying on it for the kernel-CVE remediation window.
  - Canonical NVIDIA tags from `ublue-os/ucore`:
    - `:stable-nvidia` — current default NVIDIA driver (590-series open, pre-signed kmod).
    - `:stable-nvidia-lts` — 580 LTS open driver (NVIDIA-recommended "preferred" since March 2026).
    - `:testing-nvidia-lts` — pre-release LTS.
- **Streams:** Daily builds across `stable`/`testing`/`lts`. `stable` tracks FCOS stable stream on kernel **6.12 LTS** for server consistency. `testing` tracks rolling upstream kernel.
- **ZFS:** Now included in all `ucore*` images (NVIDIA and non-NVIDIA) — image count reduction. Verify build assumptions don't conflict.
- **Open tracking issues + PR in `ublue-os/ucore`** (updated 2026-05-19):
  - **#385** — "Bump kernel to address 'Copy Fail' (CVE-2026-31431)" filed 2026-05-01. **Updated 2026-05-17T12:54Z, 7 new comments** (comment bodies not retrievable via WebFetch — GitHub HTML pages render comments via JS). Still open. The 8-day-stale daily-build cadence compounds the risk. *(updated 2026-05-24: **still open even though PR #392 merged today** — the maintainers did not close it on the merge, so the Copy-Fail tracker stays live until a rebuilt image actually ships and is verified to carry the fixed kernel.)*
  - **#362** — "Migrate LTS image from longterm-6.12 to longterm-6.18". *(updated 2026-05-24: **effectively resolved by PR #392** — the merged PR flips the LTS kernel flavor to longterm-6.18 on F44.)* Once the rebuilt image lands, MiOS's `iommu=pt` / VFIO assumptions need re-validation on 6.18, and the KVMFR build patch (§9.3 — `MODULE_IMPORT_NS` string-literal form + `vmalloc.h`) becomes mandatory for the bake.
  - **#392 (PR)** — "fix: resolve build failures for Fedora 44" by dylanmtaylor. *(updated 2026-05-24: **MERGED 2026-05-24**, commit `412e7be`, approved by @bsherman, all 73 checks passed.)* Migrates the LTS kernel flavor longterm-6.12 → longterm-6.18 on the F44 base + mergerfs F44 RPM manifests + nfs-utils build fix. **This is the kernel-CVE remediation merge path** — but the fix only reaches MiOS once a new `ucore-hci:stable-nvidia-lts-YYYYMMDD` bakes on the merged base. *(updated 2026-05-26: **that bake landed — `stable-nvidia-lts-20260526`, built 2026-05-26T04:19Z on the merged base** (image revision `e63e21f` = ucore #397, 2026-05-25). The upstream gate is closed; remaining work is project-side adoption.)* Note issue #385 did NOT auto-close (still open as of 2026-05-26).
- **Recommendation for MiOS:** Migrate `BASE_IMAGE` from `ghcr.io/ublue-os/ucore-hci:stable-nvidia` to `ghcr.io/ublue-os/ucore:stable-nvidia-lts`. Renovate's `customManagers` regex on `Containerfile` will need its `depName` updated too. **Flagged ACTION REQUIRED.**
- **Note on releases:** `ublue-os/ucore` has **zero published GitHub Releases**; tags are flowed only through OCI registry. Renovate must track via the docker datasource on the GHCR image, not via github-releases.
- Source: https://github.com/ublue-os/ucore, https://github.com/ublue-os/ucore/issues, https://github.com/ublue-os/ucore/pkgs/container/ucore-hci.

---

## 3. Fedora bootc / FCOS / composefs / OSTree

### 3.1 Fedora releases
*Recorded 2026-05-11.*

- **Fedora 44** released **2026-04-28**. F45 branched from rawhide **2026-02-06**.
- **Fedora Atomic Desktops in F44** ship sealed container images using **UKIs + systemd-boot** — Silverblue, Kinoite, Sway Atomic, Budgie Atomic, COSMIC Atomic.
- **DEPRECATION (flagged):** **FCOS 43 disabled OSTree-repo updates** — OCI registry is now the sole update channel. F42 was the transitional dual-channel release. Anything still pulling from `ostree://` for FCOS is broken. Project uses bootc/OCI flow, so this is informational, not a blocker.

### 3.2 composefs
*Recorded 2026-05-11.*

- **Latest:** `v1.0.8` (2025-01-03). **No tagged release in 2026 yet.**
- **Changes since 1.0.7:** fs-verity measurement APIs, EROFS file-backed mount support, userspace signatures replace built-in fs-verity signatures, EROFS bloom filters for xattr lookup, small files inlined.
- **bootc integration:** bootc uses composefs by default for `/` (via ostree). Default base-image config does **not** require signatures/fsverity. The native composefs backend remains **experimental** (bootc docs note on-disk formats may change); waiting on overlayfs kernel changes for GA.
- **Atomic Desktops:** Composefs enabled by default since F42 sealed images; carried into F44.
- Source: `https://github.com/composefs/composefs/releases`, `https://bootc.dev/bootc/experimental-composefs.html`.

### 3.3 OSTree (libostree)
*Recorded 2026-05-11.*

- **Latest:** `v2026.1` (2026-04-10).
- **Changes:** soft-reboot mount fixes (`var`/`sysroot`/`boot`), extension BLS key preservation, `ostree admin status --json` includes origin refspec. Composefs signature support for bootc commits (v2025.7).
- **Deprecation posture:** libostree is the storage substrate beneath bootc; actively maintained, no sunset. FCOS's drop of OSTree-repo *update delivery* is distinct from libostree itself.
- Source: `https://github.com/ostreedev/ostree/releases`.

### 3.4 Fedora 45 schedule + Atomic Desktops direction
*Added 2026-05-19.*

- **F45 Beta:** confirmed **2026-08-25** via cross-reference (Wikipedia release-history page + Fedora ChangeSet wiki). Beta freeze ≈ 2 weeks prior (~2026-08-11) per Fedora milestone-freeze policy.
- **`fedoraproject.org/wiki/Releases/45/Schedule` returns HTTP 404** on second consecutive pass — the wiki page may not exist yet. `fedorapeople.org/groups/schedule/f-45/*` is now Anubis-gated. Use F45 ChangeSet wiki (`fedoraproject.org/wiki/Releases/45/ChangeSet`) as the authoritative substitute.
- **F45 Atomic Desktops direction (from ChangeSet):** "switch the builds of the Fedora Atomic Desktop ISOs over from lorax to image-builder" + add qcow2 / raw artifacts. **Composefs+UKI sealed-image (`composefs` native bootc backend + UKI-signed boot path) is NOT confirmed as a default F45 deliverable** — the work continues in `travier/fedora-atomic-desktops-sealed` (WIP, unofficial test images). Sealed-image GA is still gated on bootc-side composefs native backend leaving experimental status.
- Source: https://fedoraproject.org/wiki/Releases/45/ChangeSet, https://en.wikipedia.org/wiki/Fedora_Linux_release_history, https://github.com/travier/fedora-atomic-desktops-sealed.

---

## 4. Podman + Quadlet + rechunk

### 4.1 Podman + Quadlet
*Recorded 2026-05-11.* *(updated 2026-05-16: Podman 6.0 GA imminent — Fedora Test Days closed 2026-05-15, GA target week of 2026-05-25.)* *(updated 2026-05-18: **GA target slipped — Fedora Change Proposal now tagged `ChangeAcceptedF45`** (last edited 2026-03-11, so the F44 target was actually abandoned before the bootstrap pass). No 6.0 RC tag has been cut as of 2026-05-18 — upstream latest is still v5.8.2. Test Days closed 2026-05-15 with no public post-test-day report. New realistic target: F45 (Oct 2026).)* *(updated 2026-05-19: still no 6.0 RC tag; no v5.8.3 either. Atom feed metadata for v5.8.2 shows updated 2026-05-07 — likely tag metadata refresh, not a content change. Fedora wiki Podman6 change page is unchanged — no Quadlet schema delta documentation has been published yet.)* *(updated 2026-05-22: still **v5.8.2** latest stable, **no 6.0 RC/beta/GA tag**, no v5.8.3. Quadlet pre-flight remains F45-paced.)*

- **Latest stable:** **Podman v5.8.2** (2026-04-14). **v6.0 GA slipped — Fedora Change retargeted to F45.** No upstream RC tag exists yet. Fedora ran Podman 6.0 Test Days 2026-05-11 → 2026-05-15.
- **Recent line:**
  - `v5.8.2` (Apr 14) — fixes for `unless-stopped` restart policy and Quadlet config bugs; CVE fix (see below).
  - `v5.8.0` (Feb 12) — Quadlet supports multiple units per file via `---` delimiters; new `AppArmor=` and `HttpProxy=` keys for `.container`; `podman update --ulimit`; mandatory BoltDB→SQLite migration.
  - `v5.7.0` (Nov 2025) — TLS/mTLS for remote, multi-YAML `podman kube play`, new **`.artifact` Quadlet unit type**.
- **Podman 6.0 breaking removals** (updated 2026-05-18; target now F45 per `fedoraproject.org/wiki/Changes/Podman6` `ChangeAcceptedF45` tag):
  - **BoltDB removed** — SQLite is the only storage backend. v5.8.0 auto-migration is a hard prerequisite; un-migrated state will not boot under 6.0.
  - **slirp4netns removed** — Pasta is the only rootless networking backend.
  - **cgroups v1 removed** — host must be running unified-cgroup-v2 (Fedora bootc already is, so MiOS unaffected).
  - **netavark default switches iptables → nftables** — any host-side firewall integration assuming iptables backend needs review.
- **Quadlet unit coverage in 6.0:** `.container`, `.volume`, `.network`, `.build` (with `BuildArg`, `IgnoreFile`), `.pod` (with `StopTimeout`), `.kube` (multi-doc YAML), `.image`, `.artifact`. Templated vol/net dependencies stabilized.
- **AutoUpdate=registry:** Behavior unchanged — requires systemd-managed unit, daily timer (`podman-auto-update.timer`). Use `local` policy when CI pre-pulls.
- **LBI (Logically Bound Images):** Stable. **Quadlet remains the recommended path** for bootc-integrated lifecycle — `podman kube` works but is not the bootc-preferred direction. No move away from Quadlet observed.
- **CVE:** **CVE-2026-33414** — Podman 5.8.x Windows Hyper-V backend. **Not relevant to Linux bootc deployments**, but note when bumping `podman` version label.
- **Project status (LBI):** `Containerfile` lines 67–76 currently have LBI pre-pull **disabled** due to lack of `--privileged` BuildKit on GitHub-hosted runners. Quadlet `AutoUpdate=registry` first-boot pull (commented hint in Containerfile) is the migration path.
- **Pre-flight review needed before pulling Podman 6.0** — see `NEXT-RESEARCH.md` ACTION REQUIRED item. Verify no MiOS Quadlet relies on slirp4netns, no iptables-only host integration, no BoltDB on-disk state surviving across the 5.8 → 6.0 boundary. (updated 2026-05-18: deadline pressure relieved by GA slip — pre-flight is still required but is now F45-paced rather than late-May.)
- Source: https://github.com/containers/podman/releases, https://bootc.dev/bootc/logically-bound-images.html, https://communityblog.fedoraproject.org/join-us-for-podman-6-0-test-days-may-11-15-2026/.

### 4.2 `bootc-base-imagectl rechunk`
*Recorded 2026-05-11.*

- Subcommand of `bootc-base-imagectl`; distinct from the `hhd-dev/rechunk` GitHub Action (different impl, similar goal).
- **`--max-layers` guidance:** Upstream examples 64–96. **Project default of 67 is in the sweet spot.** No upstream recommendation has shifted.
- **Bazzite-reported metrics:** ~40% weekly download reduction, 60–80% daily, >90% back-to-back. 6–10 min processing overhead.
- Source: `https://github.com/hhd-dev/rechunk`.

---

## 5. K3s + Ceph + Pacemaker/Corosync

### 5.1 K3s
*Recorded 2026-05-11.* *(updated 2026-05-16: v1.34.8-rc1, v1.35.5-rc1 cut 2026-05-14; v1.36.0 stable 2026-05-06; etcd 3.5.30 shipped 2026-05-01.)* *(updated 2026-05-18: v1.34.8 / v1.35.5 GA still pending — RCs cut 2026-05-14 have not promoted in the 2-day window. **CVE-2026-33186 (gRPC-Go authz bypass via malformed `:path`, CVSS 9.1) is still NOT explicitly called out** in any v1.34.8-rc / v1.35.5-rc release notes. Watch item until GA.)* *(updated 2026-05-19: still RC-only after 5 days — `v1.36.1-rc1+k3s1`, `v1.35.5-rc1+k3s1`, `v1.34.8-rc1+k3s1` all cut 2026-05-14. v1.35.5-rc1 release notes inspected this pass — backports for 2026-05, local-path-provisioner image bump, **Go 1.25.9**, klipper-helm tag bump. Still **no grpc-go bump or CVE-2026-33186 callout**. Whether grpc-go ≥1.79.3 is pulled in via Go-1.25.9's vendored deps is not visible from release notes — would require inspecting `go.mod` in the RC branches. CVE-2026-33186 confirmed details: critical CVSS 9.1, authorization bypass via improper HTTP/2 `:path` validation, fixed in grpc-go v1.79.3+, published 2026-03-17.)* *(updated 2026-05-22: **GA SHIPPED — all three RCs promoted to GA on 2026-05-20:** `v1.36.1+k3s1` (latest), `v1.35.5+k3s1`, `v1.34.8+k3s1`, plus `v1.33.12+k3s1`. **CVE-2026-33186 is REMEDIATED** even though the GA release notes are silent on it: the `v1.36.1+k3s1` `go.mod` carries `replace google.golang.org/grpc => google.golang.org/grpc v1.79.3` (the GHSA-p77j-4mvh-x3m3-fixed version) overriding `require ...grpc v1.80.0` — the `replace` wins at build time, so the shipped binary links grpc-go v1.79.3. **CORRECTION: vendored Go is 1.26.2, not 1.25.9** as the 2026-05-19 note recorded. Recommend bumping the project's K3s pin to a GA tag.)*

- **Latest stable:** `v1.34.7+k3s1`. **NEW: `v1.36.0+k3s1` stable shipped 2026-05-06** (Kubernetes 1.36). **`v1.36.1+k3s1` / `v1.35.5+k3s1` / `v1.34.8+k3s1` GA shipped 2026-05-20** (the RCs cut 2026-05-14 promoted to GA), plus `v1.33.12+k3s1`. These carry the CVE-2026-33186 grpc-go v1.79.3 fix via a `replace` directive (notes silent). `v1.32.x` on the older maintenance line.
- **Bundled runtimes:** v1.34 → containerd `2.2.3-k3s1`, runc `1.4.2`; v1.33 → containerd `2.1.x` / runc `1.3.4`.
- **etcd line state** (updated 2026-05-16):
  - v1.34 ships embedded **etcd 3.6.7-k3s1**; v1.33 stays on 3.5.x.
  - **etcd 3.5.30 shipped 2026-05-01** (latest 3.5.x). 3.5.29 (2026-04-01), 3.5.28 (2026-03-20 — security release: CVE-2026-33343 nested-txn authz bypass + CVE-2026-33413 gRPC authn bypass).
  - **Migration: etcd 3.5 → 3.6 is NOT direct** — must transit through one of v3.5.26+ first (3.5.30 is fine).
  - (updated 2026-05-22: stable trains unchanged — 3.6.11 / 3.5.30 / 3.4.44 (2026-05-01). New pre-release **v3.7.0-beta.0 (2026-05-19)** — not for production; no new security patch release.)
- **Containerd config:** Now uses **versioned drop-in dirs** (`config.toml.d` for v2, `config-v3.toml.d` for v3) — auto-loaded.
- **SELinux:** `k3s-selinux` policy still ships separately; `selinux=true` in config. No regressions reported.
- **Sqlite remains default** for single-node; embedded etcd still required for HA.
- **CVEs (ecosystem):**
  - `CVE-2026-33186` — gRPC-Go authz bypass via malformed `:path`; fixed in grpc-go v1.79.3. CVSS 9.1 per `GHSA-p77j-4mvh-x3m3`. (updated 2026-05-18: K3s v1.34.8-rc / v1.35.5-rc notes **still do not explicitly call out a grpc-go bump** as of 2026-05-18 — RCs unchanged in the 2-day window. Continues to be a watch item.) (updated 2026-05-22: **RESOLVED — remediated in the 2026-05-20 GA.** GA notes are still silent, but `v1.36.1+k3s1` `go.mod` pins grpc-go to v1.79.3 via a `replace` directive overriding the v1.80.0 `require`. CVE closed for any host on a K3s GA tag.)
  - `CVE-2026-33343` — etcd nested-txn authz bypass (fixed in 3.5.28).
  - `CVE-2026-33413` — etcd gRPC authn bypass (fixed in 3.5.28).
- Source: https://github.com/k3s-io/k3s/releases, https://github.com/etcd-io/etcd/releases, https://etcd.io/blog/2026/mar20-patch-release/.

### 5.2 Ceph
*Recorded 2026-05-11.*

- **Latest:** **20.2.1 Tentacle** (released 2026-04-06). 19.2.x Squid supported through ~Sept 2026. Reef (18.2) is **EOL**.
- **rook-ceph:** v1.19.x (v1.19.3 patch). Minimum Ceph **19.2.0**.
- **Single-node knobs:** `osd_pool_default_size = 1` and `allowMultiplePerNode: true` for mon+mgr colocation.
- **cephadm:** Container deploy path supported.
- No new 2026 Ceph CVEs surfaced.
- Source: `https://ceph.io/en/news/blog/2026/v20-2-1-tentacle-released/`.

### 5.3 Pacemaker / Corosync
*Recorded 2026-05-11.*

- **Pacemaker:** 3.0.x line (3.0.0 Jan 2025; minors throughout 2025–early 2026). Added **X.509/TLS** for Pacemaker Remote + remote CIB admin. (updated 2026-05-18: **3.0.2-rc2 cut 2026-05-11** — 45 commits, XPath + memory-leak fixes. 3.0.1 final was 2025-08-07; expect 3.0.2 final shortly after the RC stabilizes.) (updated 2026-05-19: 3.0.2 final **NOT yet shipped**. rc2 is now 8 days old; rc1→rc2 gap was ~17 days; if the same cadence holds, expect final ~2026-05-28 ± a few days.) (updated 2026-05-22: still **`3.0.2-rc2` (2026-05-11)** — no rc3, no 3.0.2 final. Now 11 days into rc2; projected final ~2026-05-28 still plausible. Last stable remains 3.0.1.) (updated 2026-05-24: no change — still `3.0.2-rc2`, no rc3/final. Projected ~2026-05-28; tight watch.)
- **Corosync:** **v3.1.10 (2024-11-15)** is current — addresses **CVE-2025-30472**. No 2026 release activity visible. (Previously the doc said `3.1.1` — that was the bootstrap-pass baseline before the 3.1.10 line existed; corrected on 2026-05-19.) New extended node/link info API; cfgtool uses it; cfg tracking callback fixed.
- **Breaking from 2.x line (still relevant for any rebase):** Dropped rolling upgrades from <2.0.0; 3.0 nodes cannot talk to Pacemaker 1.1.14 or earlier Remote endpoints. Stricter XML validation; deprecated env vars removed.
- **bootc fit:** Config in `/etc/corosync/` + state in `/var/lib/pacemaker/` — both writable. No Quadlet rework needed.
- No new CVEs.

---

## 6. CrowdSec + fapolicyd + usbguard + SELinux

### 6.1 CrowdSec
*Recorded 2026-05-11.* *(updated 2026-05-18: latest 1.7.8, 2026-05-11; bootstrap baseline 1.7.6 superseded.)*

- **Agent:** **`v1.7.8` (2026-05-11)** — latest. Line cadence in window: 1.7.6 → 1.7.7 (2026-03-30) → 1.7.8 (2026-05-11). 1.7.8 adds **WAF OpenAPI schema validation**, body-size limits, decision-stream chunked-transfer improvements. (Sub-agent noted some WebFetch results returned 2024 dates that look misparsed — release ordering and content match the 2026 cadence; treat 1.7.8 as current.)
- **`cs-firewall-bouncer`:** Continues to support iptables / nftables / **firewalld** backends.
- **SELinux:** **No upstream-shipped CrowdSec SELinux module exists.** MiOS continues to need a local policy module (or confined container) for fapolicyd/enforcing hosts.
- No new CVEs in window.
- Source: https://github.com/crowdsecurity/crowdsec/releases.

### 6.2 fapolicyd
*Recorded 2026-05-11.* *(updated 2026-05-18: bootstrap baseline 1.3.8 was stale — current latest is **v1.4.5**.)* *(updated 2026-05-19: date correction — v1.4.5 shipped **2026-03-30**, not 2025-03-30. The previous WebFetch parsed the HTML date as "2025"; the project's GitHub releases atom feed renders the date as 2026 and is internally consistent with the 1.4.x cadence (1.4.2 = 2025-11-26, 1.4.3 = 2026-01-13, 1.4.4 = 2026-03-19, 1.4.5 = 2026-03-30). Atom feed treated as authoritative.)* *(updated 2026-05-22: **v1.5 shipped 2026-05-20** — first minor bump off the 1.4.x line. Adds transactional rule reload, `--check-rules` validation, per-rule hit counters, and a decision-timing framework. **Review rule-reload behavior before pinning** — a minor bump on a deny-by-default execution gate warrants validating the trust DB / rules still load identically at image-build time.)*

- **Latest:** **`v1.5` (2026-05-20)** — first minor bump off the 1.4.x line (transactional rule reload, `--check-rules`, per-rule hit counters, decision-timing framework). `v1.4.5` (2026-03-30) was the prior latest. Verify reload/trust-DB parity before pinning v1.5.
- **Historical line context (1.3.x, retained for reference):**
  - 1.3.8 — `ignore_mounts` perf option (drop noisy mounts from fanotify; useful on bootc overlays).
  - 1.3.7 — unified queue enqueue/dequeue, improved `text/x-shellscript` detection, `--ftype` regression fix, state report includes watched mount points.
  - 1.3.6 — larger default subject cache, descriptor leak fix.
- **1.4.x highlights** (verify against image-builder pin): re-check whether Fedora 43/44 has actually shipped 1.4.x; the project may still be on the 1.3.8 Fedora package version. **Worth confirming via `dnf list fapolicyd` inside the Containerfile build context** rather than assuming upstream-latest.
- **bootc trust-DB integration:** `fapolicyd-dnf-plugin` only fires on package transactions; bootc does these at image build, not runtime. **Trust DB needs to be rebuilt at image build time (`fapolicyd-cli --update`) and shipped baked, or regenerated on first boot.** No dedicated dnf5-OSTree-bootc native integration yet.
- No CVEs in window.
- Source: https://github.com/linux-application-whitelisting/fapolicyd/releases.

### 6.3 usbguard
*Recorded 2026-05-11.*

- **Latest:** `1.1.4`. Mature, low-churn project.
- Changes: `FDStreamBuf` destructor fix for fd leak in `FDInputStream`, CI dep bumps.
- Defaults: `ImplicitPolicyTarget=block`, `PresentDevicePolicy=apply-policy`, `InsertedDevicePolicy=apply-policy` remain sensible.
- No CVEs.

### 6.4 SELinux on bootc
*Recorded 2026-05-11.*

- **Policy version:** Fedora 43 `selinux-policy-minimum-42.12-1.fc43` line.
- **Critical persistence rule on composefs/bootc:**
  - `/usr/lib/selinux/.../policy/*` ships in the image; **replaced wholesale** on each bootc update — local edits there do **not** persist.
  - `/etc` and `/var` are mutable. `semodule -i` writes to `/etc/selinux/targeted/active/modules/400/` which **does persist** across bootc updates.
  - For image-time policy: build with `checkmodule` + `semodule_package` + `semodule -i` during the Containerfile build.
- **Project status:** `Containerfile` line 80 installs `selinux-policy-targeted`. SECURITY.md lists site modules (`mios_portabled`, `mios_kvmfr`, `mios_cdi`, `mios_quadlet`, `mios_sysext`). Verify those land in `/etc/selinux/targeted/active/modules/400/` (or are baked correctly into `/usr/share/selinux/packages/`).
- **Operational pain points:** Fedora 43 Silverblue reports of Quadlet units entering restart loops under enforcing; boot-time denial reports surfaced April 2026. Track before promoting any new Quadlet-managed K3s/Ceph/CrowdSec service.
- No CVEs.

### 6.5 Linux kernel CVE cluster — May 2026
*Added 2026-05-16.* *(updated 2026-05-18: expanded from 5 → ~12 CVEs; cluster now dominated by AMDGPU subsystem.)* *(updated 2026-05-19: **CVE-2026-31431 is on the CISA Known Exploited Vulnerabilities list with federal remediation deadline 2026-05-15 — that deadline is now 4 days past**. NVD entry last-modified 2026-05-18 confirms KEV inclusion. AlmaLinux's CVE-2026-46300 ("Fragnesia") advisory page lists fixed-kernel pins: AlmaLinux 10 = `kernel-6.12.0-124.56.3.el10_1`, AlmaLinux 9 = `kernel-5.14.0-611.54.5.el9_7`, AlmaLinux 8 = `kernel-4.18.0-553.124.3.el8_10`.)* *(updated 2026-05-22: **cluster grew from a 1-named-LPE focus to FIVE named local-root vectors.** Added below: `CVE-2026-31635` "DirtyDecrypt", `CVE-2026-46333` "ssh-keysign-pwn", and `CVE-2026-43500` (folded into the "Dirty Frag" / RHSB-2026-003 grouping). **kernel.org current stable/longterm (2026-05-17, unchanged through 2026-05-22): 7.0.9 / 6.18.32 / 6.12.90 / 6.6.140 / 6.1.173; mainline 7.1-rc4.** All five vectors are fixed in these trees; the MiOS base (`stable-nvidia-lts-20260511`) predates all of them.)* *(updated 2026-05-24: **kernel.org bumped 2026-05-23** — mainline 7.1-rc4; stable **7.0.10**; longterm **6.18.33 / 6.12.91 / 6.6.141 / 6.1.174** / 5.15.208 / 5.10.257. **DirtyDecrypt fix date resolved: 2026-04-25 upstream**, affected ranges **6.16.1–6.18.22 and 6.19.0–6.19.12** (so fixed in 6.18.23 / 6.19.13 / 7.0+). **Fedora 44 `updates` now ships kernel `7.0.9-205.fc44`** — past every affected range, so the current F44 kernel carries the DirtyDecrypt + Copy-Fail + ssh-keysign-pwn + Dirty-Frag fixes. **ublue-os/ucore PR #392 MERGED 2026-05-24** moves the LTS image to longterm-6.18 on F44 (see §2 + items 1/8) — but the new image must bake ≥ 6.18.23 (DirtyDecrypt) and ≥ 6.18.31 (ssh-keysign-pwn); current longterm-6.18 is 6.18.33, so a fresh bake satisfies. **The fix is unblocked at the source but not yet delivered to MiOS** — no rebuilt `ucore-hci` tag confirmed.)* *(updated 2026-05-26: **rebuilt tag CONFIRMED — `ucore-hci:stable-nvidia-lts-20260526`** built 2026-05-26T04:19Z on the merged-#392 base (longterm-6.18 = 6.18.33 ≥ all cluster floors); see §2 + items 1/8. The cluster fix is now baked and *deliverable* (Renovate digest bump or `:stable-nvidia-lts` migration) but not yet *delivered* to MiOS. No new kernel local-root CVE this window; kernel.org mainline → **7.1-rc5**, stable/longterm unchanged (7.0.10 / 6.18.33 / 6.12.91 / 6.6.141 / 6.1.174); F44 `updates` kernel unchanged at `7.0.9-205.fc44`.)*

Kernel CVEs disclosed in the 2026-05-01 → 2026-05-18 window directly affect the MiOS host kernel (AMD 9950X3D + RTX 4090). Fixes are merged on the kernel-6.18 stable branch and backported to longterm-6.12. **Specifically: kernel 6.18.28 (2026-05-08), 6.18.30 (2026-05-14), 6.12.87, and 6.12.88** carry the backports. MiOS rebuild required after `ublue-os/ucore` issue #385 lands a kernel rev; as of 2026-05-19 issue #385 is still open, **PR #392 is the implicit kernel-bump path (F43 → F44 base) and is unmerged**, and the ucore-hci daily-rebuild cadence has now been stalled 8 days.

**General LPE / network:**
- **CVE-2026-31431 — "Copy Fail"** (`algif_aead` AF_ALG LPE; Microsoft advisory 2026-04-22, published to NVD 2026-04-22, last-modified 2026-05-18). **CVSS 7.8. ON CISA KEV; federal remediation deadline 2026-05-15 (now 4 days past).** NVD-listed vulnerable kernels: ≤6.18.21 / ≤6.19.11 / ≤6.12.84. Fixed in **6.18.22, 6.19.12, 7.0** (stable cherry-picks landing rolling onto LTS). Source: https://nvd.nist.gov/vuln/detail/CVE-2026-31431, https://access.redhat.com/security/cve/cve-2026-31431, https://www.sysdig.com/blog/cve-2026-31431-copy-fail-linux-kernel-flaw-lets-local-users-gain-root-in-seconds.
- **CVE-2026-31635 — "DirtyDecrypt"** *(added 2026-05-22).* RxGK (AFS rxrpc auth) LPE. **CVSS 7.5. Only triggers on `CONFIG_RXGK` kernels (Fedora / Arch / openSUSE Tumbleweed) → applies to the MiOS Fedora base.** Fix is upstream (commit `aa54b1d27fe0`). *(updated 2026-05-24: **date conflict RESOLVED — fixed 2026-04-25 upstream** per the NVD entry; **affected ranges 6.16.1–6.18.22 and 6.19.0–6.19.12** → fixed in 6.18.23 / 6.19.13 / 7.0+.)* **Public PoC released 2026-05-18.** Present in kernel.org stable trees 7.0.10 / 6.18.33 / 6.12.91 (2026-05-23). Container/pod-escape pathway noted. **Fedora NVR (updated 2026-05-24): F44 `updates` now ships `kernel-7.0.9-205.fc44`** (mirrors.kernel.org `updates/44/.../k/`) — 7.0.x is past the affected range, so the current F44 kernel carries the fix; exact first-fixed F44 NVR not pinpointed (bodhi Anubis-gated) but any 7.0.x F44 build is safe. Defense-in-depth if unused: RxGK is an AFS/rxrpc auth surface MiOS almost certainly doesn't use — blacklisting `rxrpc` removes it, but `CONFIG_RXGK` is base-kernel config the project doesn't own. Source: https://thehackernews.com/2026/05/dirtydecrypt-poc-released-for-linux.html.
- **CVE-2026-46333 — "ssh-keysign-pwn"** *(added 2026-05-22).* ptrace exit-race info-disclosure → LPE; steals SSH host keys + `/etc/shadow`. Disclosed 2026-05-15; fix commit `31e62c2ebbfd` (Linus, 2026-05-14). **Fix floors: 7.0.8 / 6.18.31 / 6.12.89 / 6.6.139 / 6.1.173** (also 5.15.207 / 5.10.256). Source: https://blog.qualys.com (CVE-2026-46333 advisory, 2026-05-20).
- **CVE-2026-43284 — "Dirty Frag" (2026-05-08).** Kernel LPE via ESP / RxRPC fragmentation. *(updated 2026-05-22: now grouped by Red Hat under **RHSB-2026-003 "Dirty Frag"** together with **`CVE-2026-43500`** (new — RxRPC page-cache-write LPE) and `CVE-2026-46300`. Mechanism: deterministic LPE-to-root via `splice` / `MSG_SPLICE_PAGES` page-cache writes. **Public PoC exists; affects Fedora via the RxRPC path.** Source: https://access.redhat.com/security/vulnerabilities/RHSB-2026-003.)* Source: https://www.wiz.io/blog/dirty-frag-linux-kernel-local-privilege-escalation-via-esp-and-rxrpc.
- **CVE-2026-43500 — RxRPC page-cache-write LPE** *(added 2026-05-22).* New member of the "Dirty Frag" / RHSB-2026-003 grouping (see the CVE-2026-43284 bullet above). Source: https://access.redhat.com/security/vulnerabilities/RHSB-2026-003.
- **CVE-2026-46300 — "Fragnesia" (AlmaLinux advisory 2026-05-13).** Networking-stack vulnerability — `skb_try_coalesce` + `SKBFL_SHARED_FRAG` LPE via ESP-in-TCP / rxrpc. Patched 2026-05-13. *(updated 2026-05-22: also folded into Red Hat's RHSB-2026-003 "Dirty Frag" grouping.)* AlmaLinux fixed-kernel pins: 10 = `6.12.0-124.56.3.el10_1`, 9 = `5.14.0-611.54.5.el9_7`, 8 = `4.18.0-553.124.3.el8_10`. Source: https://almalinux.org/blog/2026-05-13-fragnesia-cve-2026-46300/.

**AMDGPU cluster** (added 2026-05-18 — most relevant to 9950X3D iGPU):
- **CVE-2026-43398 — AMDGPU user-queue wait ioctl OOM DoS (NVD 2026-05-08).**
- **CVE-2026-43400 — AMDGPU `amdgpu_userq_signal_ioctl` OOM DoS** (missing bounds check).
- **CVE-2026-43318 — AMDGPU DMA-BUF sync GPU page faults.** Fixed in 6.15.3 / 6.14.10 / 6.12.20.
- **CVE-2026-43305 — AMDGPU DC `dmplane_atomic_check` deadlock** on error path. Backported to 6.13.12 / 6.12.20.
- **CVE-2026-43298 — AMDGPU VCN 2.5 VF teardown UAF (SR-IOV).** Not directly relevant to single-host MiOS posture but lives in the same backport set.
- **CVE-2026-43237 — AMDGPU stale DMA fences → kernel panic.**
- **CVE-2026-43320 — AMD display DSC eDP** (Azure Linux advisory mirror; same kernel set).

**DRM core (NVIDIA-friendly path also affected):**
- **CVE-2026-43300 — DRM panel NULL deref (mainline 2026-05-08).** DRM core. Source: https://windowsnews.ai/article/cve-2026-43300-linux-drm-null-pointer-flaw-flagged-by-microsoft-for-windows-environments.417437.
- **CVE-2026-43287 — DRM property-blob memcg accounting (2026-05-08).**

**AMD iGPU exposure on 9950X3D (added 2026-05-18):** MiOS uses NVIDIA dGPU for display, but the AMD iGPU on the 9950X3D is still bound by the kernel `amdgpu` driver unless explicitly blacklisted. The AMDGPU CVE cluster therefore lands on MiOS hosts directly. **Defense-in-depth recommendation:** consider `modprobe.d` blacklist of `amdgpu` if the iGPU is unused in the MiOS workflow. Flagged as a follow-up question for the project owner — research agent does not apply.

**Project status:** None of these have remediation paths the MiOS image owns directly — the fix lands when `ucore-hci`'s base kernel is bumped. Track `ublue-os/ucore` issue #385 (Copy Fail tracker, opened 2026-05-01); also implicitly tracks the AMDGPU cluster since they share the same backport window. Once the kernel rev lands in `stable-nvidia-lts`, MiOS needs a rebuild for the CVE pin to take effect.

**Note on CVE-2026-31431 mis-attribution:** The bootstrap pass on 2026-05-11 listed this CVE under §7.1 (cosign) as a `verify-blob-attestation` bug. **That was wrong** — the CVE is the kernel "Copy Fail" LPE. Corrected here and in §7.1.

---

## 7. cosign / Sigstore + Secure Boot / MOK

### 7.1 cosign / Sigstore
*Recorded 2026-05-11.* *(updated 2026-05-19: `GHSA-w6c6-c85g-mmv6` now has CVE assignment **CVE-2026-39395** (CVSS 4.3 Moderate, `verify-blob-attestation` false positive); already fixed in v3.0.6, no remediation change. No new releases; v3.0.6 still latest.)* *(updated 2026-05-22: re-verified — **still v3.0.6 (2026-04-06)**, no newer release, no new GHSAs/CVEs.)*

- **Latest:** **cosign 3.0.6** (2026-04-06). policy-controller **v0.15.1** (2026-03-26; bumped internal cosign v2→v3).
- **cosign 3 defaults:**
  - Standardized **Sigstore bundle format** is the default.
  - OCI 1.1 referrer artifacts used for signatures.
  - Single-file trust root + signing service URL.
  - `--bundle` flag for bundle output is **REQUIRED** (was optional in v2).
- **Breaking:** Bundle-format default affects CI signing pipelines and any signature artifacts stored alongside images. policy-controller 0.15.x requires re-checking `ClusterImagePolicy` CRDs against v3 trust-root format.
- **CVEs (FLAG — pin ≥ 3.0.6):** *(updated 2026-05-16: removed mis-attribution of CVE-2026-31431 to cosign — that CVE is the Linux kernel "Copy Fail" LPE, not cosign-related. See §6.5 for the kernel CVE cluster.)*
  - `CVE-2026-22703` — Cosign accepts a Rekor entry that doesn't reference the artifact's digest/sig/key (verification bypass). Fixed in 2.6.2 / 3.0.4.
  - **Cosign GHSAs** (collectively fixed in **3.0.6 / 2.6.3**): `GHSA-w6c6-c85g-mmv6` (published 2026-04-06; CVE assignment **CVE-2026-39395** added 2026-05-19, CVSS 4.3 Moderate — `verify-blob-attestation` false positive), `GHSA-wfqv-66vq-46rm`, `GHSA-whqx-f9j3-ch6m`. Pinning cosign ≥ 3.0.6 covers all currently-known sig-verification bypasses.
- **Project status:** `automation/42-cosign-policy.sh` exists. Verify cosign binary version baked in and confirm signature verification flow uses the new bundle format.
- Source: https://blog.sigstore.dev/cosign-3-0-available/, https://github.com/sigstore/cosign/security/advisories.

### 7.2 Secure Boot / MOK
*Recorded 2026-05-11.* *(updated 2026-05-18: bodhi.fedoraproject.org now gated by Anubis — direct fetch blocked. No Fedora Discussion thread in the 2-day window indicates shim-16.1-6 has reached F44 stable. **Schedule next checkpoint for 2026-06-05** — if shim-16.1-6 still has not landed in F44 stable by then, MiOS needs a fallback plan before the 2026-06-26 cutover.)* *(updated 2026-05-19: **STATUS UNCERTAIN** — all Fedora infrastructure (Koji `packageinfo`, Bodhi, src.fedoraproject.org dist-git, Fedora Discussion search) is now Anubis-gated and returns "Access Denied: error code 9e4edb5b6b850c41". WebFetch cannot bypass. **Working alternate paths** (not Anubis-gated): (a) `https://dl.fedoraproject.org/pub/fedora/linux/updates/44/Everything/x86_64/Packages/s/` for stable F44, (b) `https://dl.fedoraproject.org/pub/fedora/linux/updates/testing/44/Everything/x86_64/Packages/s/` for updates-testing, (c) `https://mirrors.fedoraproject.org/metalink?repo=updates-released-f44&arch=x86_64` for a metalink with current package hashes, (d) `https://pagure.io/fedora-package-sources/shim` for dist-git source on Pagure. Recommended: add a poll job (curl + grep on the dl.fedoraproject.org mirror) to detect shim-16.1-6 promotion before the 2026-06-26 cutover (now 38 days away). rhboot/shim upstream tip is still **16.1 (2025-08-13)** — the `-6` suffix is Fedora's package release, not an upstream tag.)* *(updated 2026-05-22: **directly verified via `curl` on the Anubis-free mirrors.kernel.org tree** — `releases/44/Everything/x86_64/os/Packages/s/` has `shim-x64-16.1-5` / `shim-ia32-16.1-5`; `updates/44/...` and `updates/testing/44/...` contain **no shim package at all**. So **16.1-6 has not landed anywhere in F44** (stable, updates, or updates-testing). mirrors.kernel.org `releases/` + `updates/` + `updates/testing/` is now the canonical check path. Cutover ~35 days out; hard checkpoint 2026-06-05.)* *(updated 2026-05-26: **re-verified, no movement** — F44 `releases/` still `shim-x64/ia32-16.1-5` (+ `shim-unsigned-*-16.1-1`); `updates/` and `updates/testing/` still carry no shim package at all; rhboot/shim tip still tag `16.1` (2025-08-13). Cutover ~31 days out; hard checkpoint 2026-06-05 now ~10 days out. Tightest unresolved risk on the board.)*

- **Microsoft 2011 CA expiry: 2026-06-26.**
  - MS stops signing with the 2011 CA. Firmware DBX-revocation of BootHole/BlackLotus-era binaries signed by it continues.
  - **Already-installed Linux systems keep booting** — the expiry blocks **new** components signed by the 2011 cert, not chains already trusted by firmware.
  - Distros (RHEL 9.7+, Fedora in parallel) are reshipping **shim signed by the 2023 UEFI CA**.
- **Fedora 43 test day** (2026-01-12) validated multi-signed shim coverage.
- **`sbctl`:** Still only signs EFI binaries; akmods/kmod signing flow unchanged.
- **akmod NVIDIA signing on F43:** Standard flow (`openssl`/`kmodgenca` → `mokutil --import` → automatic sign on akmod build) works.
- **Action items (mirror in `NEXT-RESEARCH.md`):**
  - Pull Fedora's 2023-CA-signed shim before mid-June 2026.
  - Apply Microsoft's DBX update via `fwupdmgr` on target hardware; verify firmware accepts updated dbx.
  - Hardware that **only** trusts MS 2011 CA will fail new Fedora shim installs after cutover unless firmware updates.
- **Project status:** `automation/generate-mok-key.sh` and `automation/enroll-mok.sh` exist. Verify they don't pin the 2011 cert chain.

---

## 8. NVIDIA kmods + Container Toolkit / CDI

### 8.1 NVIDIA open kernel modules (`nvidia-open`)
*Recorded 2026-05-11.* *(updated 2026-05-16: LTS floor lifted to 580.159.04, released 2026-05-14.)* *(updated 2026-05-18: **595.44.08 Vulkan developer-beta** confirmed 2026-05-15 — clarified branch split between production-feature 595.71.x and developer-beta 595.44.x. They are not the same branch; pinning to "feature" is ambiguous unless one is specified.)* *(updated 2026-05-22: **the NVIDIA May 2026 security bulletin IS NOW PUBLISHED — corrects the prior "no May bulletin" line below.** `a_id/5821`, published 2026-05-19 (updated 2026-05-21), 13 CVEs (12 Linux), top **CVE-2026-24187** CVSS **8.8** Linux UAF. Project pins already satisfy all fix floors — no bump forced. See the new §8.1 bullet.)*

- **Production Branch (LTS, "preferred"):** **580.159.04** (released 2026-05-14 — post Jan-2026 advisory). Earlier `580.126.20` / `580.126.09` are still safe-from-CVE but superseded.
- **Feature Branches (DISTINCT):**
  - **Production-Feature: 595.71.05** (2026-04-28). Stable feature line; recommended for non-LTS production.
  - **Developer-Beta: 595.44.09** (2026-05-22 — updated 2026-05-24, supersedes 595.44.08 of 2026-05-15). Vulkan Developer Beta. **Not** the same branch as 595.71.x — beta/devel only, not a project pin. Project pins should specify which line they track.
- **No 600-series driver shipped** as of 2026-05-16. Kernel 6.18 ships no Blackwell-specific FLR/IOMMU 1:1 patches.
- **Blackwell (RTX 50-series):** Requires open modules — proprietary blob is unsupported. NVIDIA has fully transitioned to open modules.
- **Open issues to track:**
  - Issue #1117 — RTX 50-series s2idle resume hangs on **kernel 7.0**; works on 6.17.
  - Issue #1132 — GB205 (RTX 5070) BAR1→BAR3 mapping triggers `krcWatchdog` lock with rBAR disabled.
- **CVEs (Jan 2026 bulletin, advisory `a_id/5747` — pin ≥ 580.126.09 or LTS 580.159.04):**
  - `CVE-2025-33219` — integer overflow in kernel module → LPE / RCE.
  - `CVE-2025-23277`, `CVE-2025-23280` — UAF on Linux.
- **NVIDIA May 2026 bulletin (`a_id/5821`) — PUBLISHED 2026-05-19** (updated 2026-05-21; corrects the earlier "no May bulletin" status). 13 CVEs (12 Linux), top **CVE-2026-24187** CVSS **8.8** (Linux kernel-mode UAF). Linux fix floors: R595 = 595.71.05, R580 RTX/Quadro/NVS = 580.159.03, R580 GeForce = 580.126.09, R535 = 535.309.01. (GamingOnLinux lists R570 as EOL/no-fix — not a project pin, immaterial.) **Project pins already satisfy:** LTS 580.159.04 ≥ 580.159.03, feature 595.71.05 = R595 floor — **no bump forced.** Remaining action: confirm the GHCR `ucore-hci` image bakes ≥ 580.159.03 (lts-20260511 ≈ 580.159.04, so it should). The Jan 2026 advisory `a_id/5747` still applies as the older floor. Source: https://nvidia.custhelp.com/app/answers/detail/a_id/5821, https://www.gamingonlinux.com/2026/05/nvidia-reveal-more-gpu-driver-security-flaws-for-may-2026/.
- **VM-gating impact:** None — `modprobe.d` blacklist + `softdep nvidia pre: vfio-pci` pattern is unchanged. NVIDIA 595 keeps `nvidia-drm.ko modeset=1` default. **Avoid kernel 7.0 on any Blackwell host.**
- **Project status:** Project targets RTX 4090 — passthrough remains stable. RTX 50 upgrade should be deferred (see §9). Project pin floor should track 580.159.04 (LTS) or 595.71.05 (feature).
- Source: https://github.com/NVIDIA/open-gpu-kernel-modules/releases, https://nvidia.custhelp.com/app/answers/detail/a_id/5747.

### 8.2 NVIDIA Container Toolkit + CDI
*Recorded 2026-05-11.*

- **Latest:** **`v1.19.1` (2026-05-21)** (updated 2026-05-22; was v1.19.0, 2026-03-12). v1.19.1 CDI-relevant changes: egl-wayland2 lib/config now discovered + added to CDI specs; MIG `/dev/dri*` node injection; `nvidia-cdi-refresh` systemd condition fix (WSL2); new `disable-ipc-discoverer` nvcdi flag. No read-only-rootfs-specific change, but the CDI-generation tweaks are the relevant surface for bootc — low-priority version-bump candidate.
- **Changes:**
  - **v1.18.0** — CDI is now the default runtime mode (legacy mode demoted). Added `nvidia-cdi-refresh.service` systemd unit that auto-regenerates `/var/run/cdi/nvidia.yaml` on install/upgrade/driver-change.
  - **v1.19.0** — improved triggering of `nvidia-cdi-refresh`, **read-only root filesystem support** (initramfs / **bootc** ✓).
- **Open bugs:**
  - #1735 — `nvidia-cdi-refresh.service` ordering constraint can stall boot.
  - #1740 — non-privileged `MIG_STRATEGY=mixed` fails on 1.18+/1.19.0 due to missing cgroup access for `nvidia-cap1/2`.
- **bootc quirk:** `nvidia-ctk cdi generate` must write to `/var/run/cdi/` (tmpfs) not `/etc/cdi/`. v1.19 ro-rootfs support means the systemd unit handles this correctly now.
- **Project status:** `automation/45-nvidia-cdi-refresh.sh` exists — verify it targets the v1.19 layout (not the pre-1.18 manual `ExecStartPre` workaround).

---

## 9. VFIO/IOMMU + Looking Glass + KVMFR + QEMU + libvirt

### 9.1 VFIO / IOMMU / RTX 50-series passthrough
*Recorded 2026-05-11.*

- **State:** **RTX 5090 / RTX PRO 6000 passthrough is broken** as of May 2026 — confirmed reproducible reset bug. Acknowledged by NVIDIA, no fix. RTX 4090 is **unaffected** (project's target — supported).
- **Symptoms (RTX 50 series only):**
  - FLR fails after guest shutdown → `not ready 65535ms after FLR; giving up` → host requires power-cycle.
  - D3cold → D0 transition triggers CPU soft lockup post-shutdown.
  - Blackwell GPUs set PCIe config flag requesting IOMMU 1:1 identity mapping; **kernel 6.17 rejects device config when `iommu=pt` is on cmdline** — use `iommu=on` (DMA mode) for RTX 50.
  - ASUS X870E (the 9950X3D platform!): FLR causes permanent x8 bifurcation until cold boot. **Worth verifying on MiOS target hardware even with RTX 4090.**
- **Mitigations:**
  - `pcie_aspm=off disable_idle_d3=1` on kernel cmdline.
  - Early vfio-pci bind (`softdep nvidia pre: vfio-pci`) — still the correct VM-gating pattern.
  - `nvidia,reset-method=` quirks NOT helpful for Blackwell. `vendor-reset` is AMD-only.
- **Project posture:** Stays on RTX 4090. **Avoid kernel 7.0 with any NVIDIA Blackwell.** Project kargs.d (`00-mios.toml`, `20-vfio.toml`, `13-rtx50-vfio-workaround.toml`) already encode `iommu=pt` + AMD passthrough.

### 9.2 Looking Glass
*Recorded 2026-05-11.* *(updated 2026-05-19: master cadence has dropped substantially — no visible master commits between 2026-01-17 and 2026-05-19 (4-month gap, confirmed across two probes). Possible upstream stall. Latest visible work: Janrupf wayland discrete-scroll fix + AUTHORS update on 2026-01-17.)* *(updated 2026-05-22: **stall confirmed, now ~4 months.** Master still at the 2026-01-17 commit; `module/` still at 2025-03-05 ("[doc] update copyright year", preceded by the 2025-03-04 `MODULE_IMPORT_NS` string-literal fix). Still B7, no B8/RC. The kernel ≥6.13 KVMFR build patch still rides on that single 2025-03-04 commit — nothing new to track for the eventual 6.12→6.18 base bump.)*

- **Latest stable:** **B7** (2025-03-06) — now 14+ months stale on tagged releases. **No B8 announced.** Cadence is slow (B6→B7 took ~2 years).
- **Recent (post-B7 git-master):** Wayland clipboard crash fix, Wayland protocol error on capture-mode toggle, libdecor builds for GNOME Wayland window decorations. (No fresh commits since 2026-01-17.)
- **Wayland client:** **Feature parity with X11 in B7** — scaling, fullscreen, clipboard, cursor. Build with `-DENABLE_WAYLAND=ON -DENABLE_X11=OFF` to drop X11 deps on bootc.
- **Project status:** `automation/53-bake-lookingglass-client.sh` exists.

### 9.3 KVMFR (`/dev/kvmfr0`)
*Recorded 2026-05-11.* *(updated 2026-05-16: kernel ≥6.13 build patches noted.)*

- **Source:** Still **DKMS-only**; not in mainline, no submission planned. Lives in LookingGlass repo (`module/`).
- **Secure Boot signing:** Required — Fedora bootc will refuse unsigned `kvmfr.ko`.
  - On immutable rootfs, **signing must happen at image build time** in the Containerfile, not first-boot (DKMS auto-sign assumes mutable rootfs).
  - `/etc/dkms/framework.conf` needs `mok_signing_key=` and `mok_certificate=` for auto-sign on rebuild.
- **Setup:** sysfs-based — `kvmfr.static_size_mb=128` modprobe option; udev rule for `/dev/kvmfr0` owner/group/mode. Older sysconfig approach is deprecated in B7 docs.
- **Kernel compat patches (updated 2026-05-16):** Community patches required to build kvmfr against **kernel ≥6.13** — add `#include <linux/vmalloc.h>` and `MODULE_IMPORT_NS("DMA_BUF")` in `module/module.c`. No upstream submission. Once ucore-hci LTS image migrates from 6.12 → 6.18 (issue #362), these patches must be applied before the rebuild succeeds. Source: https://forums.gentoo.org/viewtopic-t-1176809.html. (updated 2026-05-19: confirmed via LookingGlass repo — last `module/` commit is 2025-03-04 "MODULE_IMPORT_NS now requires a string literal in 6.13" — that single patch handles the kernel 6.13 build break within LookingGlass's own tree. No upstreaming to mainline Linux visible.)
- **Project status:** `automation/52-bake-kvmfr.sh` exists.

### 9.4 QEMU
*Recorded 2026-05-11.* *(updated 2026-05-18: bootstrap baseline 10.2.0 was stale — current latest is **11.0.0 (2026-04-22)**; 11.0 had already shipped before the bootstrap pass and was missed.)* *(updated 2026-05-26: **v11.0.1 tagged 2026-05-25** — first 11.0.x point release, alongside stable backports **v10.2.3** and **v10.0.10** the same day. Lands during the publicly-disclosed CXL "QEMUtiny" window (oss-sec, ~2026-05-11): chained OOB-read in `GET_LOG` + OOB-write in `SET_FEATURE` in **CXL Type-3 device emulation** (the vulnerable `SET_FEATURE` path was introduced in 11.0.0). **Low practical exposure for MiOS:** CXL Type-3 memory-expander emulation is not part of a VFIO GPU-passthrough VM topology — MiOS does not emulate CXL devices, so the QEMUtiny surface is not reached by the project's libvirt domains. No CVE-ID in the tag metadata yet; qemu.org/blog has not posted the 11.0.1 announcement (normal lag). Treat 11.0.1 as the likely security/stable fix and let it arrive via a base rebuild — QEMU is a Fedora/ucore base package, not a project-owned pin. No 11.1/11.1-rc.)*

- **Latest stable:** **11.0.1** (2026-05-25 — first 11.0.x point release; likely the CXL "QEMUtiny" security/stable fix, see header). **11.0.0** (2026-04-22) — 2500+ commits, 237 authors, new **Nitro Enclaves accelerator**. Stable backports **10.2.3** + **10.0.10** also tagged 2026-05-25. 10.2.0 (2025-12-24), 10.1.0 (2025-08-26), 10.0.0 (2025-04). No 11.1/11.1-rc yet.
- **Recent line (consolidated):**
  - **11.0** — Nitro Enclaves; further VFIO + confidential-guest plumbing; large cross-architecture refactor (verify VFIO/PCI passthrough behavior on the 9950X3D + RTX 4090 path before bumping).
  - 10.2 — **live update via `cpr-exec` migration mode** (in-place upgrade without VM downtime), 9pfs FreeBSD host support, io_uring perf path.
  - 10.1 — **VFIO `guest_memfd` support** for confidential guests (SEV-SNP / TDX passthrough).
  - 10.0 — virtio-scsi multiqueue; new Apple graphics devices.
- **virtiofsd** now external (Rust); libvirt `<idmap>` element for unprivileged virtiofsd.
- **OVMF / swtpm:** No breaking changes.
- Source: https://www.qemu.org/blog/.

### 9.5 libvirt
*Recorded 2026-05-11.* *(updated 2026-05-18: bootstrap baseline 12.1.0 was stale — current latest is **12.3.0 (2026-05-02)**; 12.2.0 (2026-04-01) was also missed.)*

- **Latest released:** **`12.3.0` (2026-05-02)** — adds **bhyve I/O throttling** and **Hyper-V guest-info APIs**. `12.2.0` (2026-04-01), `12.1.0` (2026-03-29), `12.0.0` (2026-01-15). (updated 2026-05-22: **`12.4.0` is listed "unreleased"/in-progress** on the news page — not yet shipped. No new VFIO/PCI-passthrough XML schema changes noted.)
- Recent fixes: AppArmor + snapshot interaction (12.1); dynamic `$PATH` lookup for helpers (12.0); POWER11 CPU support (12.0). Fixed 11.2/11.3 regression (internal snapshot revert broken; post-copy migration crash on destination).
- **virt-manager:** `virt-convert` removed (use `virt-v2v`).
- No VFIO/PCI passthrough XML schema changes through 12.3.0.
- Source: https://libvirt.org/news.html.

---

## 10. Gamescope + Waydroid + Mesa/ROCm

### 10.1 Gamescope
*Recorded 2026-05-11.* *(updated 2026-05-16: 3.16.21/22/23 point releases noted; 3.17 still not tagged.)* *(updated 2026-05-22: no change — still **3.16.23 (2026-04-07)** latest, no 3.16.24/3.17. HDR fix `7d4e835` still master-only. **Issue #2018 still OPEN with no maintainer (Ashton/misyltoad) response, no assignee, no linked PR.** Hold HDR rollout.)* *(updated 2026-05-26: **correction — `3.16.24` was tagged 2026-05-21** (the 2026-05-24 pass missed it and recorded "3.16.23 latest, no 3.16.24"). 3.16.24 is a single-line changelog: "Revert 'build: add workaround to build with CMake 4.0'" (OpenVR 2.12.1 made the hack unnecessary) — **NOT a graphics/HDR change.** HDR fix `7d4e835` remains master-only/unreleased; issue #2018 still open, no maintainer ack, no linked PR. No 3.17. The "hold HDR rollout" stance is unchanged.)*

- **Latest:** **3.16.24** (2026-05-21 — CMake-4.0-workaround revert, no graphics change). Prior points: 3.16.21 (2026-03-12), 3.16.22 (2026-03-15), 3.16.23 (2026-04-07). **No 3.17 cut yet.** The HDR fix `7d4e835` is in none of these tags (still master-only).
- **Open regressions (still open as of 2026-05-16):**
  - HDR regression on Fedora 43 / GNOME 49 / KDE Plasma 6.5.3 (issue #2018 — still open).
  - Wayland NVIDIA HDR (issue #2037 — reporter cites upstream commit `7d4e835` as the fix, but no tagged release yet contains it).
  - NVIDIA + Plasma 6.5 Wayland HDR produces grey/washed image (#2000).
- **HDR pipeline on NVIDIA 595 + gamescope 3.16.x has color-correctness regressions** — **hold HDR rollout until next gamescope minor tag.**
- `--expose-wayland` flag still required for native Wayland clients inside Gamescope. HDR requires compositor with `xx-color-management-v4` or `frog-color-management-v1`.
- Source: https://github.com/ValveSoftware/gamescope/tags, https://github.com/ValveSoftware/gamescope/issues/2018, https://github.com/ValveSoftware/gamescope/issues/2037.

### 10.2 Waydroid
*Recorded 2026-05-11.*

- **Latest:** `1.6.2` (Feb 2026). Added **Vulkan support for Intel `xe` driver**.
- **Kernel ABI:**
  - `ashmem` — no longer needed since 1.2.1 (replaced by `memfd` in mainline ≥ 5.18).
  - **`binder` now ships as Rust module (`rust_binder`)** in mainline `linux`/`linux-zen`. Fedora's kernel does **NOT** ship `rust_binder` enabled by default — still need DKMS `binder_linux` for Fedora bootc base. **Must be Secure-Boot-signed.**
- **Android image:** No public confirmation of Android 14/15 bump — Android 13 still default.
- **NVIDIA story:** Still broken-by-default; two workarounds (LXC GPU passthrough via `/dev/nvidia*` nodes + software rendering fallback). Anecdotal reports of unmodified boot on recent driver/Waydroid combos.

### 10.3 Mesa / ROCm
*Recorded 2026-05-11.* *(updated 2026-05-18: Mesa bootstrap baseline 25.3.4 was stale — current latest is **26.1.0 (2026-05-06)**; 25.3.6 (2026-02-19) was the last 25.3.x point.)* *(updated 2026-05-19: stable-branch point release **26.0.7 (2026-05-14)** confirmed via the release-notes index plus the 26.0.7 page itself. 26.0.x is the older stable branch; 26.1.x is the newer feature branch. No 26.1.1 yet (26.1.0 from 2026-05-06 still latest on the .1 branch).)* *(updated 2026-05-22: **26.1.1 has now shipped** on the feature branch; 26.0.7 remains the latest 26.0 stable backport (no 26.0.8 yet). Neither is security/build-breaking.)*

- **Mesa:** **`26.1.1`** is the current feature-branch latest (2026-05-06's 26.1.0 superseded); **`26.0.7` (2026-05-14)** remains the latest backport on the 26.0 maintenance branch (no 26.0.8 yet). `25.3.6` (2026-02-19) was final on the 25.3 series. `25.1` in Fedora 43 mainline (verify whether F43/44 packages have moved to the 26.x series yet — `ucore-hci:stable-nvidia-lts` may still ship the 25.x line). RDNA4 ray-tracing optimization, triangle pair compression (GFX12). RX 9000-series stable since 25.1.3 emergency patch.
- **ROCm:** **`7.2.3` (2026-05-04)** — confirmed current latest (no change in window). RX 9070 XT works on Fedora 43 + kernel 6.17+.
- No VFIO host-path deprecations.
- Source: https://docs.mesa3d.org/relnotes.html, https://github.com/ROCm/ROCm/releases.

---

## 11. FreeIPA/SSSD + GNOME + WSL2

### 11.1 FreeIPA + SSSD
*Recorded 2026-05-11.*

- **FreeIPA:** **4.13.0** stable (>170 fixes since 4.12.5). Beta of new responsive WebUI.
- **SSSD:** **2.13.0**. **2.11** introduced generic IdP backend (Keycloak + Entra ID via OAuth 2.0 Device Authorization, RFC 8628).
- **FIDO2 passkey auth** for centrally-managed users continues to stabilize on Fedora — works for sudo and SSH via SSSD PAM stack; physical FIDO2 devices only (no platform passkeys yet).
- No fresh 2026 CVEs.
- **Project status:** `automation/22-freeipa-client.sh` exists.

### 11.2 GNOME
*Recorded 2026-05-11.* *(updated 2026-05-18: GNOME 50.2 stable point release expected **2026-05-23** (5 days from now); GNOME 51.alpha date **confirmed** at 2026-06-27 per `release.gnome.org/calendar`.)* *(updated 2026-05-22: **50.2 not yet shipped** — tarballs due 2026-05-23 (tomorrow). No 50.2 changelog to inspect yet; re-check tomorrow. 51.alpha still 2026-06-27 (50.3 same day). The Wayland/NVIDIA explicit-sync + HDR-metadata + Remote-Desktop work the project tracks already landed in **GNOME 50.0 / Mutter 50** — nothing indicates a 50.2-specific graphics fix pending.)* *(updated 2026-05-24: **50.2 did NOT ship on 2026-05-23 — that was the tarballs-due date, not the release date.** GNOME Discourse (responsible: avovk) states 50.2 stable lands "the following week"; `download.gnome.org/core/50/50.2/` and its NEWS path both 404 (dir not created) — release expected ~week of 2026-05-25/26. Remains bugfix-only as predicted. 50.x cadence confirmed: 50.3 = 2026-06-27, 50.4 = 2026-08-01, 50.5 = 2026-09-12, 50.6 = 2026-10-10. 51.alpha still 2026-06-27.)* *(updated 2026-05-26: **still NOT released.** `download.gnome.org/core/50/50.2/` still does not resolve; the releng point-release NEWS path (`download.gnome.org/teams/releng/`) shows **50.1 as the latest**, no `50.2/` dir. The 2026-05-23 tarballs-due deadline has passed but the wrapped release/announcement has not appeared; no "50.2 released" coverage anywhere. Expected within days, bugfix-only — nothing yet to read for Mutter / gnome-remote-desktop / NVIDIA explicit-sync / HDR. 50.3 + 51.alpha both 2026-06-27 unchanged.)*

- **Latest stable: GNOME 50 "Tokyo"** (released **2026-03-18**). **GNOME 50.2 stable point release: 2026-05-23.** GNOME 51 unstable/dev (alpha **2026-06-27** — confirmed, final 2026-09-16). GNOME 49 (Sep 2025) is the old-stable.
- **GNOME 50 is fully Wayland-only.** Mutter, gnome-shell, gnome-session, Control Center had X11 backends **REMOVED** (~27.5k LOC dropped). XWayland remains.
- **NVIDIA explicit-sync** (`linux-drm-syncobj-v1`) mature across Mutter/Mesa/EGL-Wayland.
- VRR enabled on compatible monitors. HDR screen sharing in RDP.
- **GNOME Remote Desktop:** RDP backend with GDM headless login (relevant to MiOS `mios-grd-setup`); HDR screen-share added in 50 RC.
- **Breaking:** X11 session gone. Any X11 fallback in ucore-hci/MiOS profile should be removed.

### 11.3 WSL2 + bootc
*Recorded 2026-05-11.* *(updated 2026-05-16: WSL 2.7.5 pre-release with kernel 6.18.26.1 shipped 2026-05-15; 2.7.4 was skipped.)* *(updated 2026-05-19: **two new releases** — **2.7.6 stable (2026-05-18)** picks up the fix for GUI app icons disappearing from Start menu on Azure Linux 3 system distros; **2.8.6 pre-release (2026-05-14)** adds container list filters. WSL now runs parallel **2.7.x stable + 2.8.x pre-release** trains — cadence shift since the last pass.)* *(updated 2026-05-22: **stable advanced to 2.7.7 (2026-05-19)**, superseding 2.7.6. Pre-release train: 2.8.6 (2026-05-14) still newest 2.8.x.)*

- **Latest stable: WSL 2.7.7** (2026-05-19; superseded 2.7.6 from 2026-05-18). **Latest pre-release: WSL 2.8.6** (2026-05-14, on the new 2.8.x pre-release train). **2.7.5** (2026-05-15) was the previous pre-release — **kernel 6.18.26.1**, skipped 2.7.4. **2.7.3** (2026-04-25) was the prior pre-release. **2.6 stable** is the first **open-source** release (MIT-licensed). 2.5.6 is the conservative stable channel.
- **2.7.x:** VirtIO networking IPv6, DNS tunneling, statx in VirtioFS, directory mounting; .NET bumped for **CVE-2026-26127** mitigation.
- `wsl --import` + `.wsl` tarball install (since 2.4.4) is the practical path for Fedora bootc images.
- systemd-in-WSL via `/etc/wsl.conf` `[boot] systemd=true` is mature.
- **No first-class bootc-aware path** in microsoft/WSL — you import a bootc-rendered rootfs tarball; `bootc upgrade` works inside, but kernel comes from WSL host. **Kernel 6.18 on WSL 2.7.5 inherits the May 2026 DRM CVE cluster fixes (CVE-2026-43398/43300/43287)** — see §6.5.
- Source: https://github.com/microsoft/WSL/releases.

---

## 12. kargs.d + Renovate + systemd-sysext + tmpfiles + bootc lifecycle

### 12.1 kargs.d
*Recorded 2026-05-11.*

- **bootc v1.15.2 kargs.d format is unchanged from v1.13 baseline.**
- **Canonical keys: `kargs = [...]` (flat array) and optional `match-architectures = [...]`** (Rust target arch names — mind `x86_64` vs `amd64`, `powerpc64` vs `ppc64le`).
- **No `match-platforms` key. No `priority` key. No `[kargs]` table headers. No `append`/`delete` keys.** Searched issues/PRs, official docs, v1.15.x notes — none have landed.
- **Open RFE:** Issue #899 requests `/etc/bootc/kargs.d` merge in addition to `/usr/lib/bootc/kargs.d` — not yet shipped.
- v1.15.2 adds `discoverable-partitions` and a container-signature-policy install knob (unrelated to kargs).
- **Project status:** `usr/lib/bootc/kargs.d/*.toml` files follow the flat-array rule correctly. `00-mios.toml` header reaffirms it. **Honor strictly when editing.**

### 12.2 Renovate
*Recorded 2026-05-11.* *(updated 2026-05-16: now v43.181.0.)* *(updated 2026-05-18: now v43.182.4 — 8 more releases in the 2-day window: 43.181.1 → 43.181.2 → 43.182.0–43.182.4. All feature/bugfix; no security advisories.)* *(updated 2026-05-19: now **v43.185.1** — 4 releases in the 24h window: 43.183.0 / 43.184.0 / 43.185.0 all on 2026-05-18, then 43.185.1 today. 43.185.1 is a GitHub tags datasource bugfix (now uses `published_at`). No security advisories.)* *(updated 2026-05-22: now **v43.192.0 (2026-05-22)** — 7 routine patch/minor releases since 43.185.1, no security advisories, config compatibility unchanged.)*

- **Renovate:** **v43.195.8** (2026-05-26 — updated 2026-05-26, was 43.195.0; eight routine patch releases 43.195.1→.8 in the window). Routine cadence — no security advisories surfaced. Bootstrap baseline (43.173.0) is now ~22 minor versions behind, but config compatibility unchanged.
- v43 stream: patch-heavy, GitHub noreply email handling for GHE Cloud, GitLab merge-trains MR support, dryRun fixes.
- v41 added JSONC support in configs/presets and Merge Confidence badges by default in `config:recommended`.
- `customManagers` (renamed from `regexManagers`) — automerge enabled by combining `matchManagers`, `matchDatasources`, and `matchUpdateTypes: ["digest", ...]`.
- `docker:pinDigests` + `config:best-practices` still the recommended composition. **Project's `renovate.json` is current — no config drift needed.**
- **`platformAutomerge: false`** still requires two Renovate runs before merge — keep enabled for safe digest pins.
- Caveat: v41 changed branch-name composition for `separateMultipleMinor=true` with customized `branchTopic` — project does not customize `branchTopic`, so not affected.

### 12.3 systemd-sysext
*Recorded 2026-05-11.* *(updated 2026-05-16: systemd 260 stable already shipped 2026-03-17 — bootstrap doc said "in development"; that was wrong.)*

- **systemd 260 stable shipped 2026-03-17** (drops SysV, adds mstack, Varlink metrics). systemd 259.5 latest in 259 series. systemd 258 (Sep 2025) introduced major sysext improvements. *(updated 2026-05-24: **v261-rc1 cut 2026-05-22** — pre-release/unstable (Live Update Orchestration + new Varlink interfaces); stable track unchanged at v260.1 (2026-03-23), no action for the project's stable base.)*
- v258 — sysext respects `ID_LIKE=` from os-release (broader cross-derivative use).
- v259 — `/etc/systemd/systemd-sysext.conf` and `/etc/systemd/systemd-confext.conf` config files; image-policy and mutability configurable centrally. `--mutable=help` lists modes. overlayfs mount options via `$SYSTEMD_SYSEXT_OVERLAYFS_MOUNT_OPTIONS` / `$SYSTEMD_CONFEXT_OVERLAYFS_MOUNT_OPTIONS`. `systemd-stub` loads global sysexts/confexts from `ESP/loader/extensions/*.{sysext,confext}.raw` — relevant if MiOS adopts UKIs.
- **bootc + sysext integration** (sysexts as separate OCI tags managed in lockstep) is still **WIP** — not production-blessed.
- **Project status:** Containerfile lines 112–115 use `mios-sysext-pack.sh` to consolidate sysexts into `/usr/lib/extensions/source` — fine. No format break.

### 12.4 tmpfiles.d / sysusers.d
*Recorded 2026-05-11.*

- Stable. systemd 258/259 ship existing directives unchanged.
- Background: `sysusers.d/systemd-imdsd.service` (2026-03-26), `tmpfiles.d/root.conf` meson plumbing (2026-04-09).
- **CVE-2026-3888** — snap-confine + systemd-tmpfiles interaction → local privilege escalation to root (CVSS 7.8). Not a direct systemd bug — leveraged by snap-confine. MiOS does not ship snapd, so direct exposure is nil. Mentioned only as a general reminder to never create world-writable trees in user-controlled paths via tmpfiles.

### 12.5 bootc switch / upgrade / rollback + greenboot
*Recorded 2026-05-11.*

- `bootc switch` and `bootc upgrade` are semantically equivalent except switch changes the tracked image ref; both preserve `/etc` and `/var`.
- Blue/green deployments: management agent calls `bootc switch` (or declarative `bootc edit`).
- **`bootc rollback`** reorders bootloader entries; **changes to `/etc` do NOT carry into the rolled-back deployment** — they revert to that deployment's state.
- **`greenboot-rs`** (Rust rewrite) is the current Fedora track for boot-health verification. Required vs. wanted health-check scripts; failed required checks trigger reboot + auto-rollback.
- **Sharp edge:** Issue #946 — rollback after a `bootc switch` can leave the rollback deployment as default; verify before relying on it as Day-2 recovery contract.
- **Project status:** `automation/46-greenboot.sh` exists.

### 12.6 bootc CI build (image-builder action + cosign + buildkit)
*Recorded 2026-05-11.*

- **Upstream action:** **`osbuild/bootc-image-builder-action`** (ublue-os's wrapper is in maintenance — migrate).
- bootc-image-builder needs `--privileged` + `--security-opt label=type:unconfined_t`. GitHub-hosted runners grant that for a Docker-based privileged container, **but rootful BuildKit (`docker buildx` with `--privileged` worker) is the missing piece for LBI pre-pull** — this matches the project's reason for disabling it in the Containerfile.
- **cosign-installer@v3** is the current action; pair with `actions/attest-build-provenance` for SLSA. **cosign v3 requires `--bundle` in some flows.**

---

## Closing note

This file was bootstrapped 2026-05-11. Subsequent passes should **edit in place** — find the relevant subsection, update the data point, and add `(updated YYYY-MM-DD: <reason>)` inline. Use new dated subsections under a topic when a finding doesn't replace existing material. Remove proven-false content and note the removal in `ai-journal.md`.

**Iterations:**
- 2026-05-11 — bootstrap, 12 topic groups (`scheduled-research-daily`).
- 2026-05-16 — daily pass: NVIDIA LTS floor bumped (580.126.20 → 580.159.04), Podman 6.0 GA imminent, `image-builder-cli` v64 GA-ed bootc subcommand, K3s v1.34.8-rc1 + v1.35.5-rc1 + v1.36.0, etcd 3.5.30, WSL 2.7.5, Renovate 43.181.0, systemd 260 (correction — already shipped 2026-03-17), bsherman/ucore-hci now 404, CVE-2026-31431 reclassified to Linux kernel (corrected mis-attribution in §7.1), §6.5 added for May 2026 kernel CVE cluster.
- 2026-05-18 — daily pass: **Podman 6.0 GA SLIPPED to F45** (was "week of 2026-05-25" — Fedora change retargeted, no RC tag cut); kernel CVE cluster expanded from 5 → ~12 (AMDGPU cluster CVE-2026-43318/43305/43398/43400/43298/43237/43320 + Fragnesia CVE-2026-46300); kernel 6.18.28/6.18.30 and 6.12.87/6.12.88 noted as backport carriers; ucore-hci daily-rebuild cadence appears stalled (last tag 2026-05-11, 7 days old) — issues #385 + #362 still open; NVIDIA 595.44.08 Vulkan dev-beta (2026-05-15) added with branch-split clarification; stale baselines corrected — Mesa 25.3.4 → 26.1.0, QEMU 10.2.0 → 11.0.0, libvirt 12.1.0 → 12.3.0, fapolicyd 1.3.8 → 1.4.5; CrowdSec 1.7.8 (2026-05-11) added; Pacemaker 3.0.2-rc2 (2026-05-11) added; GNOME 50.2 (2026-05-23) noted; Renovate 43.182.4 (2026-05-18); K3s v1.34.8 GA still pending with CVE-2026-33186 grpc-go bump still uncalled-out; Secure Boot shim-16.1-6 unchanged in F44 stable, next checkpoint 2026-06-05; image-builder-cli format parity softened (only qcow2 + bootc-installer ISO documented).
- 2026-05-19 — daily pass: **CVE-2026-31431 confirmed on CISA KEV with federal remediation deadline 2026-05-15 — now 4 days past** (NVD updated 2026-05-18); MiOS LTS base image still ships pre-fix kernel. **NEW ucore PR #392** opened 2026-05-17 by dylanmtaylor — F43 → F44 base migration (implicit kernel-bump path); issue #385 updated 2026-05-17 with 7 new comments. ucore main idle 12 days; ucore-hci tag cadence stalled 8 days. **GHSA-w6c6-c85g-mmv6 got CVE assignment CVE-2026-39395** (CVSS 4.3, `verify-blob-attestation` false positive — already fixed in cosign 3.0.6). Upstream changes: **Mesa 26.0.7** stable backport (2026-05-14); **WSL 2.7.6 stable** (2026-05-18) + **2.8.6 pre-release** (2026-05-14, new parallel pre-release train); **Renovate 43.185.1** (2026-05-19, 4 releases in 24h). fapolicyd date corrected (v1.4.5 = 2026-03-30, not 2025-03-30 — atom feed authoritative). Corosync corrected (v3.1.10 (2024-11-15), not 3.1.1). F45 Beta confirmed **2026-08-25** via Wikipedia+ChangeSet cross-ref; F45 Atomic Desktops will switch ISO builds from lorax to image-builder, but **composefs+UKI sealed-image is NOT confirmed as a default F45 deliverable** (continues as travier/fedora-atomic-desktops-sealed WIP). LookingGlass master appears stalled — no commits since 2026-01-17 (4-month gap). Pacemaker 3.0.2 final not yet shipped (rc2 8 days old). Anubis-gating now affects bodhi + koji + src.fedoraproject.org + Fedora Discussion search — established `dl.fedoraproject.org/pub/fedora/linux/updates/44/...` as the working alternate path for shim-16.1-6 promotion checks.
- 2026-05-22 — daily pass: **folded in the 2026-05-20 findings that never reached the doc (that pass committed only NEXT-RESEARCH before a git-receive-pack 503) plus fresh 05-22 deltas.** **Kernel local-root cluster grew to FIVE named vectors** — added `CVE-2026-31635` "DirtyDecrypt" (RxGK LPE, CONFIG_RXGK=Fedora, public PoC 05-18), `CVE-2026-46333` "ssh-keysign-pwn" (ptrace exit-race → LPE, steals SSH host keys + /etc/shadow), and `CVE-2026-43500` (new RxRPC page-cache-write LPE) now grouped with CVE-2026-43284 + CVE-2026-46300 under Red Hat **RHSB-2026-003 "Dirty Frag"** (§6.5 + action item 8). kernel.org floors recorded (7.0.9 / 6.18.32 / 6.12.90). **NVIDIA May 2026 bulletin `a_id/5821` confirmed PUBLISHED** (2026-05-19; 13 CVEs, top CVE-2026-24187 CVSS 8.8) — corrects the prior "no May bulletin" line; **project pins satisfy all floors, no bump forced** (§8.1, action item 3). **K3s GA shipped 2026-05-20** (v1.36.1/v1.35.5/v1.34.8) — **CVE-2026-33186 RESOLVED** (go.mod `replace grpc => v1.79.3`, notes silent); Go-version correction (1.26.2, not 1.25.9) (§5.1). **fapolicyd v1.5** (2026-05-20, minor jump — validate rule-reload) (§6.2). Routine bumps: image-builder-cli v65, NVIDIA Container Toolkit v1.19.1, Mesa 26.1.1, WSL 2.7.7, Renovate 43.192.0, etcd 3.7.0-beta.0 (pre-release), libvirt 12.4.0 (in-progress). **Secure Boot: shim-16.1-6 STILL absent from all of F44** (stable/updates/updates-testing — verified via mirrors.kernel.org curl; ~35 days to 2026-06-26 cutover). **All ucore remediation levers still frozen** — PR #392 open, issue #385 open, main idle 15 days, GHCR stalled 11 days. NO CHANGE: cosign 3.0.6, bootc 1.15.2, OSTree 2026.1, composefs 1.0.8 (no v1.1), Pacemaker 3.0.2-rc2, Podman 5.8.2 (no 6.0 RC), Ceph 20.2.1, CrowdSec 1.7.8, systemd 260.1, QEMU 11.0.0, Looking Glass B7 (master idle ~4 months), Gamescope 3.16.23 (#2018 HDR still open), Waydroid 1.6.2, ROCm 7.2.3. GNOME 50.2 due 2026-05-23 (not yet shipped).
- 2026-05-24 — daily pass: **HEADLINE — `ublue-os/ucore` PR #392 MERGED 2026-05-24** (commit `412e7be`, approved @bsherman, 73 checks green) — migrates LTS image kernel flavor **longterm-6.12 → longterm-6.18 on the F44 base** (also effectively resolves issue #362); ucore main active again (idle gap broken); **issue #385 NOT auto-closed (still open)**; **no rebuilt `ucore-hci` GHCR tag confirmed yet** (the bake is the gate for the kernel-CVE fix reaching MiOS). Items 1/8 + §2 + §6.5 updated. **DirtyDecrypt (CVE-2026-31635) date conflict RESOLVED** — fixed 2026-04-25 upstream, affected ranges 6.16.1–6.18.22 / 6.19.0–6.19.12; **F44 `updates` now ships `kernel-7.0.9-205.fc44`** (past all affected ranges); the new 6.18 LTS image must bake ≥6.18.23 (DirtyDecrypt) / ≥6.18.31 (ssh-keysign-pwn) — current longterm-6.18 = 6.18.33. **kernel.org bumped 2026-05-23**: 7.0.10 / 6.18.33 / 6.12.91 / 6.6.141 / 6.1.174. Fragnesia (CVE-2026-46300) confirmed a regression from the CVE-2026-43284 fix. **Secure Boot shim-16.1-6 STILL absent** from all F44 trees (re-verified; ~33 days to 2026-06-26). Minor bumps: **NVIDIA dev-beta 595.44.09** (2026-05-22, not a pin); **Renovate 43.195.0**; **systemd v261-rc1** (2026-05-22, unstable — stable v260.1 unaffected). **GNOME 50.2 did NOT ship** (05-23 was tarballs-due; release ~week of 05-25). NO CHANGE: cosign 3.0.6, bootc 1.15.2, OSTree 2026.1, composefs 1.0.8 (no v1.1; bootc native backend still "Experimental"), image-builder-cli v65 (no v66; issue #506 still open; osbuild.org/blog still 404), Podman 5.8.2 (no 6.0 RC), K3s GA tags (05-20), etcd 3.6.11/3.5.30/3.4.44 (3.7.0-beta.0), Ceph 20.2.1, Pacemaker 3.0.2-rc2, Corosync 3.1.10, fapolicyd 1.5, CrowdSec 1.7.8, usbguard 1.1.4, QEMU 11.0.0, libvirt 12.3.0 (12.4.0 still unreleased), Mesa 26.1.1/26.0.7, ROCm 7.2.3, NVIDIA Container Toolkit 1.19.1, Waydroid 1.6.2, WSL 2.7.7/2.8.6, FreeIPA 4.13.0/SSSD 2.13.0, Looking Glass B7 (master idle ~4 months), Gamescope 3.16.23 (#2018 still open/unacked), RTX 50-series passthrough still broken, F45 Beta 2026-08-25, NVIDIA LTS 580.159.04 / feature 595.71.05.
- 2026-05-26 — daily pass: **HEADLINE — the kernel-CVE remediation gate is CLOSED.** A fresh **`ucore-hci:stable-nvidia-lts-20260526`** bake now exists (built 2026-05-26T04:19Z; manifest-probe: `-20260526`/`-20260525` = 200, `-20260527` = 404) on the **post-#392-merge base** (image revision `e63e21f` = ucore commit #397 "use cosign login instead of docker login", 2026-05-25; FCOS `44.20260419.3.1` stable; longterm-6.18 = 6.18.33 ≥ all cluster floors). The 11–15-day bake stall is broken; ucore main active again. The fix is now **deliverable** to MiOS (Renovate digest bump or `:stable-nvidia-lts` migration) but not yet *delivered*; the project still pins the non-LTS rolling `:stable-nvidia`. **Issue #385 still open.** Items 1/8, §2, §6.5 updated. Other changes: **QEMU v11.0.1 (2026-05-25)** — first 11.0.x point, likely the CXL "QEMUtiny" security/stable fix (CXL Type-3 emulation not in a GPU-passthrough VM topology → low MiOS exposure; +v10.2.3/v10.0.10 backports); §9.4 updated. **Gamescope 3.16.24 (2026-05-21)** — correction, the 05-24 pass missed it; it's a CMake-workaround revert, **not** the HDR fix (7d4e835 still master-only, #2018 still open); §10.1 updated. **image-builder-cli v66 (2026-05-25)** — routine (profiling, images 0.267.0 w/ erofs/squashfs mount + skopeo container resolve; no bootc/composefs work; #506 still open); §1.3 updated. **Renovate v43.195.8 (2026-05-26)**; §12.2 updated. **kernel.org mainline → 7.1-rc5** (stable/longterm unchanged); §6.5/items 8. **Secure Boot shim-16.1-6 STILL absent** from all F44 trees (re-verified; ~31 days to cutover, ~10 days to the 2026-06-05 checkpoint — tightest unresolved risk); item 4 + §7.2 updated. **GNOME 50.2 still NOT shipped** (tarball deadline passed; releng path shows 50.1 latest); §11.2 updated. NO CHANGE: cosign 3.0.6, NVIDIA LTS 580.159.04 / feature 595.71.05 / dev-beta 595.44.09 (no June bulletin), bootc 1.15.2, OSTree 2026.1, composefs 1.0.8 (no v1.1; bootc native backend still verbatim "Experimental"), Podman 5.8.2 (no 6.0 RC), K3s GA tags (05-20), etcd 3.6.11/3.5.30/3.4.44 + 3.7.0-beta.0, Ceph 20.2.1, Pacemaker 3.0.2-rc2 (no rc3/final), Corosync 3.1.10, fapolicyd 1.5, CrowdSec 1.7.8, systemd 260.1 (v261-rc1 unstable), libvirt 12.3.0 (12.4.0 still unreleased), Mesa 26.1.1/26.0.7, ROCm 7.2.3, NVIDIA Container Toolkit 1.19.1, Waydroid 1.6.2, WSL 2.7.7/2.8.6, FreeIPA 4.13.0/SSSD 2.13.0, Looking Glass B7 (master + `module/` KVMFR idle ~4 months — still a forward risk for the 6.18 base), RTX 50-series passthrough still broken (RTX 4090 unaffected), F45 Beta 2026-08-25 (no composefs-sealed-image Change).
