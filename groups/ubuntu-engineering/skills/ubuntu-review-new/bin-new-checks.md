# Binary NEW checks

Use the result model in [SKILL.md](SKILL.md); applicable checks are required unless marked conditional or advisory. Bin-NEW is deliberately lighter, especially after Source NEW: only egregious problems justify outright rejection; lesser demonstrated problems are follow-up bugs, never implicit blockers.

## Contents, FHS, and installed copyright

- Inspect every exact `.deb` for intended files and FHS-compliant installed paths; source layout alone is insufficient.
  - **Tool:** Use `dpkg-deb -c <package>.deb` to list all installed paths.
- **Conditional:** When useful, run binary lintian and record its version and actual profile. Evaluate tags and overrides by the underlying violation, not tag severity, including error tags. If needed for a concrete question but unable to run, use `NOT-VERIFIED`.
  - **Tool:** Run `lintian -I <package>.deb`; add `--profile ubuntu` where supported and optionally `-E --pedantic` for expanded output.
- Verify applicable copyright notices and license terms at `/usr/share/doc/<package>/copyright`; the file itself must be uncompressed and non-symlink. A documentation-directory symlink is valid only when both packages share the same source package and version and the linking package declares `Depends` on the target; inspect the target copyright. A valid directory symlink needs no separate regular copyright member in the linking package. Serious licensing or notice violations can be egregious; lesser defects remain follow-ups.
  - **Tool:** Use `dpkg-deb -c <package>.deb` to confirm the path. Create `tmpdir=$(mktemp -d)` with cleanup on all exits, extract with `dpkg-deb -x <package>.deb "$tmpdir"`, inspect the extracted copyright with `file`, and inspect any documentation-directory symlink with `readlink`. Use `dpkg-deb -I <package>.deb` and exact source/version metadata to validate the same-source/version and `Depends` requirements and to identify the target package to inspect.

## File conflicts, relationships, and dependencies

- Check exact installed paths for unresolved or unintended conflicts using official Ubuntu Contents indices for the exact target series, pocket context, and architecture; separately check short `$PATH` executable names for namespace collisions. Host-series or unconfigured `apt-file` output is not evidence.
  - **Tool:** Search exact paths in the official `Contents-<architecture>.gz` indices or use `apt-file search <path>` only after explicitly configuring its cache and Ubuntu archive/series/architecture. Disposition exact full-path matches; substring matches are leads only. Missing indices, stale data, and command errors make conflict analysis `NOT-VERIFIED`.
- For moved or intentionally overwritten files, validate actual package names and exact versions in `Replaces` and, normally, versioned `Breaks` against old/new contents and upgrade paths. For ongoing same-file cases, assess `Conflicts`, alternatives, or another mechanism. Never require `Breaks`/`Replaces` merely because a binary is new or a SONAME changed.
  - **Tool:** Use `dpkg-deb -I <package>.deb` to view control metadata and relationships.
- **Conditional:** For a concrete dependency concern or requested exhaustive review, resolve `Pre-Depends` and `Depends` against the exact target series/pocket and architecture, including alternatives, `Provides`, and architecture qualifiers. Distinguish unsatisfiable metadata from temporary `-proposed` transitions; mutable publication ordering alone is not a rejection reason. Missing full-resolution evidence alone does not gate routine bin-NEW review; triggered but unresolved evidence is `NOT-VERIFIED`.
  - **Tool:** Combine `dpkg-deb -I <package>.deb` with exact target `Packages` indices or equivalent explicitly configured archive metadata. Do not use host apt state as target evidence.

## Shared libraries

- Discover actual ELF libraries in their real and multiarch paths. Do not assume a `libfoo` name or use only `/usr/lib/*/libfoo.so.*` globs.
  - **Tool:** Create `tmpdir=$(mktemp -d)` with cleanup on success, failure, or interruption. Run `dpkg-deb -x <package>.deb "$tmpdir"` and `find "$tmpdir" -type f -exec file {} +`, then inspect actual ELF files with `readelf -d <actual-file>` or `objdump -p <actual-file>`.
- Follow [Debian Policy on run-time shared libraries](https://www.debian.org/doc/debian-policy/ch-sharedlibs.html#run-time-shared-libraries): the package containing a public run-time library must change name whenever its SONAME changes, normally using the documented libraryname/soversion convention. This is not a rule that every development package, internal module, or other library-related package name literally equals its SONAME. Group multiple public libraries in one runtime package only when their SONAMEs will always change together.
- Use the actual ELF `SONAME` and apply the documented transformation and transition checks. Compare the previous runtime package and existing overrides where applicable.
- For each public shared library subject to Debian's shared-library dependency rules, verify that suitable **either `symbols` or `shlibs`** metadata is provided and that consumers can generate appropriate dependencies. A missing `.symbols` file alone is not a blocker when valid `shlibs` metadata exists. Symbols are recommended for most C libraries; exhaustive symbols can be onerous for C++, making `shlibs` appropriate.
  - **Tool:** Extract the control archive with `dpkg-deb -e <package>.deb "$tmpdir/DEBIAN"` and inspect its `symbols` or `shlibs` metadata and related dependency generation.
- Check the runtime SONAME symlink and the development/runtime split and support files where applicable.

## Content, architecture, and override comparisons

- For a new architecture, compare contents with an existing architecture. For a SONAME transition, compare with the previous runtime package and investigate unexplained differences.
  - **Tool:** Use `debdiff <old>.deb <new>.deb` when available or compare exact `dpkg-deb -c <package>.deb` outputs.
- For `Architecture: all`, verify independence from the build/host architecture. ELF files, compiled modules, generated data, scripts, and architecture-specific paths trigger analysis, not automatic verdicts; cross or similar packages can legitimately contain payloads for another target architecture.
  - **Tool:** In the safely unpacked tree, run `find "$tmpdir" -type f -exec file {} +` and manually disposition architecture-specific findings.
- Check that every intended architecture was built and investigate unexplained per-architecture differences.
  - **Tool:** Use exact Launchpad queue and build metadata for the source/version and target series; do not infer coverage from the host architecture.
- Check each binary's component, section, and priority against queue metadata, contents, and existing overrides. One source may produce binaries in different components. Report incorrect overrides without changing them.
  - **Tool:** Use exact queue metadata and official archive `indices`; for SONAME transitions, compare existing overrides from those indices, and for kernel ABI renames, review the documented `kernel-overrides` evidence without applying changes.
- When relevant to human acceptance outside `universe`, record the documented warning that Web UI acceptance can override all binaries into the source's component.

## Optional follow-ups

- `wrap-and-sort` is optional advice only.
- Symbols improvements, non-egregious lintian findings, and other lesser issues are follow-up bugs, not automatic rejection reasons. This does not replace the required `symbols`-or-`shlibs` check.
