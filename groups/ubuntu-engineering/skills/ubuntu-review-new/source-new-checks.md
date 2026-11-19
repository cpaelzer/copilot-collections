# Source NEW checks

Use the result model in [SKILL.md](SKILL.md); checks are required when applicable unless marked advisory. Scanners provide leads: manually disposition unknown, generated, excluded, and unmatched-file reports and false positives. A failed or unavailable scanner makes its evidence `NOT-VERIFIED`, not `FAIL`; equivalent manual evidence may still support the overall check.

## Namespace and package identity

- Use judgement for source and binary name squatting—especially unused three-to-five-character names—and plausible future Debian namespace collisions; name length alone is not an automatic rejection.
- Verify that `debian/control` names the intended binary outputs and available source/binary metadata agrees; investigate unlisted built packages and listed packages unexpectedly absent.

## License and copyright

### Upstream grant and source availability

- Require an explicit upstream license grant in `COPYING`, `COPYRIGHT`, `LICENSE`, source headers, or equivalent upstream evidence.
- Verify that redistribution is permitted. For uncommon or bespoke licenses in the free components, explicitly verify modification, redistribution of modifications, and distribution of binaries built from the code.
- For `main` and `universe`, ensure corresponding source exists as required by the DFSG. Investigate binary blobs, generated files without inputs, and missing source.
- For `restricted` and `multiverse`, do not reject solely because a payload is binary-only. Verify redistribution rights, applicable license conditions, and correct component assignment instead.

### `debian/copyright` format and review method

- Verify that `debian/copyright` correctly and completely reports the licenses and copyright holders for all files.
- DEP-5 machine-readable copyright is encouraged but optional. If the file declares that format, validate the declared specification version, syntax, stanza structure, file-pattern matching, and license references. Demonstrated nonconformance is reject-level under Ubuntu NEW guidance; using a non-DEP-5 format is not itself a defect.
- Compare license-scanner findings thoroughly against the source and `debian/copyright`; no particular scanner is required when equivalent manual evidence is sufficient.
  - **Tool:** Within the unpacked source tree, use `lrc` for declared DEP-5 when available and `licensecheck -r .` for general license discovery, including non-DEP-5 review. Compare their findings with `debian/copyright` and manually disposition discrepancies.
- Verify that `debian/copyright` is UTF-8 and identifies the upstream source origin, if any. An upstream author or contact is recommended, but its absence is not automatically a blocker.

### License texts and common-license references

- Include the applicable distribution-license terms and all notices, exceptions, and warranty disclaimers required by the license and packaging policy. Uncommon licenses need their applicable full terms.
- For the common licenses enumerated in [Debian Policy on copyright information](https://www.debian.org/doc/debian-policy/ch-docs.html#s-copyrightfile), use the corresponding `/usr/share/common-licenses` reference where valid rather than requiring duplicated text. Do not infer this exception merely from a same-named file on the review host.
- Check the declared copyright format and each license's own notice requirements. A valid common-license reference is not a rejection reason, and quoting additional common-license text is not by itself an archive blocker. The historical University of California BSD text must be included as required by Debian Policy.

### Canonical copyright and licensing rules

- Canonical open-source projects default to GPLv3. Canonical-copyrighted files declared GPLv2 are an automatic rejection, except that Canonical contributions to an upstream project may use the upstream license. Current policy treats changes to `debian/packaging` as upstream contributions.
- Work by Canonical employees must list **Canonical Ltd.** as copyright holder. Listing an individual with an `@canonical.com` address as holder is an automatic rejection.
- Do not apply these narrow rules to unrelated upstream GPLv2 code.

## Component and dependency checks

- Check the source component/section and each intended binary's component/section/priority against the package contents and target archive metadata. Do not assume that all binaries share the source component.
- Apply Ubuntu's component rules: `main` and `universe` generally follow the DFSG and Debian interpretation, but Ubuntu does **not** consider GFDL-licensed documentation non-free in the way Debian does.
- `restricted` is for non-free hardware enablement, drivers, firmware, and non-free codec support. Other non-free software belongs in `multiverse`; a support statement alone does not justify `restricted`.
- If software otherwise belonging in `restricted` is proposed for `main` for dependency reasons, require the documented full MIR evidence.
- If Source NEW acceptance would place a package in Ubuntu `main`, verify the applicable MIR approval or documented exception; unresolved required status prevents recommending acceptance into `main`. Do not perform or repeat MIR dependency-closure analysis at NEW; reserve it for an actual MIR review or concrete dependency/component concern. Keep Debian `main`'s stronger dependency rules distinct.
- **Advisory:** Dependency-license compatibility is desirable but not currently enforced at NEW acceptance. Record evidence or uncertainty; do not invent a replacement dependency policy or blocker.

## Correctness, lintian, and layout

- Inspect descriptions, standards metadata, `debian/rules`, package splits, and source/binary consistency for concrete correctness or archive problems, not complexity, phrasing, or packaging style.
- Report only objective description errors—spelling, factual inaccuracies, or invalid formatting—never stylistic improvements. An old `Standards-Version` alone is not a bug; identify malformed or demonstrably false declarations and relevant unreviewed policy changes.
- Run source lintian, recording its version and actual profile. Evaluate tags and overrides by the underlying violation, not tag severity, including error tags; very few findings justify rejection. Declared DEP-5 nonconformance remains reject-level as defined above.
  - **Tool:** Run `lintian -I <package>.dsc`; add `--profile ubuntu` where supported and optionally `-E --pedantic` for expanded output.
- Inspect effective installation paths for FHS compliance, scoped to the inspected evidence; `.install` files alone do not prove final layout. For a remaining concrete layout question, use the optional build policy or record that question `NOT-VERIFIED`; never require a build solely for this checklist.
  - **Tool:** Inspect `debian/*.install`, `debian/rules`, helper metadata, and effective `debian/tmp` or package-staging paths when already available. When exact `.deb` artifacts are already available, use `dpkg-deb -c <package>.deb` to inspect their final paths.
