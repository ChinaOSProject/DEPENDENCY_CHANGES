# Wine 11.0 for Plants vs. Zombies GOTY on LoongArch

Upstream source: <https://github.com/ValveSoftware/wine>

This local build packages the frozen Wine handoff07 alongside the OpenGL
overlay used in the GOTY play test. The native `wine-proton-runtime` package
contains the LoongArch launcher, wineserver, and native Wine modules. The
`wine-proton-i386` package of the same version contains the Windows i386
modules, including the two additional official Wine modules `wsock32.dll` and
`mswsock.dll`. Both packages use the host's Debian architecture,
`loongarch64`, because the i386 modules belong to this specific LoongArch
runtime.

The distribution's `wine` package version `5.0-3kylin1` distinguishes the
standard Wine suite from architecture-specific loaders and libraries.
This build keeps its runtime and guest modules separately versioned under
`/opt/wine-proton-native`, since the guest runs through a separately packaged
WoW64 CPU backend. The private libX11 and GCC runtime packages are reused
byte-for-byte. The native package declares GLIBC 2.28 as its minimum and
declares its system X11, OpenGL, font, TLS, audio and device library
dependencies in Debian control metadata.

The nine-package local set installed offline on the Debian 10/LoongArch
base. `dpkg --audit` was empty, `apt-get check` succeeded, the loader
reported `wine-11.0`, and the installed game launcher created a fresh
i386-enabled prefix successfully. The launcher installs the two extra
networking DLLs only into that per-user prefix. This is a GOTY-specific
runtime package set. AMD64 loader support and other Proton titles are outside
this validation.

The two packages share the corresponding-source archive in
`ChinaOSProject/binaries` at
`source/goty-20261124/wine-proton-11.0-1+chinaos1+goty20261124-corresponding-source.tar.xz`.
It includes the Wine Git base, native and guest overlays, the exact generated
headers used by the i386 builds, build recipes, and component license notices.
The archive documents two unavailable historical generator inputs and
identifies the `.spec` files actually used by the builds. The source archive
was checked against the recorded inputs, and no bit-for-bit rebuild was
performed for publication.
