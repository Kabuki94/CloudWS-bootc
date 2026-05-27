# NEXT-RESEARCH — agenda for next scheduled-research pass

> Written by `scheduled-research-daily` on 2026-05-27 (UTC). Tomorrow's run should start here, then overwrite this file with its own next-day agenda.

> **Repo-layout reminder:** this repo has NO `CLAUDE.md`, NO `docs/changelogs/`, NO top-level `CHANGELOG.md`. The SSOT is `INDEX.md` (+ `ENGINEERING.md`, `ARCHITECTURE.md`, `.cursorrules`, `.clinerules`). Hard invariants: USR-OVER-ETC, NO-MKDIR-IN-VAR, UNPRIVILEGED-QUADLETS (`User=`/`Group=`/`Delegate=yes`), BOOTC-NATIVE (`bootc container lint` passes), bash `set -euo pipefail`, kargs.d flat-array, NVIDIA VM-gating. Research-only scope: touch only `.ai-context/`.

> **Research-tooling reminder:** (a) Frame all CVE/security research as *defensive version-tracking* — "latest version / patched version / CVE-ID assignment status" — NOT exploit mechanics; a subagent tripped a cyber-content safeguard on 2026-05-27 with OOB-read/write phrasing. (b) GHCR `tags/list` pagination is broken (first ~1000 alphabetical tags only); enumerate dated container tags by HEAD/manifest probe (`-YYYYMMDD` 200 vs 404). To inspect a GHCR manifest unauth: `curl -s "https://ghcr.io/token?scope=repository:ublue-os/ucore-hci:pull"` → use the token against `https://ghcr.io/v2/ublue-os/ucore-hci/manifests/<tag>` with the OCI index Accept header. (c) GitHub issue *comment* bodies render via JS (invisible to WebFetch); only title/body/state are readable. (d) `download.gnome.org` autoindex pages 404 through WebFetch — read them with `curl` via Bash. (e) Prefer GitHub `.atom` feeds over HTML release pages for reliable dates. (f) Always `git fetch` before assuming the remote is behind — the local `origin/main` tracking ref has been stale for four consecutive passes while the MCP per-file pushes had actually landed.

---

## ACTION REQUIRED items (carry forward until resolved)

These are upstream signals that imply a build-breaking or security change to the project. **The research agent never applies these.** They are surfaced here for human review and follow-up.

1. **ACTION REQUIRED (remediation now FULLY DELIVERABLE — only a project-side digest bump remains): kernel local-root CVE cluster.** **NEW 2026-05-27: the delivery gate is wide open.** The *rolling* `ucore-hci:stable-nvidia` tag — the exact tag MiOS pins in `image-versions.yml` line 12 — **is being re-baked daily** on the post-#392-merge base. Registry probe 2026-05-27: rolling `:stable-nvidia` rebaked **2026-05-27T04:13:01Z** (index `sha256:bf999c72…`, amd64 child `sha256:1f44a959…`), revision `e63e21f` = ucore commit #397, same source/run as the dated `:stable-nvidia-lts-20260527` (baked 04:12:59Z). Both carry the longterm-6.18 kernel (6.18.33 ≥ all cluster floors). **So the kernel-CVE fix reaches MiOS the moment Renovate bumps the `:stable-nvidia` digest pin** (or the project migrates to `:stable-nvidia-lts`, item 2) — no more upstream waiting. Issue #385 (Copy Fail tracker) is **still open** despite the fixed bakes. The cluster: **`CVE-2026-31431` "Copy Fail"** (algif_aead AF_ALG LPE, CVSS 7.8, CISA KEV — federal deadline 2026-05-15 passed; fixed 6.18.22/6.19.12/7.0); **`CVE-2026-31635` "DirtyDecrypt"** (RxGK, CVSS 7.5, CONFIG_RXGK=Fedora; fixed 6.18.23/6.19.13/7.0); **`CVE-2026-46333` "ssh-keysign-pwn"** (ptrace exit-race → LPE; fixed 7.0.8/6.18.31/6.12.89); **`CVE-2026-43500`+`CVE-2026-43284`+`CVE-2026-46300` "Dirty Frag" (RHSB-2026-003)** — still "Ongoing/Important." Defense-in-depth the owner *could* set while adopting the bake (config the project owns — kargs/modprobe.d): `modprobe.blacklist=algif_aead algif_skcipher algif_hash algif_rng` (Copy Fail); `modprobe.blacklist=rxrpc` (DirtyDecrypt RxGK + Dirty-Frag RxRPC path, if AFS/rxrpc unused — almost certainly is); `kernel.dmesg_restrict=1`. **Caveat:** the OCI config carries no kernel-NVR label (`44.20260419.3.1` = FCOS stream, not kernel); the ≥6.18.33 kernel is inferred from the longterm-6.18 flavor, not a manifest string — confirm the baked NVR by rootfs inspection if exactness is required at adoption.

