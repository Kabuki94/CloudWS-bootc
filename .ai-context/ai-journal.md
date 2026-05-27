# AI Journal — CloudWS-bootc / MiOS

> Append-only journal of AI-agent activity touching this project. Each entry is timestamped (UTC), identifies the agent, and records what was investigated/changed/resolved/invalidated. Entries should be self-contained — a future agent reading any single entry should know what happened without prior context.

---

## 2026-05-11T11:30Z — `scheduled-research-daily` (BOOTSTRAP)

**Agent ID:** `scheduled-research-daily`
**Context:** First scheduled-research pass on this project. No prior `.ai-context/` directory, no prior journal, no prior `NEXT-RESEARCH.md`, no `CLAUDE.md`. Substituted `INDEX.md` + `ARCHITECTURE.md` + `ENGINEERING.md` + `SECURITY.md` for the missing CLAUDE.md "sections 1–3" the brief expected.

**Project state at time of run:**
- Version: `v0.1.4` (from `VERSION`).
- Base image: `ghcr.io/ublue-os/ucore-hci:stable-nvidia` (Containerfile line 19; digest pinned by Renovate in `image-versions.yml`).
- Most recent shipped commits: `136e6fc fix(build): resolve 01-repos.sh failure and version typo` (HEAD), `b181d95 fix(arch): align with FHS and zero-trust mandates`. v0.2.0 line abandoned in `024d5f1 reset.`
- No `docs/changelogs/` exists; no top-level `CHANGELOG.md`. Inferred recent direction from git log.

**Research scope (full baseline, since no prior agenda existed):**
Twelve topic groups — bootc/BIB/image-builder-cli; ucore-hci/Universal Blue; Fedora bootc/FCOS/composefs/OSTree; Podman/Quadlet/rechunk; K3s/Ceph/Pacemaker; CrowdSec/fapolicyd/usbguard/SELinux; cosign/Secure Boot/MOK; NVIDIA kmods + Container Toolkit/CDI; VFIO/Looking Glass/KVMFR/QEMU/libvirt; Gamescope/Waydroid/Mesa/ROCm; FreeIPA/SSSD/GNOME/WSL2; kargs.d/Renovate/systemd-sysext/tmpfiles/bootc lifecycle.

Executed by four parallel research sub-agents (general-purpose subagent_type) with the instruction "anchor against what changed since ~Feb 2026 — the project is roughly two months into v0.1.4."

**Files created (this directory only — strict scope compliance):**
- `bootc-research-april2026.md` — live knowledge doc (~450 lines, 12 topic sections, top-level action-items header).
- `ai-journal.md` — this file.
- `NEXT-RESEARCH.md` — tomorrow's agenda.

**Top findings flagged as `ACTION REQUIRED` in NEXT-RESEARCH.md** (not applied — research agent only):
1. `bsherman/ucore-hci` upstream is **archived**. The project's `BASE_IMAGE=ghcr.io/ublue-os/ucore-hci:stable-nvidia` should migrate to `ghcr.io/ublue-os/ucore:stable-nvidia-lts` (carries 580 LTS pre-signed `kmod-nvidia-open`). Renovate's `customManagers` regex on `Containerfile` will need its `depName` updated; auto-PRs cannot self-correct a depName change.
2. **cosign verification-bypass CVEs** — `CVE-2026-22703` and `CVE-2026-31431` — both fixed in **cosign ≥ 3.0.6**. Any sig-verification gate using older cosign is bypassable.
3. **NVIDIA Jan 2026 advisory** — pin `kmod-nvidia-open` ≥ 580.126.20 (LTS) or ≥ 595.71.05 (feature) to cover CVE-2025-33219, CVE-2025-23277, CVE-2025-23280.
4. **Secure Boot 2011-CA expiry — 2026-06-26.** MiOS must pick up Fedora's 2023-CA-signed shim before that date.
5. **RTX 50-series VFIO passthrough is broken.** RTX 4090 (project target) unaffected; any roadmap upgrade past 4090 must defer.

