# libX11 1.8.12 source for the ChinaOSProject Proton runtime packages

This bundle accompanies the three runtime packages at version `1.8.12-1+chinaos1+proton11.0.2e1`. It includes the exact X.Org source archive used for the build, its detached signature, the build and packaging recipes, license notices, and a package identity and notice audit.

The published archive is `source/goty-20261124/libx11-1.8.12-proton-runtime-source-20261124.tar.gz` in `ChinaOSProject/binaries`. Its SHA-256 is `f0749f020af0e9fd59f6242284439dbc54668b0408dec961978f5f9731c7129d`.

| Package | Architecture | Release filename |
| --- | --- | --- |
| `libx11-6-proton` | `loongarch64` | `libx11-6-proton_v1.8.12-1+chinaos1+proton11.0.2e1_loongarch64.deb` |
| `libx11-xcb1-proton` | `loongarch64` | `libx11-xcb1-proton_v1.8.12-1+chinaos1+proton11.0.2e1_loongarch64.deb` |
| `libx11-data-proton` | `all` | `libx11-data-proton_v1.8.12-1+chinaos1+proton11.0.2e1_all.deb` |

The source is `upstream/libX11-1.8.12.tar.xz`, SHA-256 `fa026f9bb0124f4d6c808f9aef4057aad65e7b35d8ff43951cef0abe06bb9a9a`. The build used the unmodified X.Org release and its supplied `configure` script. The official Git tag `libX11-1.8.12` has object ID `b1cf5da316e6465a65503fc54fa95737750f422a` and resolves to `59917d28a3c41ad22d6fc52e323cafe2cdd596d5`.

The original verification compared the signed release to the pinned Git tree. All 1,628 Git-tracked files present in the release matched byte for byte. The remaining nine tracked files are Git, CI and bootstrap files. The detached signature validated to signing subkey `2F7DBCAD5B30E668BBD516C1CFDF148828C642A7` and primary key `4A193C06D35E7C670FA4EF0BA2FB9E081F2D130E`. The original verifier recorded undefined local key trust. The bundled signature can be checked independently using a keyring whose trust the verifier has established.

From the extracted bundle directory, `sha256sum -c SHA256SUMS` checks the included files. `BUILD-PACKAGE.md` records the procedure and build environment. `runtime-package-audit.json` gives the `.deb` and compressed `data.tar.xz` SHA-256 for each package and the three notices or provenance files embedded in each. It also records the package control and shared-library metadata.

Each of the three released `.deb` files carries `COPYING.upstream` and `copyright` as regular files under `/opt/proton-11.0-2e/runtime/share/doc/<package>/`. The audit compared those files byte for byte with the X.Org archive's `COPYING` and Debian's `debian/copyright`, respectively. It also checked each embedded notice and the `build-provenance` file against the package's `md5sums`. `licenses/lcUniConv-COPYRIGHT` preserves the separate source notice in full. Its copyright, permission, redistribution condition, disclaimer and name-use clauses are present in both notices carried by each `.deb`, as verified after normalizing whitespace.

This bundle reuses the November 14, 2026 adaptation report and original validation records. The release bundle audit read archive contents and compared hashes and notices. The original build and runtime checks are summarized in `BUILD-PACKAGE.md`. AI assisted the original orchestration, auditing, packaging and reporting, and the preparation of this bundle.

Official X.Org source and release references:

- https://gitlab.freedesktop.org/xorg/lib/libx11.git
- https://www.x.org/releases/individual/lib/libX11-1.8.12.tar.xz
- https://www.x.org/releases/individual/lib/libX11-1.8.12.tar.xz.sig
- https://lists.x.org/archives/xorg-announce/2025-March/003587.html