2. **ACTION REQUIRED: Base image on a deleted upstream repo + base-ref migration.** `Containerfile` / `image-versions.yml` (line 12) still reference `ghcr.io/ublue-os/ucore-hci:stable-nvidia`; `bsherman/ucore-hci` is HTTP 404. Migration target remains `ghcr.io/ublue-os/ucore:stable-nvidia-lts`. Requires hand-editing `renovate.json` `customManagers` regex `depName` + `image-versions.yml` line 12 `depName`. Cannot be done by Renovate automerge. **Note (2026-05-27):** because the *rolling* `:stable-nvidia` pin now re-bakes daily with the fixed kernel (item 1), a plain Renovate digest bump on the *current* pin already delivers the fix — the `-lts` migration is the more robust long-term fix but is no longer strictly required to close the kernel cluster.

3. **ACTION REQUIRED: Pin cosign ≥ 3.0.6** wherever the project verifies signatures. GHSAs (`GHSA-w6c6-c85g-mmv6` = CVE-2026-39395 CVSS 4.3, `GHSA-wfqv-66vq-46rm`, `GHSA-whqx-f9j3-ch6m`) fixed in 3.0.6 / 2.6.3; `CVE-2026-22703` (Rekor not bound to artifact) fixed in 2.6.2 / 3.0.4. cosign latest still **v3.0.6 (2026-04-06)** — no newer release, no new GHSA (re-verified 2026-05-27). Verify the baked cosign binary + `automation/42-cosign-policy.sh` + CI passes `--bundle` (v3 requires it where v2 made it optional).

4. **ACTION REQUIRED (TIGHTEST HARD DEADLINE): Secure Boot 2023-CA shim refresh before 2026-06-26 (~29 days).** MS 2011 CA stops signing then. **F44 stable + updates + updates-testing ALL still ship only `shim-16.1-5` (2021-key) — `shim-16.1-6` (2023-key) is absent everywhere** (re-verified 2026-05-27 via mirrors.kernel.org `releases/` + `updates/` + `updates/testing/`). rhboot/shim upstream tip still `16.1` (2025-08-13) — the `-6` suffix is Fedora's package release. **Hard checkpoint: 2026-06-05 (~9 days)** — if still 16.1-5 by then, MiOS needs a fallback (apply MS DBX update via `fwupdmgr` on target hardware). No movement in four passes.

5. **ACTION REQUIRED (RESOLVED-as-satisfied): NVIDIA kmod pin.** May 2026 bulletin `a_id/5821` floors satisfied by project pins (LTS 580.159.04 ≥ 580.159.03; production-feature 595.71.05 = R595 floor). Re-verified 2026-05-27: no June bulletin. **NEW: a new-feature-branch driver `610.43.02` shipped 2026-05-26** — NOT LTS/production/beta, **not a project pin** (informational; Vulkan multi-GPU + DRM color-pipeline for Linux 6.19). Only remaining action: confirm the `ucore-hci` bake the project adopts carries NVIDIA NVR ≥ 580.159.03 (it pins the LTS line, not 610.x).

6. **ACTION REQUIRED: Migrate from `ublue-os/bootc-image-builder-action` to `osbuild/bootc-image-builder-action`** if CI uses the former (verify in `.github/workflows/` — still not inspected; research-only scope). osbuild fork actively maintained; ublue-os fork maintenance-mode.

7. **ACTION REQUIRED: Fix `osautomation` → `osbuild` typo + zero digest in `image-versions.yml` line 21.** `osautomation` GitHub user has zero repos/packages; `image_builder_cli_digest` is all-zeros. Reference should be `ghcr.io/osbuild/image-builder-cli`. Trivial hand-fix.

