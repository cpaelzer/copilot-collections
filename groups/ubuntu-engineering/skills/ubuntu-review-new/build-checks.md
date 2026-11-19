# Optional native build checks

Run a build only when it answers a concrete and important review question. Record that question first; a build is not required merely because tooling exists. Never install or directly execute downloaded or built candidate artifacts.

## Applicability and isolation gate

With no concrete build question, record `N/A` regardless of environment. Otherwise, run and record `systemd-detect-virt` before preflight:

- Positive container/VM detection, including LXD-based environments, permits preflight; no special LXD detector or additional build-specific confirmation is needed after successful preflight.
- Default `none` means bare metal: record `N/A`, continue static review, and do not install build tools or reconfigure the host.
- An unavailable, erroneous, or inconclusive result prohibits building; record build support `NOT-VERIFIED`.

A chroot or private user namespace alone does not pass this gate. Positive detection does not waive credential, configuration, or cleanup requirements.

## Preflight

Require all of the following before claiming build support in an eligible container or VM:

- Non-root invocation and native architecture from `dpkg --print-architecture`.
- Installed `sbuild`, `mmdebstrap`, `newuidmap`, and `newgidmap`, plus `ip` where required by the installed sbuild version. In an eligible container/VM, install missing build packages in that environment before continuing preflight, using explicitly configured official Ubuntu repositories and recording their sources, versions, and any failures. Do not install build tools on bare metal merely to enable this optional check, and never install candidate artifacts as part of provisioning.
- Valid subordinate UID/GID ranges and setuid-root mapping tools.
- Working user, mount, PID, UTS, IPC, and network namespace creation.
- Successful actual `mmdebstrap` tarball creation and `sbuild` operation. `unshare true` alone is insufficient; when validating the environment independently, build a small known-good native package first.

Do not claim cross/foreign-architecture coverage. Do not change sysctls or LXD profiles, enable privileged containers, weaken AppArmor/seccomp/user-namespace restrictions, or substitute schroot/sudo chroot. Any failed prerequisite makes build evidence `NOT-VERIFIED`; continue static review.

## Exact build context and configuration

Create a fresh absolute scratch directory with checked cleanup for every entry. Create tarballs on demand with `mmdebstrap --mode=unshare --variant=buildd`; do not use a persistent chroot or persistent sbuild cache. During a series run, reuse a tarball only for entries with identical series, native architecture, components, apt sources, pocket configuration, and trust configuration.

Select the official archive or ports mirror for the native architecture. Separate the base series from the target distribution/pocket and configure exact apt sources deliberately:

- An explicit single mirror/suite can configure only the release suite; verify the resulting sources rather than assuming updates or security entries were added.
- Include updates and security as appropriate, and proposed or backports only when the exact review target requires them. `--dist` does not add apt sources.
- Record the effective sources, archive keyring/trust, and target distribution. Do not add repositories implicitly to resolve dependencies.

Place one exact local `.dsc` and every referenced source file in scratch input. Before bootstrap and build, set scratch `HOME`, `XDG_CONFIG_HOME`, `XDG_CACHE_HOME`, and `TMPDIR`. Use a minimal allowlisted environment retaining only necessary tool and explicit proxy/network settings; keep credentials, private artifacts, and credentialed URLs out of the environment, scratch, configuration, logs, and reports.

`SBUILD_CONFIG` is an **additional** configuration file, not standalone isolation: system and applicable user configuration are still read. Inspect and record effective relevant settings and reset inherited extra repositories/packages, bind mounts, external hooks, build environment changes, and logging behavior. Hooks can run outside the chroot. If effective configuration cannot be established and made safe, record `NOT-VERIFIED` rather than building.

Ensure the installed sbuild version's temporary and unpack directories are inside scratch; configure them explicitly where necessary.

## Candidate build

Run normal lintian separately. With the validated tarball, exact target distribution, native architecture, and scratch paths, run:

```bash
sbuild --chroot-mode=unshare --chroot="$tarball" \
  --dist="$distribution" --arch="$arch" --build-dir="$scratch/output" \
  --build-dep-resolver=apt --bd-uninstallable-explainer=apt \
  --no-enable-network --no-run-lintian --no-run-autopkgtest \
  --no-run-piuparts --purge=always "$scratch/input/package.dsc"
```

`--no-enable-network` blocks external networking during `dpkg-buildpackage`; it does not prevent earlier dependency installation, and loopback may be enabled. Identify attempted non-hermetic access rather than re-enabling networking to make the build pass.

## Evidence and cleanup

Record the build question, virtualization identifier, architecture, exact archive/pocket configuration, relevant tool versions and effective settings, exit status, and concise evidence. Distinguish:

- Environment/preflight failure: build `NOT-VERIFIED`.
- Dependency resolution failure: unresolved dependencies and the exact configured sources used.
- Non-hermetic network requirement: the attempted access.
- Candidate build failure: relevant compiler/package evidence, classified separately from infrastructure.

A failed or unrun build is not a candidate `FAIL` without candidate-specific evidence. Decide whether unresolved evidence affects the verdict under [SKILL.md](SKILL.md). Preserve sanitized diagnostic excerpts before deleting temporary trees and logs on success, failure, or interruption; use checked, namespace-aware cleanup for residual subordinate-owned trees while retaining the original command or signal status.