**Surprises:**
- `image-versions.yml` lists `ghcr.io/osautomation/image-builder-cli` with digest `0000…0000`. No `osautomation` GitHub org locatable upstream. Either a typo for `osbuild/image-builder-cli` or an internal/private fork. Flagged for clarification.
- LBI pre-pull is intentionally disabled in `Containerfile` (lines 67–76) because GitHub-hosted runners don't grant `--privileged` BuildKit; migration path is Quadlet `AutoUpdate=registry` at first boot. Confirmed this is still the right call in May 2026.
- GNOME 50 is out and **fully Wayland-only** (X11 backends removed from Mutter/gnome-shell/gnome-session/Control Center, ~27.5k LOC dropped). Any X11 fallback in the MiOS profile is dead code.
- The ASUS X870E platform (project's 9950X3D target board) has a documented FLR-permanent-bifurcation pathology with RTX 50-series — worth verifying behaves correctly on the 4090.
- bootc kargs.d `match-platforms` and `priority` keys do NOT exist despite occasional community wishlists. The project's `usr/lib/bootc/kargs.d/00-mios.toml` flat-array format is correct and current. Honor it strictly.

**Prior journal entries resolved or invalidated:** None — this is the first entry.

**Files modified outside `.ai-context/`:** None. Strict scope compliance.

**Next pass:** Per `NEXT-RESEARCH.md`. Priority list reasoning is captured there.

---

## 2026-05-16T13:00Z — `scheduled-research-daily`

**Agent ID:** `scheduled-research-daily`
**Context:** Second pass on the live research doc. 5 days since bootstrap. Followed `NEXT-RESEARCH.md` agenda (P0 reverify → P10 systemd 260). Dispatched two parallel general-purpose research subagents — one for P0 reverify + P1–P5, one for P6–P10 + CVE feeds.

**Project state at time of run (unchanged):**
- Version: `v0.1.4` (still — no rev in 5 days).
- Base image: `ghcr.io/ublue-os/ucore-hci:stable-nvidia` (Containerfile line 19 / `image-versions.yml`).
- Most recent commit: `45bf2b0 research: daily pass 2026-05-11 — bootstrap .ai-context` (HEAD of `main`).

**CHANGED upstream this window (knowledge-doc edits applied):**
1. **NVIDIA LTS floor → 580.159.04** (2026-05-14) — supersedes prior 580.126.20. Action item #3 updated; §8.1 rewritten with new release dates.
2. **`image-builder-cli` v64 (2026-05-13)** dropped "bootc is experimental" — bootc subcommand now GA. `ghcr.io/osbuild/image-builder-cli:latest` is canonical. §1.3 rewritten; new action item #6.
3. **Podman 6.0 GA imminent** — Fedora test days closed 2026-05-15, GA target week of 2026-05-25. Breaking removals: BoltDB, slirp4netns, cgroups v1. Netavark default switches iptables → nftables. §4.1 rewritten; new action item #7.
4. **K3s** — v1.34.8-rc1 + v1.35.5-rc1 cut 2026-05-14; v1.36.0 stable 2026-05-06. **etcd 3.5.30** shipped 2026-05-01. §5.1 rewritten with the etcd patch line (3.5.28 was a security release for CVE-2026-33343, CVE-2026-33413).
5. **WSL 2.7.5 pre-release** 2026-05-15 (kernel 6.18.26.1, skips 2.7.4). §11.3 updated.
6. **Renovate 43.181.0** (was 43.173.0 at bootstrap — 8 minor bumps in window). §12.2 updated.
7. **systemd 260** is actually **stable since 2026-03-17** — bootstrap doc said "in development". §12.3 corrected.
8. **`bsherman/ucore-hci` upstream now HTTP 404** (was "archived" at bootstrap). `ublue-os/ucore-hci` GHCR container still actively rebuilt (`stable-nvidia-lts-20260511`). §2 updated. New tracking: `ublue-os/ucore` issues #362 (longterm-6.12 → longterm-6.18) and #385 (kernel bump for CVE-2026-31431).
9. **Gamescope point releases** — 3.16.21 (2026-03-12), 3.16.22 (2026-03-15), 3.16.23 (2026-04-07) shipped between Sept-2025 base 3.16.17 and this pass. §10.1 corrected from "3.16.17" to "3.16.23".
10. **KVMFR kernel ≥6.13 compat patches** — need `#include <linux/vmalloc.h>` + `MODULE_IMPORT_NS("DMA_BUF")`. §9.3 updated. Important once ucore-hci LTS image migrates 6.12 → 6.18.

**NEW — added section §6.5 "Linux kernel CVE cluster — May 2026"** capturing five 2026-05-01 → 2026-05-08 kernel CVEs:
- CVE-2026-31431 ("Copy Fail" root LPE)
- CVE-2026-43398 (AMDGPU OOM DoS)
- CVE-2026-43300 (DRM panel NULL deref)
- CVE-2026-43287 (DRM property-blob memcg)
- CVE-2026-43284 ("Dirty Frag" via ESP/RxRPC)

Project owns no direct remediation — track `ucore-hci` kernel bump in issue #385.

**CORRECTION / prior-entry invalidation:**
- **2026-05-11 entry §7.1:** listed `CVE-2026-31431` as a cosign `verify-blob-attestation` false-OK bug. **That attribution is wrong.** CVE-2026-31431 is the Linux kernel "Copy Fail" LPE published by Microsoft on 2026-05-01. The cosign verification fixes in 3.0.6 / 2.6.3 are covered by `GHSA-w6c6-c85g-mmv6`, `GHSA-wfqv-66vq-46rm`, `GHSA-whqx-f9j3-ch6m` — cosign pin recommendation ≥ 3.0.6 still stands. §7.1 rewritten to remove the CVE-31431 reference; correction noted inline in §6.5 (added) and in the top action-items header.

**RESOLVED follow-up questions from bootstrap pass:**
- Q1 (bootstrap follow-up #1: "What is `ghcr.io/osautomation/image-builder-cli`?") — **CONFIRMED typo for `osbuild`.** GitHub user `osautomation` exists with zero public repos / no GHCR packages. Action item carried forward in NEXT-RESEARCH.md for hand-fix in `image-versions.yml`.
- Q5 (bootstrap follow-up #5: K3s HA mode question) — NOT resolved this pass; was a code-inspection question (`automation/13-ceph-k3s.sh`) outside the research-only scope.

**UNRESOLVED follow-up questions** (carried forward to NEXT-RESEARCH):
- Q2 (workflows still calling `ublue-os/bootc-image-builder-action`?)
- Q3 (cosign binary pinned in `automation/42-cosign-policy.sh`?)
- Q4 (SELinux site modules — going to `/etc/selinux/targeted/active/modules/400/`?)
- Q6 (`52-bake-kvmfr.sh` signs at build-time, not first-boot?)
- Q7 (fapolicyd trust-DB rebuilt at image-build vs runtime?)
- Q8 (GNOME 50 in `ucore-hci:stable-nvidia`?)

**NO CHANGE this window:**
- GNOME 51 alpha — still on calendar for 2026-06-27.
- Fedora 45 schedule — wiki schedule page 404; Beta target ~2026-08-25 per consensus.
- Composefs v1.1 — no tag (still 1.0.8, 2025-01-03). Kernel prereqs already landed in 6.5/6.6; bottleneck is now userspace.
- Looking Glass B8 — still B7 as latest.
- Gamescope 3.17 — no tag.
- RTX 50-series passthrough — still broken; 595.71.05 did not include a fix; no 600-series driver.

**Files modified outside `.ai-context/`:** None. Strict scope compliance.

**Surprises:**
- The image-builder-cli "drop bootc-experimental" PR landed three days before this pass (2026-05-13). Significant — opens a real migration path for the project.
- `bsherman/ucore-hci` went from "archived" to "404" between bootstrap and this pass. Suggests the owner deleted it; doesn't affect the ublue-os/ucore-hci GHCR rebuild but the historical link is gone.
- The CVE-2026-31431 mis-attribution in the bootstrap pass was a clean copy-paste of two different unrelated vulns with the same CVE-year tag. Worth a journal flag against future "same number, different vuln" footguns.

**Next pass:** Per overwritten `NEXT-RESEARCH.md`.

---

## 2026-05-18T13:00Z — `scheduled-research-daily`

**Agent ID:** `scheduled-research-daily`
**Context:** Third pass on the live research doc. 2 days since previous (2026-05-16). Followed `NEXT-RESEARCH.md` agenda (P0 reverify → P10 RTX 50-series). Dispatched two parallel general-purpose research subagents — one for P0 + P1–P4, one for P5–P10 + the ecosystem watch list.

**Project state at time of run (unchanged):**
- Version: `v0.1.4` (still — no rev in 7 days).
- Base image: `ghcr.io/ublue-os/ucore-hci:stable-nvidia` (Containerfile line 19 / `image-versions.yml`).
- Most recent commit on `origin/main`: `a208eac research: daily pass 2026-05-16 (cont'd) — live knowledge doc`.

**CHANGED upstream this 2-day window (knowledge-doc edits applied):**

1. **Podman 6.0 GA SLIPPED — retargeted to Fedora 45.** Bootstrap and 2026-05-16 entries both expected GA "week of 2026-05-25." The Fedora Change Proposal at `fedoraproject.org/wiki/Changes/Podman6` is now tagged `ChangeAcceptedF45` (last edited 2026-03-11) — the F44 target was actually abandoned **before** the bootstrap pass; the 2026-05-16 prediction was wrong in retrospect. No upstream 6.0 RC tag has been cut. Test Days closed 2026-05-15 with no post-test-day report. Knowledge-doc §4.1 + action-item #7 rewritten; deadline pressure relieved.

2. **Kernel CVE cluster expanded from 5 → ~12 CVEs.** Sub-agent surfaced an AMDGPU sub-cluster (CVE-2026-43318, 43400, 43298, 43237, 43320, 43305) all in the 2026-05-08 disclosure window, plus CVE-2026-46300 "Fragnesia" (networking, AlmaLinux 2026-05-13). Kernel 6.18.28 (2026-05-08), 6.18.30 (2026-05-14), 6.12.87, 6.12.88 carry the backports. Knowledge-doc §6.5 + action-item #8 expanded. Added an "AMD iGPU exposure on 9950X3D" note since the AMDGPU cluster lands directly on the host iGPU unless blacklisted — flagged for the project owner.

3. **ucore-hci daily-rebuild cadence stalled.** Latest tag still `stable-nvidia-lts-20260511`, 7 days old as of 2026-05-18. This is unusual for the ublue-os daily-bake pattern and compounds risk on the kernel CVE cluster (none of the new fixes have propagated to MiOS's base image). Action-item #1 + §2 updated.

4. **NVIDIA 595.44.08 Vulkan developer-beta** released 2026-05-15. Bootstrap doc had listed it without distinguishing it from the 595.71.x production-feature branch. §8.1 split into Production-Feature (595.71.05) vs Developer-Beta (595.44.08) — they are **not** the same line. Important for project pin clarity.

5. **K3s v1.34.8 / v1.35.5 GA still pending.** RCs cut 2026-05-14 have not promoted in the 2-day window. **CVE-2026-33186** (gRPC-Go authz bypass, CVSS 9.1) is still not explicitly called out in any K3s RC notes. Watch item until GA. §5.1 updated.

6. **Pacemaker 3.0.2-rc2 cut 2026-05-11** — 45 commits, XPath + memory-leak fixes. §5.3 updated.

7. **CrowdSec 1.7.8** (2026-05-11) — adds WAF OpenAPI schema validation, body-size limits, decision-stream chunked-transfer improvements. Bootstrap baseline 1.7.6 superseded. §6.1 updated. (Sub-agent flagged some WebFetch results returned "2024" dates that look misparsed — release ordering matches 2026 cadence; treat 1.7.8 as current.)

8. **GNOME 50.2 stable point release scheduled 2026-05-23** (5 days out). GNOME 51.alpha date **confirmed** at 2026-06-27. §11.2 updated.

9. **Renovate 43.182.4** (2026-05-18 10:44Z) — 8 minor versions in the 2-day window. Routine cadence. §12.2 updated.

**CORRECTIONS — stale baselines fixed (these had been wrong since bootstrap, not new this window):**

- **Mesa 25.3.4 → 26.1.0** (2026-05-06). The 26.x series shipped before bootstrap and was missed. §10.3 corrected.
- **QEMU 10.2.0 → 11.0.0** (2026-04-22). 2500+ commits, 237 authors, Nitro Enclaves accelerator. Shipped before bootstrap and missed. §9.4 corrected. **Worth verifying VFIO/PCI passthrough behavior on the 9950X3D + RTX 4090 path before any project bump.**
- **libvirt 12.1.0 → 12.3.0** (2026-05-02). 12.2.0 (2026-04-01) was the intermediate point. §9.5 corrected.
- **fapolicyd 1.3.8 → 1.4.5** (2025-03-30). 1.4.x line had already shipped before bootstrap and was missed. §6.2 corrected, with a flag to verify Fedora package pin matches upstream-latest.

**NO CHANGE confirmed this window** (still current):
- bootc v1.15.2 (2026-05-01).
- composefs 1.0.8 (2025-01-03) — still experimental in bootc native backend.
- OSTree v2026.1 (2026-04-10).
- cosign v3.0.6 (2026-04-06) — no new GHSAs.
- Looking Glass B7 (2025-03-06) — no B8 RC.
- Gamescope 3.16.23 (2026-04-07) — issue #2037 HDR fix commit `7d4e835` still unreleased.
- etcd 3.6.11 / 3.5.30 / 3.4.44 (2026-05-01).
- Ceph 20.2.1 Tentacle (2026-04-06).
- systemd 260 (2026-03-17) / 260.1 (2026-03-23).
- WSL 2.7.3 GA / 2.7.5 pre-release (2026-05-15) — same as 2026-05-16.
- Waydroid 1.6.2, ROCm 7.2.3, SSSD 2.13.0, NVIDIA Container Toolkit 1.19.0, usbguard 1.1.4, NVIDIA LTS 580.159.04.
- RTX 50-series passthrough still broken; no fix.
- shim-16.1-6 still not in F44 stable — bodhi.fedoraproject.org now gated by Anubis (direct fetch blocked); no Fedora Discussion thread in the 2-day window. Next checkpoint scheduled for 2026-06-05.
- image-builder-cli v64 (2026-05-13) — no v65 yet. **Softened the parity claim** — public docs only enumerate qcow2 + bootc-installer ISO patterns; `raw`, `ami`, `vmdk`, `vhd`, `gce` are not documented. BIB still has wider format matrix.

**RESOLVED follow-up questions from 2026-05-16 pass:** None — the 6 unresolved questions (workflow inspection, cosign script pin, SELinux module path, KVMFR signing timing, fapolicyd trust DB, GNOME 50 in stable-nvidia) all remained code-inspection questions outside the research-only scope. Carried forward to NEXT-RESEARCH.

**UNRESOLVED follow-up questions** (carried forward to NEXT-RESEARCH):
- Same 6 from the 2026-05-16 pass plus follow-ups 9 (AMD iGPU usage / blacklist defense-in-depth) and 10 (systemd 260 central sysext config consumption).
- **NEW this pass:** Does the project pin `nvidia-open` to a specific branch (595.71.x production-feature vs 595.44.x developer-beta vs 580.x LTS)? Bootstrap noted feature/LTS but did not specify branch line.
- **NEW this pass:** Is Mesa in the running `ucore-hci:stable-nvidia` image on the 25.x or 26.x line? (Cannot inspect from research-only scope.)

**Files modified outside `.ai-context/`:** None. Strict scope compliance.

**Surprises:**
- The Podman 6.0 GA target had **already** been shifted to F45 by 2026-03-11. Both bootstrap (2026-05-11) and 2026-05-16 passes called it "imminent" / "Test Days are pre-GA" — but the F44 plan was abandoned weeks earlier. The communityblog / wiki signal got read as "Test Days happening this week = GA next week" when in fact it was "Test Days happening this week before deferring the version to next Fedora cycle." Worth journaling as a "don't infer GA from Test Days proximity" footgun.
- Three knowledge-doc baselines (Mesa, QEMU, libvirt) were stale at bootstrap and only caught on this pass. The bootstrap sweep apparently took outdated numbers from secondary sources. Anchoring against project GitHub Releases pages should be primary going forward.
- ucore-hci daily-rebuild cadence stalling at exactly the same week as the kernel CVE cluster is unfortunate timing — the project relies on ublue-os to land the kernel bump, and the bake cadence has just paused. Worth a closer watch tomorrow.

**Prior journal entries resolved or invalidated:** Bootstrap pass and 2026-05-16 pass both predicted Podman 6.0 GA week of 2026-05-25. **Both predictions are now invalidated** by the F45 retarget. No earlier journal entries were proven outright false beyond what was already corrected in the 2026-05-16 entry.

**Next pass:** Per overwritten `NEXT-RESEARCH.md`. P0 (reverify) priority remains; new top-of-funnel watches: kernel 6.12.88 propagation in ucore-hci, K3s GA + CVE-2026-33186 callout, shim-16.1-6 F44 stable.

**Drive mirror note:** Daily Drive snapshot uploaded as `CloudWS-bootc-research-2026-05-18.md` (Drive file id `1qK6cKQDU63KIFwrVZ8PJDagg41AeUt8a`). **Trade-off:** the Drive file is an *index* pointing back to the git-tracked full doc at this pass's commit, not a verbatim 56KB copy of the knowledge doc. The Read tool returned a hard 25K-token cap per call and the formatted markdown crosses that limit (~26K tokens); chunked-reassembly into a single MCP `textContent` parameter was attempted but proved fragile, and the git commit is the authoritative archive regardless. Future passes should consider chunked upload or an alternate mirror path if the verbatim-copy requirement matters.

---

## 2026-05-19T12:00Z — `scheduled-research-daily`

**Agent ID:** `scheduled-research-daily`
**Context:** Fourth pass on the live research doc. 1 day since previous (2026-05-18). Followed `NEXT-RESEARCH.md` agenda (P0 reverify of 9 ACTION REQUIRED items → P1–P10). Dispatched two parallel general-purpose research subagents — one for P0 reverify + P1–P4, one for P5–P10 + ecosystem watch.

**Project state at time of run (unchanged):**
- Version: `v0.1.4` (still — no rev in 8 days).
- Base image: `ghcr.io/ublue-os/ucore-hci:stable-nvidia` (Containerfile line 19 / `image-versions.yml`).
- Latest commit on `origin/main`: `a208eac research: daily pass 2026-05-16 (cont'd) — live knowledge doc` (unchanged; the 2026-05-18 pass committed but I have not verified the local-HEAD vs. origin/main status in this notebook).

**CHANGED upstream this 24h window (knowledge-doc edits applied):**

1. **NEW: ucore PR #392 (F43 → F44 base migration)** opened 2026-05-17T15:57Z by dylanmtaylor — flips `FEDORA_VERSION` 43 → 44 in `ucore/Justfile` plus mergerfs github-pkgs JSON tag bump + `install-ucore.sh` tweaks. **This is the implicit kernel-bump path** (F44 base brings a newer kernel package set; not a direct kernel pin change). Still open, unreviewed. Same author as issue #385, so #385 may resolve when #392 lands. **Most significant new datapoint of the 24h window** — see §2 + action item #1.

2. **Issue #385 activity:** updated 2026-05-17T12:54Z, now has **7 comments** (was 0 before). Comment bodies not retrievable via WebFetch (JS-rendered). §2 updated.

3. **CVE-2026-31431 "Copy Fail" is on CISA KEV; federal remediation deadline 2026-05-15 — NOW 4 DAYS PAST.** NVD entry last-modified 2026-05-18 confirms KEV inclusion. CVSS 7.8. Vulnerable kernels ≤6.18.21 / ≤6.19.11 / ≤6.12.84; fixed in 6.18.22 / 6.19.12 / 7.0. **MiOS LTS base (`stable-nvidia-lts-20260511`) is built before any of these fixes landed → MiOS is vulnerable.** Action item #8 + §6.5 rewritten with federal-tier severity flag.

4. **GHSA-w6c6-c85g-mmv6 now has CVE assignment CVE-2026-39395** (CVSS 4.3 Moderate; `verify-blob-attestation` false positive). Already fixed in cosign 3.0.6 — no remediation change. Action item #2 + §7.1 updated.

5. **Mesa 26.0.7 stable backport** shipped 2026-05-14. The 26.0.x is the maintenance branch parallel to the newer 26.1.x feature branch. §10.3 updated.

6. **WSL 2.7.6 stable (2026-05-18)** + **WSL 2.8.6 pre-release (2026-05-14)** — parallel 2.7.x stable + 2.8.x pre-release trains is a new cadence shift. 2.7.6 fixes Start menu GUI app icons on Azure Linux 3 system distros. §11.3 updated.

7. **Renovate 43.185.1 (2026-05-19, today)** — 4 releases in the 24h window: 43.183.0, 43.184.0, 43.185.0 (all 2026-05-18) then 43.185.1 today. 43.185.1 is a GitHub tags datasource bugfix. §12.2 updated.

8. **AlmaLinux CVE-2026-46300 ("Fragnesia") fixed-kernel pins captured** for cross-distro reference: AlmaLinux 10 = `kernel-6.12.0-124.56.3.el10_1`, AlmaLinux 9 = `kernel-5.14.0-611.54.5.el9_7`, AlmaLinux 8 = `kernel-4.18.0-553.124.3.el8_10`. §6.5 updated.

9. **F45 Beta confirmed 2026-08-25** via Wikipedia release-history + Fedora ChangeSet cross-reference (`fedoraproject.org/wiki/Releases/45/Schedule` still 404 on second pass; `fedorapeople.org/groups/schedule/f-45/*` is now Anubis-gated). F45 Atomic Desktops direction: "switch the builds of the Fedora Atomic Desktop ISOs over from lorax to image-builder" + add qcow2/raw artifacts. **Composefs+UKI sealed-image is NOT confirmed as a default F45 deliverable** — sealed-image work continues in `travier/fedora-atomic-desktops-sealed` (WIP, unofficial). New §3.4 added.

10. **Looking Glass master appears stalled** — no visible commits between 2026-01-17 and 2026-05-19 (4-month gap, confirmed across two probes). Possible upstream stall. §9.2 updated.

**CORRECTION — prior-entry data points fixed (these had been wrong before today):**

- **fapolicyd v1.4.5 date corrected: 2025-03-30 → 2026-03-30.** The 2026-05-18 entry pulled the date from a WebFetch of the GitHub Releases HTML page that lost a year on parse. The Releases atom feed renders 2026 and is internally consistent with the 1.4.x cadence (1.4.2 = 2025-11-26, 1.4.3 = 2026-01-13, 1.4.4 = 2026-03-19, 1.4.5 = 2026-03-30). Atom feed is now the authoritative date source. §6.2 rewritten. Two other items had the same parse-failure pattern this pass (NVIDIA Container Toolkit v1.19.0 "2025-03-12" → 2026-03-12; systemd v260.1 "2025-03-23" → 2026-03-23); both atom-feed-verified, both already on 2026 in the knowledge doc, so no edits needed.

- **Corosync version corrected: 3.1.1 → v3.1.10 (2024-11-15).** Bootstrap baseline (3.1.1) was many releases behind. v3.1.10 carries the CVE-2025-30472 fix. §5.3 updated.

**NO CHANGE confirmed this window:**
- bootc v1.15.2 (2026-05-01).
- OSTree v2026.1 (2026-04-10).
- cosign v3.0.6 (2026-04-06) — no new GHSAs; just CVE assignment to existing GHSA-w6c6-c85g-mmv6.
- composefs v1.0.8 (2025-01-03) — still 16+ months stale on tags; main has last visible commit 2026-01-15 ("Add CNCF copyright footer"); no v1.1 motion.
- bootc native composefs backend — still "experimental" with on-disk-format-may-change warning.
- image-builder-cli v64 (2026-05-13) — no v65 (6 days, cadence is 1-2 weeks). Issue #506 (composefs+UKI sealed-image bootc backend, 2026-04-29) is the upstream gating item.
- BIB — 76 open issues, no deprecation timeline.
- Pacemaker 3.0.2-rc2 (2026-05-11) — 3.0.2 final not shipped (8 days into rc2; rc1→rc2 gap was 17 days).
- NVIDIA driver lineup unchanged (LTS 580.159.04 / production-feature 595.71.05 / dev-beta 595.44.08).
- No new NVIDIA security bulletin since Jan 2026.
- Looking Glass B7 (2025-03-06) — no B8 RC.
- Gamescope 3.16.23 (2026-04-07) — HDR fix commit `7d4e835` still in master, not in any tagged release.
- K3s — RCs still 5 days old, no GA promotion. CVE-2026-33186 still uncalled-out in v1.35.5-rc1 release notes (which I inspected this pass: notes mention Go 1.25.9 + 2026-05 backports + local-path-provisioner bump but no grpc-go callout).
- etcd 3.6.11 / 3.5.30 / 3.4.44 (2026-05-01).
- Ceph 20.2.1 Tentacle (2026-04-06); also noted Reef 18.2.8 (2026-03-20, final backport).
- CrowdSec 1.7.8 (2026-05-11). NVIDIA Container Toolkit v1.19.0 (2026-03-12). QEMU 11.0.0 (2026-04-22). libvirt 12.3.0 (2026-05-02). systemd v260.1 (2026-03-23).
- shim-16.1-6 in F44 stable — still cannot verify (Anubis-gated infrastructure); next checkpoint 2026-06-05; 38 days to 2026-06-26 cutover.
- Podman 6.0 — no RC tag.
- RTX 50-series passthrough still broken; Level1Techs forum thread now returns 503; Tom's Hardware confirms active $1,000 bounty.
- `osbuild/bootc-image-builder-action` — one new dependabot commit on 2026-05-05 (`31d72f7` npm-production group bump); latest tag still `0.0.2` (2025-05-18).
- `ublue-os/bootc-image-builder-action` — confirmed maintenance-mode, no new activity.

**RESOLVED follow-up questions from 2026-05-18 pass:** None — the unresolved questions remain code-inspection questions outside the research-only scope. Carried forward.

**UNRESOLVED follow-up questions** (carried forward to NEXT-RESEARCH).

**NEW this pass:**
- **Does ucore PR #392 land before 2026-06-05 (next shim checkpoint)?** If yes, kernel-CVE remediation path arrives at MiOS via a base-image rebuild. If no, the federal-deadline-missed status (action item #8) compounds with the Secure Boot deadline.
- **Does the F44 base kernel package set in ucore actually carry the CVE-2026-31431 fix?** F44 typically ships kernel ≥ 6.18.x, but the exact kernel pin needs verification once PR #392 merges. NVD lists fixed in 6.18.22 / 6.19.12 / 7.0 — F44 may already be on a fixed version.
- **What is the canonical non-Anubis path to verify Fedora F44 package versions?** Today's pass settled on `dl.fedoraproject.org/pub/fedora/linux/updates/44/Everything/x86_64/Packages/s/` as the answer. Worth a poll-job-style automation to detect shim-16.1-6 promotion.

**Files modified outside `.ai-context/`:** None. Strict scope compliance.

**Surprises:**
- **ucore upstream main has been idle for 12 days** (last commit 2026-05-07 "Fix 404 link to cosign in README #382"). This is a stronger signal than the cadence stall alone — the maintainers may be working off-tree or attention has shifted. PR #392 is a one-author drive-by from dylanmtaylor.
- **CISA KEV deadline already past for CVE-2026-31431** changes the severity framing for federal-adjacent users. MiOS itself is not federally bound, but the "deadline-already-past" framing is a stronger nudge than "CVSS 7.8 LPE."
- **GitHub HTML pages render issue comments via JS** — WebFetch's markdown conversion does not see them. Comment-body retrieval requires either authenticated `api.github.com` (403 unauthenticated) or `mcp__github__issue_read` (scoped to `kabuki94/cloudws-bootc` only). `mcp__github__search_issues` returns issue metadata + body cross-repo but not comments. This shapes what's verifiable in research-only mode for upstream issues.
- **WSL forked into parallel 2.7.x stable + 2.8.x pre-release trains** is a real cadence shift — previous pre-releases were on the same train as stable.

**Prior journal entries resolved or invalidated:** None outright invalidated. The 2026-05-18 entry's fapolicyd date (2025-03-30) was corrected to 2026-03-30 — a parse-failure correction, not a fundamental invalidation.

**Next pass:** Per overwritten `NEXT-RESEARCH.md`. New top-of-funnel watches: PR #392 merge status, kernel package version landing in any new ucore-hci tag, shim-16.1-6 F44 stable promotion via dl.fedoraproject.org mirror.

**Drive mirror note:** Daily Drive snapshot uploaded as `CloudWS-bootc-research-2026-05-19.md` (Drive file id `1GArjTrFe323rxfAdMBD6M1q3KBupJ94d`). Per-pass git push was blocked by a persistent HTTP 503 on the proxy's git-receive-pack endpoint (4+ retries with exponential backoff all failed); fell back to per-file `mcp__github__create_or_update_file` uploads — three commits on the remote, one per file. After remote was synchronized, local was hard-reset to origin/main to align with the MCP-push state.

---

## 2026-05-22T12:00Z — `scheduled-research-daily`

**Agent ID:** `scheduled-research-daily`
**Context:** Fifth substantive pass on the live research doc. 3 days since the previous *journaled* pass (2026-05-19). Followed the `NEXT-RESEARCH.md` agenda (P0 reverify of 9 ACTION REQUIRED items → P1–P10). Dispatched **three** parallel general-purpose research subagents: (A) kernel CVEs + base image + Secure Boot + cosign + NVIDIA; (B) K3s/Pacemaker/Podman/image-builder/composefs/bootc/etcd/Ceph/CrowdSec/fapolicyd/GNOME/WSL/Renovate/systemd; (C) Looking Glass/Gamescope/QEMU/libvirt/Mesa/Waydroid/ROCm/NVIDIA-Container-Toolkit.

**Project state at time of run (unchanged):**
- Version: `v0.1.4` (still — no rev in 11 days).
- Base image: `ghcr.io/ublue-os/ucore-hci:stable-nvidia` (Containerfile / `image-versions.yml` line 12).
- Git: started on detached HEAD at `efdf712 research: daily pass 2026-05-20 — NEXT-RESEARCH.md (1/3, MCP push, git-receive-pack 503)`; checked out `main` (in sync with `origin/main`) before editing.

**IMPORTANT — the missing 2026-05-20 pass:** HEAD was `efdf712`, a commit titled "daily pass 2026-05-20 — NEXT-RESEARCH.md (1/3 ...git-receive-pack 503)". That pass committed **only** `NEXT-RESEARCH.md` (the "1/3") before a git-receive-pack 503; its journal (2/3) and knowledge-doc (3/3) edits **never landed**. That is why both `ai-journal.md` and `bootc-research-april2026.md` stopped at 2026-05-19 while `NEXT-RESEARCH.md` carried 2026-05-20 findings. **Today's pass therefore had to (a) fold the un-committed 2026-05-20 findings into the doc AND (b) add fresh 2026-05-22 deltas.** No 2026-05-20 or 2026-05-21 journal entry exists; this entry covers the gap.

**CHANGED upstream / folded in this pass (knowledge-doc edits applied):**

1. **Kernel local-root cluster grew from a Copy-Fail-centric framing to FIVE named vectors.** Added to §6.5 + action item 8:
   - `CVE-2026-31635` "DirtyDecrypt" (RxGK/AFS-rxrpc LPE, CVSS 7.5, **only on `CONFIG_RXGK` kernels = Fedora/Arch** → applies to MiOS base; public PoC 2026-05-18; fix commit `aa54b1d27fe0`). **Date conflict noted:** earlier source said merged 2026-04-25, kernel-git shows commit ~2026-05-10 — left unconfirmed since the base is pre-fix regardless.
   - `CVE-2026-46333` "ssh-keysign-pwn" (ptrace exit-race info-disclosure → LPE; steals SSH host keys + `/etc/shadow`; fix floors 7.0.8 / 6.18.31 / 6.12.89 / 6.6.139 / 6.1.173; fix commit `31e62c2ebbfd`, 2026-05-14).
   - `CVE-2026-43500` (new RxRPC page-cache-write LPE) — now grouped by Red Hat under **RHSB-2026-003 "Dirty Frag"** together with the existing CVE-2026-43284 and CVE-2026-46300. Mechanism: deterministic LPE via `splice` / `MSG_SPLICE_PAGES` page-cache writes; public PoC; affects Fedora via RxRPC. Updated the CVE-2026-43284 + CVE-2026-46300 bullets to reflect the RHSB-2026-003 consolidation.
   - Recorded kernel.org current stable/longterm (2026-05-17, unchanged: 7.0.9 / 6.18.32 / 6.12.90 / 6.6.140 / 6.1.173; mainline 7.1-rc4).

2. **NVIDIA May 2026 security bulletin `a_id/5821` is PUBLISHED** (2026-05-19, updated 2026-05-21; 13 CVEs, 12 Linux, top **CVE-2026-24187** CVSS 8.8 Linux UAF). **This corrects the doc's prior "No May 2026 NVIDIA bulletin published" line** in §8.1, which was true on 2026-05-16 but false by 2026-05-19. Fix floors recorded; **project pins (LTS 580.159.04, feature 595.71.05) already satisfy all of them — no bump forced.** Action item 3 downgraded. §8.1 header + bullet rewritten.

3. **K3s GA shipped 2026-05-20** — v1.36.1+k3s1 / v1.35.5+k3s1 / v1.34.8+k3s1 (+ v1.33.12+k3s1). **CVE-2026-33186 (grpc-go authz bypass, CVSS 9.1) is REMEDIATED** even though GA notes are silent: `v1.36.1+k3s1` `go.mod` carries `replace google.golang.org/grpc => v1.79.3` overriding `require ...grpc v1.80.0`. **Correction: vendored Go is 1.26.2, not 1.25.9** as the 2026-05-19 note recorded. §5.1 header, body line, and the CVE-2026-33186 bullet all updated (bullet marked RESOLVED).

4. **fapolicyd v1.5 (2026-05-20)** — first minor bump off 1.4.x (transactional rule reload, `--check-rules`, per-rule hit counters, decision-timing framework). Flagged "validate rule-reload/trust-DB parity before pinning" since it's a deny-by-default execution gate. §6.2 updated.

5. **Routine bumps recorded:** image-builder-cli **v65** (2026-05-21, no new format coverage; issue #506 still open); NVIDIA Container Toolkit **v1.19.1** (2026-05-21, CDI tweaks); Mesa **26.1.1** (feature branch); WSL **2.7.7** (2026-05-19 stable); Renovate **43.192.0** (2026-05-22); etcd **3.7.0-beta.0** (2026-05-19 pre-release, not for prod); libvirt **12.4.0** in-progress/unreleased.

**CORRECTION / prior-content invalidation:**
- §8.1's "No May 2026 NVIDIA bulletin published (verified 2026-05-16)" is now **invalidated** — the bulletin exists (a_id/5821). Corrected in place; the Jan 2026 advisory a_id/5747 retained as the older floor.
- §5.1's "Go 1.25.9" recorded on 2026-05-19 was **wrong** — GA vendored Go is 1.26.2. Corrected.

**RESOLVED follow-up questions:**
- **CVE-2026-33186 grpc-go callout (open since 2026-05-16)** — RESOLVED. The K3s 2026-05-20 GA links grpc-go v1.79.3 via a `replace` directive; release notes remain silent but the binary is patched. Any host on a K3s GA tag is covered. Long-standing watch item closed.
- **NVIDIA May 2026 bulletin (cadence-due)** — RESOLVED (published, pins satisfy).

**Carried forward (still unresolved — out of research-only scope):** the same code-inspection set (workflow `bootc-image-builder-action` migration; cosign binary pin in `42-cosign-policy.sh`; SELinux site-module path; KVMFR build-time signing; fapolicyd trust-DB rebuild timing; GNOME 50 in stable-nvidia; AMD iGPU blacklist decision; K3s HA vs sqlite mode; NVIDIA driver branch pin; Mesa line in the running image). New unresolved: which Fedora kernel NVR first carries DirtyDecrypt (bodhi Anubis-gated, unverifiable); whether fapolicyd v1.5's rule-reload changes affect the image-build trust-DB bake.

**Files modified outside `.ai-context/`:** None. Strict scope compliance.

**Surprises:**
- **The 2026-05-20 pass's doc + journal edits were lost to a git-receive-pack 503** — only the NEXT-RESEARCH commit (1/3) survived. This is the *second consecutive pass* (after 2026-05-19) to hit a proxy 503 on git push, and the first to lose work because of it. The recurring 503 on the git endpoint is a real operational risk for this scheduled job — flagged in NEXT-RESEARCH.
- **Zero forward motion on every ucore remediation lever for 3+ days** while the kernel local-root surface grew from ~1 headline CVE to 5 named vectors with public PoCs. PR #392 (the only path) has been static since 2026-05-17; ucore main idle 15 days; GHCR bake stalled 11 days. The gap between exposure and the project's ability to remediate is widening, and the project doesn't own the lever.
- **K3s shipping a CVE-9.1 fix via a silent `replace` directive** (no release-note callout) is a reminder that release notes are not a reliable CVE-remediation signal — go.mod inspection was required to confirm. Worth remembering for future "is this CVE fixed?" questions.

**Prior journal entries resolved or invalidated:** No prior journal entry outright invalidated. Two doc data points corrected (NVIDIA May-bulletin status; K3s Go version) as noted above.

**Next pass:** Per overwritten `NEXT-RESEARCH.md`. Tightest watches remain (1) Secure Boot shim-16.1-6 in F44 (~35 days to cutover, hard checkpoint 2026-06-05) and (2) PR #392 merge / any new ucore-hci tag. K3s pin bump to GA is now actionable for the owner.

**Drive mirror note:** Daily Drive snapshot uploaded as `CloudWS-bootc-research-2026-05-22.md` (Drive file id `1knZCUmYYRwmMI8E6d0IQlc_s9DmFjnj8`). **Trade-off (unchanged from prior passes):** the knowledge doc is now ~82KB / ~47K tokens — over the single-Read 25K-token cap and, per the 2026-05-18 journal note, chunked-reassembly into one MCP `textContent` parameter "proved fragile." The Drive file is therefore a dated **snapshot index** (action-item state + deltas + pointer back to the git-authoritative full doc), not a verbatim copy. The git commit for this pass is the authoritative verbatim archive. Future passes should consider an alternate verbatim-mirror mechanism if the requirement hardens (e.g. a path-based upload tool, or splitting the doc into per-topic files small enough to round-trip).

---

## 2026-05-24T12:00Z — `scheduled-research-daily`

**Agent ID:** `scheduled-research-daily`
**Context:** Sixth substantive pass. 2 days since the previous journaled pass (2026-05-22). Followed the `NEXT-RESEARCH.md` agenda (P0 reverify of 12 ACTION REQUIRED items → P1 Secure Boot → P2 PR #392 → P3 GNOME 50.2 → P4 Pacemaker → P5 DirtyDecrypt NVR → P6–P10). Dispatched **three** parallel general-purpose research subagents: (A) Secure Boot + ucore PR #392/issue #385 + kernel CVE cluster + cosign + NVIDIA; (B) GNOME/Pacemaker/Podman/K3s/etcd/Ceph/composefs/image-builder-cli/bootc/OSTree/fapolicyd/CrowdSec/systemd/Renovate/F45; (C) Looking Glass/Gamescope/QEMU/libvirt/Mesa/ROCm/NVIDIA-CT/Waydroid/WSL/FreeIPA-SSSD/RTX50.

**Project state at time of run (unchanged):**
- Version: `v0.1.4` (still — no rev in 13 days).
- Base image: `ghcr.io/ublue-os/ucore-hci:stable-nvidia` (`image-versions.yml` line 12).
- Git: started on detached HEAD at `2ae02de` (the 2026-05-22 pass, which had committed locally but `origin/main` was stale-tracked at `efdf712` / 2026-05-20). Checked out `main`, fast-forwarded `main` to `2ae02de` (clean FF — `main` was a strict ancestor), then confirmed via `git fetch` that `origin/main` **already had** `2ae02de` — the local tracking ref was simply stale. So the 2026-05-22 work was never actually lost on the remote (despite the "git-receive-pack 503" commit-message suffixes); it round-tripped via the MCP per-file pushes. No re-push needed beyond confirming sync. Working tree clean before editing.

**HEADLINE — `ublue-os/ucore` PR #392 MERGED 2026-05-24:**
- Commit `412e7be`, approved by @bsherman, all 73 checks passed. Migrates the **LTS image kernel flavor longterm-6.12 → longterm-6.18 on the F44 base**, plus mergerfs F44 RPM manifests + nfs-utils build fix. **ucore main is active again** (mergerfs v2.42.0 #390 on 2026-05-23, then the #392 merge — the 15-day idle gap is broken). This is the kernel-CVE remediation lever **finally moving at the source**.
- **Caveats folded into the doc (items 1/8, §2, §6.5):** (a) **issue #385 (Copy Fail tracker) was NOT auto-closed — still open**; (b) **no rebuilt `ucore-hci:stable-nvidia-lts-YYYYMMDD` GHCR tag confirmed yet** (GHCR tag enumeration unavailable via WebFetch; GitHub MCP scoped to `kabuki94/cloudws-bootc` only — manual `skopeo`/registry check is the open verification item); (c) longterm-6.18 is itself inside the DirtyDecrypt affected range (6.16.1–6.18.22), so the new image must bake **≥ 6.18.23 (DirtyDecrypt) / ≥ 6.18.31 (ssh-keysign-pwn)** — current longterm-6.18 is 6.18.33, so a fresh bake satisfies. (d) The merge does NOT change the base-ref the project must adopt (`ucore:stable-nvidia-lts`) or the Renovate `depName` hand-fix — only the kernel carried.
- **Issue #362** (longterm-6.12 → longterm-6.18 migration) is **effectively resolved** by this merge — noted in §2.

**CHANGED upstream this 2-day window (knowledge-doc edits applied):**
1. **PR #392 merged** — items 1 + 8, §2 header + #392/#385/#362 bullets, §6.5 header, and the Iterations log all updated.
2. **DirtyDecrypt (CVE-2026-31635) date conflict RESOLVED** — confirmed fixed **2026-04-25 upstream**; affected ranges **6.16.1–6.18.22 and 6.19.0–6.19.12** (fixed 6.18.23 / 6.19.13 / 7.0+). **F44 `updates` now ships `kernel-7.0.9-205.fc44`** (mirrors.kernel.org `updates/44/.../k/`) — past all affected ranges, so the current F44 kernel carries the DirtyDecrypt + Copy-Fail + ssh-keysign-pwn + Dirty-Frag fixes. §6.5 DirtyDecrypt bullet rewritten; the old "date unconfirmed — kernel-git shows ~2026-05-10" hedge removed.
3. **kernel.org bumped 2026-05-23** — mainline 7.1-rc4; stable **7.0.10**; longterm **6.18.33 / 6.12.91 / 6.6.141 / 6.1.174** (was 7.0.9 / 6.18.32 / 6.12.90 / 6.6.140 / 6.1.173). §6.5 updated.
4. **Fragnesia (CVE-2026-46300) confirmed a regression from the CVE-2026-43284 fix** (dropped `SKBFL_SHARED_FRAG` on the espintcp path) — clarifies it is not a separate root cause. §6.5 / item 8 updated.
5. **NVIDIA dev-beta 595.44.09 (2026-05-22)** supersedes 595.44.08 — Vulkan/bugfix pre-release, **not a project pin**. LTS 580.159.04 / feature 595.71.05 unchanged and still satisfy all May-bulletin floors; no June bulletin. Item 3 + §8.1 updated.
6. **systemd v261-rc1 (2026-05-22)** — unstable; stable track unchanged at 260.1. §12.3 updated.
7. **Renovate 43.195.0** (was 43.192.0). §12.2 updated.
8. **GNOME 50.2 did NOT ship** — 2026-05-23 was the *tarballs-due* date, not release; `download.gnome.org/core/50/50.2/` 404s; release expected ~week of 2026-05-25/26; remains bugfix-only. 50.x cadence dates captured (50.3 = 2026-06-27, …). §11.2 updated.

**Secure Boot (TIGHTEST DEADLINE) — re-verified, still absent:** `shim-16.1-6` is STILL absent from all three F44 trees (releases = `shim-x64/ia32-16.1-5`; updates + updates-testing = no shim package). rhboot/shim upstream tip still `16.1` (2025-08-13). ~33 days to the 2026-06-26 cutover; hard checkpoint 2026-06-05. Item 4 + §7.2 updated.

**NO CHANGE confirmed this window:** cosign 3.0.6 (no new GHSA/CVE); bootc 1.15.2; OSTree 2026.1; composefs 1.0.8 (no v1.1; main active but no release-prep; bootc native backend still verbatim "Experimental … not yet suitable for production use"); image-builder-cli v65 (no v66; issue #506 still open/no movement; BIB active, no deprecation; osbuild.org/blog still HTTP 404); Podman 5.8.2 (no 6.0 RC/beta, no v5.8.3; Quadlet schema-delta docs still placeholder); K3s GA tags from 2026-05-20 (no newer patch); etcd 3.6.11/3.5.30/3.4.44 + 3.7.0-beta.0; Ceph 20.2.1 Tentacle (no 20.2.2/bulletin); Pacemaker 3.0.2-rc2 (no rc3/final; projected ~2026-05-28); Corosync 3.1.10; fapolicyd 1.5; CrowdSec 1.7.8; usbguard 1.1.4; QEMU 11.0.0 (no 11.0.1/11.1); libvirt 12.3.0 (12.4.0 still unreleased/in-progress); Mesa 26.1.1 / 26.0.7; ROCm 7.2.3; NVIDIA Container Toolkit 1.19.1; Waydroid 1.6.2; WSL 2.7.7 stable / 2.8.6 pre; FreeIPA 4.13.0 / SSSD 2.13.0 (zero new 2026 CVEs); Looking Glass B7 (master idle ~4 months — no commits since 2026-01-17; `module/` KVMFR patch still the lone 2025-03-04 commit); Gamescope 3.16.23 (#2018 + #2037 still open, no maintainer ack, HDR fix `7d4e835` still master-only); RTX 50-series VFIO passthrough still broken (RTX 4090 target unaffected); F45 Beta 2026-08-25 (ChangeSet unchanged — no composefs-sealed-images-specific Change).

**RESOLVED follow-up questions:**
- **"Which F44 kernel NVR first carries the DirtyDecrypt fix?"** (open since 2026-05-22) — RESOLVED enough: F44 `updates` ships `kernel-7.0.9-205.fc44`, which is past the affected range. Exact first-fixed NVR still not pinpointed (bodhi Anubis-gated), but any 7.0.x F44 build is safe.
- **"Does PR #392 land before the 2026-06-05 checkpoint?"** (open since 2026-05-19) — RESOLVED: **it merged 2026-05-24.** The remaining gate is the `ucore-hci` rebuild (new watch).

**Carried forward (still unresolved — out of research-only scope):** the code-inspection set (workflow `bootc-image-builder-action` migration; cosign binary pin in `42-cosign-policy.sh`; SELinux site-module path; KVMFR build-time signing — **now more urgent since the LTS base is moving to 6.18 and needs the `MODULE_IMPORT_NS` string-literal patch**; fapolicyd trust-DB rebuild timing + v1.5 rule-reload parity; GNOME 50 in stable-nvidia; AMD iGPU blacklist decision; K3s HA vs sqlite; NVIDIA driver branch pin; Mesa line in the running image). New unresolved: which exact F44 LTS-6.18 kernel NVR the rebuilt `ucore-hci` image will carry (must be ≥6.18.31).

**Files modified outside `.ai-context/`:** None. Strict scope compliance.

**Surprises:**
- **The 503-suffixed 2026-05-22 commits were actually on `origin/main` all along** — the local `origin/main` tracking ref was stale (showed `efdf712`), but a `git fetch` revealed `2ae02de`. So the "lost work" anxiety from the recurring git-receive-pack 503 was, this time, a false alarm at the remote level — the MCP per-file pushes had landed. The operational 503 risk (item 12) is real but the remote was consistent. Worth always running `git fetch` before assuming the remote is behind.
- **PR #392 went from 5+ days static to merged-with-73-green-checks in the 2-day window** — after ucore main had been idle 15 days. The stall broke abruptly; the maintainer (@bsherman) approved a drive-by contributor's (dylanmtaylor) PR. The single biggest remediation unblock of the entire research series — though MiOS still waits on the downstream `ucore-hci` bake.
- **F44's stable kernel is already on the 7.0.x line** (`7.0.9-205.fc44`), not 6.18.x — meaning the F44 *base* (as opposed to the longterm-6.18 LTS *flavor* PR #392 selects) is past the entire local-root cluster. The LTS flavor deliberately stays on longterm-6.18 (6.18.33) for server stability; both are now post-fix.

**Prior journal entries resolved or invalidated:** No prior entry outright invalidated. The 2026-05-22 entry's framing of "all ucore remediation levers frozen" is now superseded by the PR #392 merge (noted here, not edited there — journal is append-only). The DirtyDecrypt "date unconfirmed" hedge from the 2026-05-22 doc edit is resolved (2026-04-25).

**Next pass:** Per overwritten `NEXT-RESEARCH.md`. Tightest watches: (1) Secure Boot shim-16.1-6 in F44 (~33 days, checkpoint 2026-06-05); (2) **new `ucore-hci:stable-nvidia-lts` bake on the merged PR #392 base** + its kernel NVR (must be ≥6.18.31) — this is the gate for the kernel-CVE fix actually reaching MiOS; (3) GNOME 50.2 ship + NEWS; (4) Pacemaker 3.0.2 final (~2026-05-28).

**Drive mirror note:** Daily Drive snapshot uploaded as `CloudWS-bootc-research-2026-05-24.md` (Drive file id `1SZ85gaws1oSyC9v2rOpvtQ0O7RYOZQdj`). Same trade-off as prior passes: the knowledge doc is now ~90KB / ~39K tokens (per the Read tool's own count) — over the single-Read 25K-token cap — so the Drive file is a dated **snapshot index** (headline + action-item state + version table + next-pass watches + pointer to the git-authoritative full doc), not a verbatim copy. The git commit for this pass is the authoritative verbatim archive.

---

## 2026-05-26T12:00Z — `scheduled-research-daily`

**Agent ID:** `scheduled-research-daily`
**Context:** Seventh substantive pass. 2 days since the previous journaled pass (2026-05-24). Followed the `NEXT-RESEARCH.md` agenda (P0 reverify of 12 ACTION REQUIRED items → P1 Secure Boot → P2 ucore-hci bake → P3 GNOME 50.2 → P4 Pacemaker → P5 composefs → P6 image-builder-cli/#506 → P7 Podman 6.0 RC → P8 LG/Gamescope → P9 routine). Dispatched **three** parallel general-purpose research subagents: (A) Secure Boot + ucore-hci bake/issue #385 + kernel CVE cluster + cosign + NVIDIA; (B) GNOME/Pacemaker/composefs/image-builder-cli/Podman + bootc/OSTree/K3s/etcd/Ceph/CrowdSec/fapolicyd/systemd/Renovate/F45; (C) Looking Glass/Gamescope/QEMU/libvirt/Mesa/ROCm/NVIDIA-CT/Waydroid/WSL/FreeIPA-SSSD/RTX50.

**Repo-layout note (recurring):** This repo has **no `CLAUDE.md`, no `docs/changelogs/`, no top-level `CHANGELOG.md`** — the brief's expected files don't exist here. The SSOT is `INDEX.md` (+ `ENGINEERING.md`, `.cursorrules`, `.clinerules`). Hard invariants confirmed unchanged: USR-OVER-ETC, NO-MKDIR-IN-VAR, UNPRIVILEGED-QUADLETS (`User=`/`Group=`/`Delegate=yes`), BOOTC-NATIVE (`bootc container lint` passes), bash `set -euo pipefail`, kargs.d flat-array, NVIDIA VM-gating. Stayed strictly inside `.ai-context/`.

**Project state at time of run (unchanged):**
- Version: `v0.1.4` (still — no rev in 15 days).
- Base image: `ghcr.io/ublue-os/ucore-hci:stable-nvidia` (`image-versions.yml` line 12).
- Git: started on detached HEAD at `203838b` (the 2026-05-24 pass); local `main` was stale-tracked at `efdf712` (2026-05-20). A `git fetch origin main` revealed `origin/main` was **already at `203838b`** (the 2026-05-24 MCP per-file pushes had landed; the tracking ref was just stale — same false-alarm pattern as the prior pass). Checked out `main`, fast-forwarded clean to `203838b`, working tree clean before editing. (Reaffirms operational-risk item 12: always `git fetch` before assuming the remote is behind.)

**HEADLINE — the kernel-CVE remediation gate is CLOSED:**
- A fresh **`ucore-hci:stable-nvidia-lts-20260526`** bake now exists (built **2026-05-26T04:19Z**). Registry-manifest probe: `-20260526` and `-20260525` return HTTP 200; `-20260527` 404 → `-20260526` is newest. GHCR `tags/list` pagination is broken (returns only the first ~1000 alphabetical tags), so dated-tag probing by HEAD/manifest is the working enumeration method.
- Image config `org.opencontainers.image.revision = e63e21f` = `ublue-os/ucore` commit **#397 "use cosign login instead of docker login" (2026-05-25)** → built **on the post-#392-merge base**. FCOS base `version=44.20260419.3.1`, stream `stable`. The longterm-6.18 LTS flavor tracks kernel.org 6.18.x = **6.18.33** (≥ the 6.18.31 ssh-keysign-pwn floor and ≥ the 6.18.23 DirtyDecrypt floor) — so the bake closes the entire kernel local-root cluster.
- **The fix is now *deliverable* to MiOS but not yet *delivered*.** The project pins the *non-LTS* rolling `ucore-hci:stable-nvidia` by digest, so remediation arrives only via (a) a Renovate digest bump (if the rolling tag is also re-baking — NOT probed this pass) or (b) migrating the base-ref to `ghcr.io/ublue-os/ucore:stable-nvidia-lts`. Neither is auto-applied (research-only scope).
- **Issue #385 (Copy Fail tracker) is STILL OPEN** despite the fresh bake — the maintainers have not closed it. ucore main is active again (#397 on 2026-05-25).

**CHANGED upstream this 2-day window (knowledge-doc edits applied):**
1. **ucore-hci fresh bake** — items 1 + 8, §2 header + the #392 bullet, §6.5 header all updated (gate closed; project-side adoption pending).
2. **QEMU `v11.0.1` (2026-05-25)** — first 11.0.x point release, alongside stable backports `v10.2.3` + `v10.0.10` the same day. Lands during the publicly-disclosed CXL **"QEMUtiny"** window (oss-sec, ~2026-05-11): chained OOB-read in `GET_LOG` + OOB-write in `SET_FEATURE` in CXL Type-3 device emulation (vulnerable `SET_FEATURE` path introduced in 11.0.0). **Low practical exposure for MiOS** — CXL Type-3 memory-expander emulation is not part of a VFIO GPU-passthrough VM topology. No CVE-ID in tag metadata yet; qemu.org/blog has not posted the announcement. QEMU is a Fedora/ucore base package, not a project-owned pin. §9.4 header + body line updated.
3. **Gamescope `3.16.24` (2026-05-21)** — **correction**: the 2026-05-24 pass recorded "3.16.23 latest, no 3.16.24," but 3.16.24 had already tagged on 2026-05-21. It's a single-line changelog ("Revert 'build: add workaround to build with CMake 4.0'") — **NOT** the HDR fix. HDR fix `7d4e835` still master-only; issue #2018 still open/unacked. §10.1 header + body updated.
4. **image-builder-cli `v66` (2026-05-25)** — routine: profiling options (#516), osbuild/images 0.266.0 → 0.267.0 (#524) bringing new erofs/squashfs mount stages (#2348) + `container: resolve via skopeo` (#2346), ELN packit (#521). **No bootc/composefs/UKI/sealed-image work**; issue #506 still open. §1.3 updated.
5. **Renovate `v43.195.8` (2026-05-26)** — 8 routine patch releases since 43.195.0; no security advisory. §12.2 updated.
6. **kernel.org mainline → `7.1-rc5`** (was rc4); stable/longterm unchanged (7.0.10 / 6.18.33 / 6.12.91 / 6.6.141 / 6.1.174). F44 `updates` kernel unchanged at `7.0.9-205.fc44`. §6.5 / item 8 updated.

**Secure Boot (TIGHTEST DEADLINE) — re-verified, STILL ABSENT, no movement.** mirrors.kernel.org F44 `releases/.../os/Packages/s/` still has only `shim-x64/ia32-16.1-5` (+ `shim-unsigned-x64/ia32-16.1-1`); `updates/` and `updates/testing/` still carry **no shim package at all**. rhboot/shim upstream tip still tag `16.1` (2025-08-13). Cutover now **~31 days** out (2026-06-26); hard checkpoint 2026-06-05 now **~10 days** out. Item 4 + §7.2 updated. This remains the tightest unresolved risk on the board.

**NO CHANGE confirmed this window:** cosign 3.0.6 (no new GHSA/CVE); NVIDIA LTS 580.159.04 / feature 595.71.05 / dev-beta 595.44.09 (no new tag, no June bulletin — re-verified, item 3 noted); GNOME 50.2 (still not shipped — releng path shows 50.1 latest, tarball deadline passed); bootc 1.15.2; OSTree 2026.1; composefs 1.0.8 (no v1.1; bootc native backend still verbatim "Experimental … not yet suitable for production use"); Podman 5.8.2 (no 6.0 RC); K3s GA tags (05-20); etcd 3.6.11/3.5.30/3.4.44 + 3.7.0-beta.0; Ceph 20.2.1 (lower-confidence — ceph.io blog not re-fetched cleanly); Pacemaker 3.0.2-rc2 (no rc3/final; projected ~2026-05-28); Corosync 3.1.10; fapolicyd 1.5; CrowdSec 1.7.8; usbguard 1.1.4; systemd 260.1 (v261-rc1 unstable); libvirt 12.3.0 (12.4.0 still unreleased); Mesa 26.1.1/26.0.7; ROCm 7.2.3; NVIDIA Container Toolkit 1.19.1; Waydroid 1.6.2; WSL 2.7.7/2.8.6 (moderate confidence — GitHub releases page returned a stale snapshot to WebFetch); FreeIPA 4.13.0/SSSD 2.13.0 (no new 2026 CVE); Looking Glass B7 (master + `module/` KVMFR idle ~4 months — still a forward risk for the 6.18 base bump); RTX 50-series passthrough still broken (RTX 4090 unaffected); F45 Beta 2026-08-25 (no composefs-sealed-image Change).

**RESOLVED follow-up questions:**
- **"Has a new `ucore-hci:stable-nvidia-lts` bake appeared on the merged PR #392 base?"** (open since 2026-05-24) — **RESOLVED YES: `-20260526`, built 2026-05-26T04:19Z** on the post-#392 base. The P2 watch (the gate) is closed.
- **"Which kernel NVR will the rebuilt image carry — must be ≥6.18.31?"** — RESOLVED enough: the longterm-6.18 flavor = 6.18.33, comfortably past all cluster floors. No explicit kernel NVR label in the image config, so the exact NVR is confirmed by the longterm-6.18 line, not a package string.

**Carried forward (still unresolved — out of research-only scope):** the code-inspection set (CI `bootc-image-builder-action` migration; cosign binary pin in `42-cosign-policy.sh`; SELinux site-module path; `52-bake-kvmfr.sh` build-time signing + the `MODULE_IMPORT_NS` string-literal patch — **now imminent since the rebuilt LTS base is on longterm-6.18 and LG's `module/` has had no new commits in ~4 months, so the project's bake script must apply the patch itself**; fapolicyd v1.5 rule-reload/trust-DB parity at image-build; GNOME 50 in stable-nvidia; AMD iGPU blacklist decision; K3s HA vs sqlite; NVIDIA driver branch pin; Mesa line in the running image). **New unresolved:** (a) is the rolling `:stable-nvidia` tag (the one the project actually pins) also freshly re-baked, and is there an open Renovate digest-bump PR? (b) QEMU 11.0.1 "QEMUtiny" CVE-ID assignment.

**Files modified outside `.ai-context/`:** None. Strict scope compliance.

**Surprises:**
- **The bake gate closed within ~2 days of the PR #392 merge** — after 11–15 days of total stall, the ublue-os daily-bake machinery resumed and produced `-20260525` then `-20260526` back-to-back. The single biggest delivery-side unblock of the entire research series. MiOS now genuinely *can* pull a fixed kernel; the only thing between exposure and remediation is a project-side digest bump or base-ref migration.
- **Issue #385 stayed open through the fresh bake** — maintainers gate-close on verification, not on "an image exists." Consistent with the 2026-05-24 prediction.
- **GHCR `tags/list` is unreliable for enumeration** (first ~1000 alphabetical tags only) — dated-tag manifest/HEAD probing (`-YYYYMMDD` 200 vs 404) is the working method to find the newest bake. Worth remembering for the next "is there a new tag?" question.
- **The 2026-05-24 pass missed Gamescope 3.16.24** (tagged 2026-05-21, three days before that pass recorded "3.16.23 latest"). A reminder that "latest tag" checks can lag the actual tag list by a few days; corrected this pass. The substantive point (HDR fix unreleased) is unchanged.
- **QEMU shipped 11.0.1 + two stable backports in one day (2026-05-25)** with no blog post and no CVE-ID in tag metadata — the QEMUtiny/CXL framing came from oss-sec, not QEMU's own channels yet. Another "release notes/metadata are not a reliable CVE signal" datapoint (cf. the K3s silent-`replace` finding).

**Prior journal entries resolved or invalidated:** No prior entry outright invalidated. The 2026-05-24 entry's framing of "no rebuilt `ucore-hci` tag confirmed yet / the bake is the remaining gate" is now superseded by the `-20260526` bake (noted here, not edited there — journal is append-only).

**Next pass:** Per overwritten `NEXT-RESEARCH.md`. Tightest watches: (1) Secure Boot shim-16.1-6 in F44 (checkpoint 2026-06-05, ~10 days — TIGHTEST); (2) project-side adoption of the `-lts-20260526` bake (rolling `:stable-nvidia` re-bake? Renovate digest-bump PR? issue #385 close?); (3) GNOME 50.2 ship + NEWS (imminent); (4) Pacemaker 3.0.2 final (~2026-05-28); (5) QEMU 11.0.1 QEMUtiny CVE-ID.

**Drive mirror note:** Daily Drive snapshot uploaded as `CloudWS-bootc-research-2026-05-26.md` (Drive file id `1WiYh28_eNO8ab-7Lc7_uoAKu1uUO1h2b`). Same trade-off as prior passes: the knowledge doc is now ~95KB — over the single-Read 25K-token cap, and verbatim round-trip into one MCP text parameter proved fragile in earlier passes — so the Drive file is a dated **snapshot index** (headline + action-item state + changed/no-change deltas + next-pass watches + pointer to the git-authoritative full doc), not a verbatim copy. The git commit for this pass is the authoritative verbatim archive.

---

## 2026-05-27T12:00Z — `scheduled-research-daily`

**Agent ID:** `scheduled-research-daily`
**Context:** Eighth substantive pass. 1 day since the previous journaled pass (2026-05-26). Followed the `NEXT-RESEARCH.md` agenda (P0 reverify of 12 ACTION REQUIRED items → P1 Secure Boot → P2 project-side bake adoption → P3 GNOME 50.2 → P4 Pacemaker 3.0.2 → P5 QEMU QEMUtiny CVE-ID → P6 composefs → P7 image-builder-cli/#506 → P8 Podman 6.0 RC + LG/Gamescope → P9 routine). Dispatched **three** parallel general-purpose research subagents: (A) Secure Boot + ucore-hci bakes/rolling-tag/issue #385 + kernel CVE cluster + cosign + NVIDIA; (B) GNOME/Pacemaker/composefs/image-builder-cli/Podman + bootc/OSTree/K3s/etcd/Ceph/CrowdSec/fapolicyd/systemd/Renovate/F45; (C) QEMU/Looking Glass/Gamescope/libvirt/Mesa/ROCm/NVIDIA-CT/Waydroid/WSL/FreeIPA-SSSD/RTX50. **Note:** subagent C's first launch tripped a cyber-content safeguard on vulnerability-mechanics phrasing (QEMU CVE / VFIO); re-dispatched with a version-tracking/patch-status framing and it completed cleanly. Worth remembering: frame upstream-CVE research as "latest version + patched-version + CVE-ID assignment," not exploit mechanics.

**Repo-layout note (recurring):** This repo still has **no `CLAUDE.md`, no `docs/changelogs/`, no top-level `CHANGELOG.md`** — the brief's expected files don't exist here. SSOT is `INDEX.md` (+ `ENGINEERING.md`, `ARCHITECTURE.md`, `.cursorrules`, `.clinerules`). Hard invariants confirmed unchanged: USR-OVER-ETC, NO-MKDIR-IN-VAR, UNPRIVILEGED-QUADLETS (`User=`/`Group=`/`Delegate=yes`), BOOTC-NATIVE (`bootc container lint` passes), bash `set -euo pipefail`, kargs.d flat-array, NVIDIA VM-gating. Stayed strictly inside `.ai-context/`.

**Project state at time of run (unchanged):**
- Version: `v0.1.4` (still — no rev in 16 days).
- Base image: `ghcr.io/ublue-os/ucore-hci:stable-nvidia` (`image-versions.yml` line 12).
- Git: started on detached HEAD at `dae2091` (the 2026-05-26 pass, knowledge-doc commit 3/3); local `main` was stale-tracked at `efdf712` (2026-05-20). A `git fetch origin main` revealed `origin/main` was **already at `dae2091`** (the 2026-05-26 MCP per-file pushes had landed; tracking ref was simply stale — same false-alarm pattern as the prior two passes). Checked out `main`, fast-forwarded clean (efdf712..dae2091, 8 commits), working tree clean before editing. (Reaffirms operational-risk item 12: always `git fetch` before assuming the remote is behind.)

**HEADLINE — the kernel-CVE fix is now DELIVERABLE on the tag MiOS actually pins:**
- The standing open question ("is the *rolling* `:stable-nvidia` tag — the one the project pins — also being re-baked, or only the dated `-lts` tags?") is **RESOLVED: YES, the rolling tag is re-baked daily.** Registry-manifest probe today: rolling `ghcr.io/ublue-os/ucore-hci:stable-nvidia` was rebaked **2026-05-27T04:13:01Z** (index digest `sha256:bf999c72…`, amd64 child `sha256:1f44a959…`) on image revision `e63e21f` = ucore commit **#397** ("use cosign login instead of docker login", 2026-05-25) — the **post-#392-merge base**. It was built from the *same source in the same run* as the dated LTS tag, which advanced to `stable-nvidia-lts-20260527` (baked 04:12:59Z, minutes earlier; `-20260528` still 404).
- **So both the rolling `:stable-nvidia` and the dated `:stable-nvidia-lts` now carry the longterm-6.18 kernel (6.18.33 ≥ all cluster floors).** The kernel-CVE cluster fix (Copy Fail / DirtyDecrypt / ssh-keysign-pwn / Dirty-Frag) is now reachable on the very tag MiOS pins. **The only remaining gate is project-side** — a Renovate digest bump of the `:stable-nvidia` pin in `image-versions.yml` line 12 (or the `:stable-nvidia-lts` base-ref migration). Neither is auto-applied (research-only scope); whether Renovate has opened a digest-bump PR can't be inspected cross-repo.
- **Caveat:** the OCI image config exposes no kernel-NVR label — `org.opencontainers.image.version = 44.20260419.3.1` is the FCOS base-stream version, NOT the kernel. The ≥6.18.33 kernel is inferred from the longterm-6.18 flavor line, not a manifest package string. To pin the exact NVR you'd have to pull and inspect the rootfs.
- **Issue #385 (Copy Fail tracker) STILL OPEN** despite the fresh bakes — maintainers gate-close on verification, not on existence (consistent with the 2026-05-24/26 prediction). ucore main HEAD unchanged at `e63e21f` (#397, 2026-05-25).

**CHANGED upstream this 24h window (knowledge-doc edits applied):**
1. **Rolling `:stable-nvidia` daily re-bake confirmed** — items 1 + 8, §2 header, §6.5 header all updated (delivery gate now project-side).
2. **NVIDIA new-feature branch `610.43.02` (2026-05-26)** — supersedes the 595.44.x dev-beta line; adds Vulkan multi-GPU logical devices, FP16 EGL on Wayland, DRM multiplanar YCbCr modifiers, per-plane DRM color-pipeline API for Linux 6.19. **NOT LTS / production-feature / beta → not a project pin; informational only.** No June bulletin (latest still May `a_id/5821`). §8.1 header + a new Feature-Branches bullet + item 3 updated.
3. **systemd stable v260.2 (2026-05-27)** + unstable v261-rc2 (2026-05-26, rc1 2026-05-24). The v260.2 patch is on the project's stable base line — no security content flagged by upstream. §12.3 updated.
4. **Renovate v43.196.1 (2026-05-27)** — routine; maven-range fix + a simple-git security-dep bump at 43.195.12. §12.2 updated.
5. **WSL pre-release 2.8.7 (2026-05-26)** — stable unchanged at 2.7.7. §11.3 updated.

**CORRECTION — stale baseline fixed:**
- **FreeIPA: 4.13.0 → 4.13.1** (git tag `release-4-13-1`, **2026-01-16**). The doc had recorded 4.13.0 since the bootstrap pass; 4.13.1 has actually been the latest tag since mid-January. FreeIPA publishes via git tags, not GitHub Releases (so the releases-page check missed it). §11.1 corrected with a date for 4.13.0 (2025-12-04) too. Not new this window — a long-standing baseline error caught this pass.

**Secure Boot (TIGHTEST DEADLINE) — re-verified, STILL ABSENT, no movement (fourth pass running).** mirrors.kernel.org F44 `releases/.../os/Packages/s/` still has only `shim-x64/ia32-16.1-5` (+ `shim-unsigned-*-16.1-1`); `updates/` and `updates/testing/` still carry **no shim package at all**. rhboot/shim upstream tip still tag `16.1` (2025-08-13). Cutover now **~29 days** out (2026-06-26); hard checkpoint 2026-06-05 now **~9 days** out. Item 4 + §7.2 updated. Tightest unresolved risk on the board.

**RESOLVED follow-up questions:**
- **"Is the rolling `:stable-nvidia` tag also freshly re-baked?"** (open since 2026-05-26) — **RESOLVED YES**: re-baked 2026-05-27T04:13Z on the post-#392 base. The kernel-CVE fix is now deliverable on the pinned tag. (The companion "is there an open Renovate digest-bump PR?" remains out of research-only scope.)
- **"QEMU 11.0.1 QEMUtiny CVE-ID assignment?"** (open since 2026-05-26) — **answered (negative)**: still NO published CVE-ID; the only public reference remains the oss-sec advisory (2026-05-20). Nothing to enter in the CVE column. Kept on the watch list until/unless a CVE-ID is assigned.

**Carried forward (still unresolved — out of research-only scope):** the code-inspection set (CI `bootc-image-builder-action` migration; cosign binary pin in `automation/42-cosign-policy.sh`; SELinux site-module path; `52-bake-kvmfr.sh` build-time signing + the `MODULE_IMPORT_NS` string-literal patch — **imminent: the rebuilt LTS/rolling base is now on longterm-6.18 and LG's `module/` has had no new commits in ~4 months, so the project's bake script must carry the patch itself**; fapolicyd v1.5 rule-reload/trust-DB parity at image-build; GNOME 50 in stable-nvidia; AMD iGPU blacklist decision; K3s HA vs sqlite; NVIDIA driver branch pin; Mesa line in the running image). **New unresolved:** has Renovate opened a digest-bump PR for the `:stable-nvidia` pin now that the rolling tag re-bakes daily? (cross-repo / CI inspection — out of scope).

**NO CHANGE confirmed this window:** cosign 3.0.6 (no new GHSA/CVE; newest advisory still GHSA-w6c6-c85g-mmv6 2026-04-06); kernel CVE cluster (no new disclosure 05-26→27); kernel.org (mainline 7.1-rc5; stable 7.0.10; longterm 6.18.33/6.12.91/6.6.141/6.1.174); RHSB-2026-003 ("Ongoing/Important", last updated 2026-05-21); CVE-2026-31431 (7.8 / CISA KEV); NVIDIA LTS 580.159.04 / production-feature 595.71.05; NVIDIA Container Toolkit 1.19.1; bootc 1.15.2; OSTree 2026.1; composefs 1.0.8 (no v1.1; main at a7ea6cd 2026-05-19; bootc native backend still verbatim "Experimental … not yet suitable for production use"); image-builder-cli v66 (#506 still open, no movement; BIB active, no deprecation; osbuild.org/blog still 404); Podman 5.8.2 (no 6.0 RC/beta; Quadlet `podman-systemd.unit.5.md` still has no 6.0 schema-delta); K3s GA tags (05-20); etcd 3.7.0-beta.0 (no GA/rc); Ceph 20.2.1 (no 20.2.2/bulletin); CrowdSec 1.7.8; fapolicyd 1.5; usbguard 1.1.4; Pacemaker 3.0.2-rc2 (no rc3/final — a web-search "3.0.2 final 2026-04-23" hit was a hallucination, ATOM feed authoritative, disregarded); Corosync 3.1.10; QEMU 11.0.1 (no 11.0.2/11.1; no blog post; QEMUtiny still no CVE-ID); libvirt 12.3.0 (12.4.0 still unreleased/in-progress); Mesa 26.1.1 / 26.0.7; ROCm 7.2.3; Waydroid 1.6.2; SSSD 2.13.0; Looking Glass B7 (master idle since 2026-01-17; `module/` last code change 2025-03-04 + a no-op doc commit 2025-03-05 — still a forward risk for the 6.18 base); Gamescope 3.16.24 (#2018 still open/unacked; HDR fix 7d4e835 still master-only); RTX 50-series passthrough still broken (RTX 4090 unaffected); F45 Beta 2026-08-25 (no composefs-sealed-image Change).

**Files modified outside `.ai-context/`:** None. Strict scope compliance.

**Surprises:**
- **The biggest delivery-side unblock of the entire series is now complete:** not only does a fixed dated LTS bake exist (2026-05-26), but the *rolling* `:stable-nvidia` tag the project actually pins is re-baking daily on the post-#392 base. MiOS no longer waits on upstream for anything — the kernel-CVE remediation is one Renovate digest bump away. After 11–15 days of total ucore stall earlier this month, the delivery path is fully open.
- **A research subagent tripped a cyber-content safeguard** purely on vulnerability-mechanics phrasing (OOB-read/write, exploit framing) while doing routine defensive patch-tracking. Re-framing to "latest version / patched version / CVE-ID assignment" cleared it. A reusable lesson for this scheduled job: keep CVE research framed as defensive version-tracking, not exploitation detail.
- **FreeIPA had been wrong since bootstrap** (4.13.0 vs the actual 4.13.1 from 2026-01-16) — the releases-page check missed it because FreeIPA ships via git tags only. Another "anchor on the authoritative tag source, not the GitHub Releases page" datapoint (cf. the prior Mesa/QEMU/libvirt stale-baseline corrections).

**Prior journal entries resolved or invalidated:** No prior entry outright invalidated. The 2026-05-26 entry's open caveat ("the rolling `:stable-nvidia` tag was not re-checked this pass") is now superseded by today's confirmed daily re-bake (noted here, not edited there — journal is append-only).

**Next pass:** Per overwritten `NEXT-RESEARCH.md`. Tightest watches: (1) Secure Boot shim-16.1-6 in F44 (checkpoint 2026-06-05, ~9 days — TIGHTEST, unchanged four passes); (2) any Renovate digest-bump PR for the `:stable-nvidia` pin / project adoption of the fixed base; (3) GNOME 50.2 ship + NEWS (now 4 days overdue); (4) Pacemaker 3.0.2 final (~2026-05-28, imminent); (5) QEMU QEMUtiny CVE-ID assignment.

**Drive mirror note:** Daily Drive snapshot uploaded as `CloudWS-bootc-research-2026-05-27.md` (Drive file id `1Ucyj3Y3Zup2d9v_241LOsZRsxfjXAHcK`). Same trade-off as prior passes: the knowledge doc is now ~100KB — over the single-Read 25K-token cap, and verbatim round-trip into one MCP text parameter proved fragile in earlier passes — so the Drive file is a dated **snapshot index** (headline + action-item state + changed/no-change deltas + next-pass watches + pointer to the git-authoritative full doc), not a verbatim copy. The git commit for this pass is the authoritative verbatim archive.