8. **ACTION REQUIRED (actionable, positive): Bump the K3s pin to a 2026-05-20 GA tag.** K3s GA shipped 2026-05-20 (v1.36.1+k3s1 / v1.35.5+k3s1 / v1.34.8+k3s1 / v1.33.12+k3s1). These remediate **CVE-2026-33186** (grpc-go authz bypass, CVSS 9.1) via a `replace google.golang.org/grpc => v1.79.3` directive in go.mod (release notes are silent — the binary is patched regardless). Pick the GA tag matching the project's minor line. No newer patch as of 2026-05-27.

9. **ACTION REQUIRED (validate before pinning): fapolicyd v1.5 (2026-05-20).** First minor bump off 1.4.x (transactional rule reload, `--check-rules`, per-rule hit counters). Since fapolicyd is a deny-by-default execution gate, **validate the trust DB / rules still load identically at image-build time before pinning v1.5.** No v1.5.x yet.

10. **ACTION REQUIRED (F45-paced, ~Oct 2026): Podman 6.0 GA slipped to Fedora 45.** No 6.0 RC tag cut (latest stable v5.8.2; Quadlet `podman-systemd.unit.5.md` still has no 6.0 schema-delta content). Pre-flight Quadlet review still required (BoltDB→SQLite, slirp4netns→Pasta, cgroups v1 removal, netavark iptables→nftables).

