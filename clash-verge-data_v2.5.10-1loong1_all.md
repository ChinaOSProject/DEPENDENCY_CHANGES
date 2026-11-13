# Clash Verge data 2.5.10-1loong1

Upstream data repository: https://github.com/MetaCubeX/meta-rules-dat . This package contains the four unmodified data files selected for Clash Verge Rev2.5.10 from the official release snapshot published November11,2026. The moving `latest` label is not used as the integrity identity; the exact release response and file hashes are retained.

The output is `/home/laevatein/ChinaOSProject/binaries/clash-verge-data/all/clash-verge-data_v2.5.10-1loong1_all.deb`, version `2.5.10-1loong1`, Architecture `all`, Multi-Arch `foreign`, 8041896 bytes, SHA256 `ca905ab2e16bc71745fdd8e720d3747f50a448044cc8d7abe8e50e2100a2e4a3`. Files install under `/usr/share/clash-verge/resources`; the application package provides the sibling resource-directory link expected by Tauri. The `2.5.10` package version identifies this fixed application resource snapshot, not a claimed upstream geodata version. The package includes the upstream repository license, README/attribution links and raw release metadata.

Payload SHA256:

- `Country.mmdb`: c0c6ea5661a3167abb24bbbaf629605aa3856c425384cbf2897c5a0f41027663
- `ASN.mmdb`: 3f729dfdbca46d6bc83fe084e15e335cef702b46f9d9afb8feaede36180959f4
- `geoip.dat`: 90b78ab861c1cdc3f871f305f9ceedfb8d6d6329e3a40295e6773e057297384c
- `geosite.dat`: da712b401ea6f4fdb45983e7a3458332decb85c244307b443df3e11feb4af366

Debian's `transmission-gtk` / `transmission-common` split is the application/data packaging reference, preserved in `/mnt/chinaos-cache/ChinaOSProject-cache/codex/v0.166.0/directory-replacement-probe-20261107/cargo-home/clash-verge-rev-v2.5.10-20261110/packaging-20261112/references`. No matching upstream-specific Debian/Ubuntu package is claimed. Pure data is architecture-independent; there is no ELF, shared library, runtime compiler dependency or maintainer script. Assembly/extraction/MD5 checks passed in the existing project container. The three core/data packages installed together in existing `clash-native-family-runtime-v2510`; `dpkg --verify` passed. A Homepage-only metadata correction preserves every payload MD5. No host installation or proxy change has been performed by this package validation.

Evidence: `/mnt/chinaos-cache/ChinaOSProject-cache/codex/v0.166.0/directory-replacement-probe-20261107/cargo-home/clash-verge-rev-v2.5.10-20261110/inputs/prebuild-verified-main-20261111/manifest.json`; `/mnt/chinaos-cache/ChinaOSProject-cache/codex/v0.166.0/directory-replacement-probe-20261107/cargo-home/clash-verge-rev-v2.5.10-20261110/packaging-20261112/clash-verge-data-package.json`; `/mnt/chinaos-cache/ChinaOSProject-cache/codex/v0.166.0/directory-replacement-probe-20261107/cargo-home/clash-verge-rev-v2.5.10-20261110/packaging-20261112/input-group-install.json` and `.log`. The runtime code and source adaptations belong to the separate application/library reports.