11. **ACTION REQUIRED (migration possible, partial parity): `bootc-image-builder` → `image-builder-cli`.** v66 (2026-05-25) added profiling + erofs/squashfs mount stages + skopeo container resolution — but **no bootc/composefs/UKI/sealed-image work**; public docs still enumerate only qcow2 + bootc-installer ISO. BIB still has the wider matrix and remains active (no deprecation timeline). Viable for qcow2/ISO only until full parity (issue #506 = upstream tracker, still open, no movement since 2026-04-29).

12. **OPERATIONAL RISK (de-escalated, keep watching): git-receive-pack HTTP 503 + stale tracking ref.** For the FOURTH consecutive pass, the local `origin/main` tracking ref was stale (showed `efdf712`) while the remote actually had the latest MCP-pushed commits (`dae2091`). **Always `git fetch` before assuming the remote is behind.** Keep the per-file-MCP / single-commit-batching fallback in mind if a real 503 recurs on `git push`.

---

## Priority topics for tomorrow's pass (2026-05-28)

Ordered by descending value. Rationale under each.

### P0 — Re-verify all ACTION REQUIRED items

Touch each upstream link to see if anything shifted in 24h. **Tightest: (a) Secure Boot shim-16.1-6 in F44 (~28 days to cutover, checkpoint 2026-06-05 = ~8 days); (b) project-side adoption — is there an open Renovate digest-bump PR for the `:stable-nvidia` pin, did issue #385 close, did the rolling tag bake again on 2026-05-28?** If any resolve, strike them and note resolution in `ai-journal.md`.

- shim-16.1-6 in F44 (Anubis-free): https://mirrors.kernel.org/fedora/releases/44/Everything/x86_64/os/Packages/s/ (+ `updates/44/...` + `updates/testing/44/...`)
- ucore-hci GHCR tags (manifest-probe `-YYYYMMDD`; check the rolling `:stable-nvidia` digest vs the one in `image-versions.yml`): https://github.com/ublue-os/ucore/pkgs/container/ucore-hci — probe for `stable-nvidia-lts-20260528`+ and re-check the rolling `:stable-nvidia` `created`/`revision`
- `ublue-os/ucore` main: https://github.com/ublue-os/ucore/commits/main
- issue #385: https://github.com/ublue-os/ucore/issues/385 (still open post-bake)
- cosign: https://github.com/sigstore/cosign/releases
- NVIDIA drivers: https://github.com/NVIDIA/open-gpu-kernel-modules/releases

### P1 — Secure Boot shim-16.1-6 in F44 (TIGHTEST hard-calendar deadline)

*Why:* 2026-06-26 cutover is ~28 days; hard checkpoint 2026-06-05 is **~8 days** out. Verified-absent across all three F44 trees for four passes running. **Flag escalation if still 16.1-5 by 2026-06-05.**

*Specific questions:* fetch the three mirrors.kernel.org paths; grep `shim-`; has 16.1-6 landed in stable or updates-testing? Does Fedora's multi-signed shim auto-roll on bootc upgrade or require explicit `fwupdmgr`?

### P2 — Project-side adoption of the fixed base (the rolling re-bake is now confirmed daily)

*Why:* As of 2026-05-27 the *rolling* `:stable-nvidia` tag MiOS pins re-bakes daily on the fixed longterm-6.18 base — the upstream gate is fully closed. The remaining gate is purely project-side: a Renovate digest bump (or the `:stable-nvidia-lts` migration). The kernel-CVE fix is one digest bump from reaching MiOS.

*Specific questions:* Has the rolling `:stable-nvidia` baked again today (probe `created`/`revision`)? Is there an open Renovate digest-bump PR for `image-versions.yml` line 12 (cannot inspect cross-repo via MCP — note as out-of-scope)? Did issue #385 close once the bake was verified? Probe for `stable-nvidia-lts-20260528`+ to confirm the daily cadence is sustained.

### P3 — Pacemaker 3.0.2 final (IMMINENT — projected ~2026-05-28 = tomorrow)

*Why:* stuck at 3.0.2-rc2 since 2026-05-11 (~17 days); rc1→rc2 gap was ~17 days, so a final or rc3 is due around 2026-05-28. Check for the final tag and read its release notes for any TLS / Pacemaker-Remote / CIB-admin change relevant to the project's HA stack. **Ignore web-search "final" claims — a 2026-04-23 final hit was a hallucination; the ATOM feed is authoritative.**

*Anchor:* https://github.com/ClusterLabs/pacemaker/releases.atom

### P4 — GNOME 50.2 ship (OVERDUE — tarball deadline 2026-05-23, now 4 days past)

*Why:* release expected any day; bugfix-only expected. Confirm it shipped and read 50.2 NEWS for any Mutter / gnome-remote-desktop / NVIDIA explicit-sync / HDR backport. **Use `curl` via Bash on the download server (autoindex 404s through WebFetch).**

*Anchors:* https://download.gnome.org/sources/mutter/50/ , https://download.gnome.org/sources/gnome-remote-desktop/50/ , https://download.gnome.org/teams/releng/ , https://release.gnome.org/calendar/

### P5 — QEMU QEMUtiny CVE-ID + any 11.0.2/11.1

*Why:* 11.0.1 shipped 2026-05-25 (+10.2.3/10.0.10 backports) with no CVE-ID and no blog post; the CXL "QEMUtiny" framing comes only from the oss-sec advisory (2026-05-20). Low MiOS exposure (no CXL Type-3 emulation in a GPU-passthrough topology), but confirm the CVE assignment when it lands.

*Anchors:* https://www.qemu.org/blog/, https://gitlab.com/qemu-project/qemu/-/tags, https://seclists.org/oss-sec/2026/q2/618

### P6 — composefs v1.1 tag + bootc native-backend GA

*Why:* main active (last commit a7ea6cd, PR #436 hardlinked-whiteout fix, 2026-05-19) but still no tag in 16.5 months; bootc still flags the native composefs backend verbatim "Experimental … not yet suitable for production use." A v1.1 cut + bootc dropping the experimental framing is a significant on-disk-format event and gates the F45 sealed-image direction.

*Anchors:* https://github.com/composefs/composefs/releases, https://github.com/composefs/composefs/commits/main, https://bootc.dev/bootc/experimental-composefs.html

### P7 — image-builder-cli parity + issue #506

*Why:* v66 (2026-05-25) added no new image formats and no bootc/composefs work. Check v67+, BIB deprecation timeline, issue #506 (composefs+UKI sealed-image backend). osbuild.org/blog still 404 — retry.

*Anchors:* https://github.com/osbuild/image-builder-cli/releases, https://github.com/osbuild/image-builder-cli/issues/506, https://github.com/osbuild/bootc-image-builder/issues

### P8 — Podman 6.0 RC tag watch (F45-paced) + Looking Glass / Gamescope HDR

*Why:* Podman GA slip relieves pressure but an RC tag will eventually drop with Quadlet schema deltas. **LG matters more now** — the project's base is on longterm-6.18, so the KVMFR build patch (§9.3 — `MODULE_IMPORT_NS` string-literal + `vmalloc.h`) is mandatory for the bake, and LG's `module/` has had no code commit since 2025-03-04 (master idle since 2026-01-17). Gamescope HDR fix `7d4e835` still master-only; #2018 still open, no maintainer ack.

*Anchors:* https://github.com/containers/podman/releases, https://github.com/gnif/LookingGlass/commits/master/module, https://github.com/ValveSoftware/gamescope/tags, https://github.com/ValveSoftware/gamescope/issues/2018

### P9 — Routine version watches (low priority)

Mesa 26.0.8 / 26.1.2; libvirt 12.4.0 release; ROCm post-7.2.3; NVIDIA Container Toolkit post-1.19.1; WSL post-2.7.7 stable / post-2.8.7 pre-release; etcd 3.7.0 GA progress (beta.0 = 2026-05-19); Ceph 20.2.2 / Squid bulletins (re-fetch ceph.io blog); CrowdSec post-1.7.8; fapolicyd post-1.5; systemd v260.3 / v261-rc progression (stable now 260.2 = 2026-05-27); Renovate cadence (now 43.196.1); bootc post-1.15.2; OSTree post-2026.1; Waydroid post-1.6.2; FreeIPA post-4.13.1 / SSSD post-2.13.0; K3s post-GA patch; NVIDIA new-feature 610.x progression (not a pin).

---

## Upstream releases + CVE feeds to monitor

| Source | What to check |
| ------ | ------------- |
| https://mirrors.kernel.org/fedora/releases/44/Everything/x86_64/os/Packages/s/ | **shim-16.1-6 F44 promotion (Anubis-free) — TIGHTEST DEADLINE** (+ updates / updates/testing) |
| https://github.com/ublue-os/ucore/pkgs/container/ucore-hci | **rolling `:stable-nvidia` daily re-bake (now confirmed) + new `-lts-YYYYMMDD` tags** (manifest-probe) |
| https://github.com/ublue-os/ucore/commits/main | upstream activity (HEAD #397 / `e63e21f`, 2026-05-25) |
| https://github.com/ublue-os/ucore/issues/385 | Copy Fail tracker (still open post-bake) |
| https://www.kernel.org/ | stable/longterm (7.0.10 / 6.18.33 / 6.12.91 / 6.6.141 / 6.1.174; mainline 7.1-rc5) |
| https://access.redhat.com/security/vulnerabilities/RHSB-2026-003 | Dirty Frag (CVE-2026-43284/-43500/-46300) — still "Ongoing/Important" |
| https://access.redhat.com/security/cve/cve-2026-31431 | Copy Fail — CISA KEV |
| https://github.com/sigstore/cosign/releases | post-3.0.6 |
| https://github.com/NVIDIA/open-gpu-kernel-modules/releases | post-580.159.04 / 595.71.05; new-feature 610.43.02 (not a pin); June bulletin watch |
| https://github.com/k3s-io/k3s/releases | post-GA patch (v1.36.1 / v1.35.5 / v1.34.8 / v1.33.12 shipped 2026-05-20) |
| https://github.com/ClusterLabs/pacemaker/releases | **3.0.2 final — IMMINENT** (rc2 ~17 days old; projected ~2026-05-28) |
| https://gitlab.com/qemu-project/qemu/-/tags | post-11.0.1; **QEMUtiny CVE-ID watch** |
| https://seclists.org/oss-sec/2026/q2/618 | QEMUtiny advisory — CVE-ID assignment |
| https://github.com/linux-application-whitelisting/fapolicyd/releases | post-1.5 |
| https://github.com/containers/podman/releases | v6.0 RC tag (none yet) |
| https://github.com/osbuild/image-builder-cli/releases | post-v66 |
| https://github.com/osbuild/image-builder-cli/issues/506 | composefs+UKI sealed-image backend |
| https://github.com/composefs/composefs/commits/main | v1.1 tag (main at a7ea6cd 2026-05-19) |
| https://github.com/bootc-dev/bootc/releases | post-v1.15.2 |
| https://github.com/ostreedev/ostree/releases | post-v2026.1 |
| https://github.com/etcd-io/etcd/releases | 3.7.0 GA progress (beta.0 = 2026-05-19) |
| https://ceph.io/en/news/blog/ | Tentacle patch / Squid bulletins (re-fetch — lower confidence) |
| https://download.gnome.org/teams/releng/ | **GNOME 50.2 (4 days overdue)**, 50.3 + 51.alpha both 2026-06-27 |
| https://github.com/microsoft/WSL/releases.atom | post-2.7.7 stable / post-2.8.7 pre-release |
| https://github.com/systemd/systemd/releases | stable 260.x (now 260.2); v261-rc progression (rc2 = 2026-05-26) |
| https://github.com/renovatebot/renovate/releases | post-43.196.1 |
| https://docs.mesa3d.org/relnotes.html | post-26.1.1 / 26.0.7 |
| https://libvirt.org/news.html | 12.4.0 release |
| https://github.com/NVIDIA/nvidia-container-toolkit/releases | post-1.19.1 |
| https://github.com/gnif/LookingGlass/commits/master/module | KVMFR `module/` commit (idle since 2025-03-04) / B8 |
| https://github.com/ValveSoftware/gamescope/tags | post-3.16.24 / 3.17 + HDR fix 7d4e835 |
| https://github.com/freeipa/freeipa/tags | post-release-4-13-1 (publishes via git tags, not Releases) |

---

## Follow-up questions raised (resolved + unresolved)

**Resolved this pass:**
- **"Is the rolling `:stable-nvidia` tag (the one MiOS pins) also being re-baked daily?"** — **YES.** Re-baked 2026-05-27T04:13Z on the post-#392 base; the kernel-CVE fix is now deliverable on the pinned tag via a Renovate digest bump. (The P2 gate is now purely project-side adoption.)
- **"QEMU 11.0.1 QEMUtiny CVE-ID?"** — answered (negative): still no published CVE-ID; only the oss-sec 2026-05-20 advisory. Kept on the watch list.

**New this pass:**
1. **Has Renovate opened a digest-bump PR for the `:stable-nvidia` pin** now that the rolling tag re-bakes daily with the fixed kernel? (cross-repo / CI inspection — out of research-only scope, but the single most important delivery signal to watch.)
2. **Does issue #385 close** now that both rolling and dated tags carry the fixed kernel? Maintainers gate-close on verification.

**Carried forward (still unresolved — out of research-only scope):**
- Workflows still calling `ublue-os/bootc-image-builder-action`? (migrate to osbuild fork)
- cosign binary pinned to a digest in `automation/42-cosign-policy.sh`? (≥ 3.0.6, `--bundle`)
- SELinux site modules landing in `/etc/selinux/targeted/active/modules/400/` (persists) vs `/usr/lib/selinux/` (wiped)?
- K3s HA mode vs single-node sqlite? (affects etcd-migration urgency)
- `automation/52-bake-kvmfr.sh` applies the LG `MODULE_IMPORT_NS` string-literal patch + signs at image-build time (not first-boot)? **Imminent — the base is now on longterm-6.18 and LG's `module/` has had no code commit since 2025-03-04, so the project's bake script must carry the patch itself.**
- fapolicyd trust DB rebuilt at image-build (`fapolicyd-cli --update`) or via dnf plugin? Does v1.5's transactional rule-reload change the bake?
- GNOME 50 in `ucore-hci:stable-nvidia` yet?
- AMD iGPU used at all? (AMDGPU CVE cluster hits the 9950X3D iGPU unless `amdgpu` blacklisted)
- `mios-sysext-pack.sh` consume systemd 260's `/etc/systemd/systemd-sysext.conf`?
- Which NVIDIA driver line does the project pin? (595.71.x prod-feature vs 580.x LTS — confirm the adopted bake carries ≥ 580.159.03; 610.43.02 is a new-feature branch, not a pin)
- Mesa line in the running image (25.x vs 26.x)?

---

## Priority-order rationale

P0 (reverify) first, as always. **P1 (Secure Boot) stays top** — it is the tightest *hard-calendar* deadline (~28 days, checkpoint 2026-06-05 = ~8 days) and has been verified-absent across all three F44 trees for four passes running with zero movement. **P2 changed shape decisively:** the upstream bake gate is fully closed AND the rolling tag MiOS pins re-bakes daily, so the watch is now purely *project-side adoption* (Renovate digest-bump PR / issue #385 close) — the long-running ucore-bake watch is resolved. **P3 (Pacemaker 3.0.2 final)** is promoted because the projected date (~2026-05-28) is tomorrow — the closest "expected any day" event. **P4 (GNOME 50.2)** is now 4 days overdue and imminent. **P5 (QEMU QEMUtiny CVE-ID)** is a low-exposure confirmation. **P6–P9** are slower monthly/routine checks; LG (P8) retains weight because the project now needs its KVMFR patch on the adopted longterm-6.18 base and LG's `module/` is idle.

Anything not on this list can be skipped tomorrow unless an upstream release explicitly demands inclusion. **Tomorrow's run should overwrite this file with its own next-day agenda.**
