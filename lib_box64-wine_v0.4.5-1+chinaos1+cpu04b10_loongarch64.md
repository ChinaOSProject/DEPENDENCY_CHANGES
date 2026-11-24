# Box64 0.4.5 WoW64 backend for the GOTY Wine runtime

Upstream source: <https://github.com/ptitSeb/box64>

This package carries the CPU04 batch10 LoongArch Unix backend
`wowbox64.dll.so` and its Wine PE metadata companion `wowbox64.dll`.
The package has an exact dependency on
`wine-proton-runtime (= 11.0-1+chinaos1+goty20261124)` and installs into its
private module tree under `/opt/wine-proton-native`. The pair is a Wine ABI
component. Its package name distinguishes it from standalone Box64 Linux
executables.

The frozen native ELF is LoongArch, flags `0x3`, with a maximum versioned
GLIBC requirement of 2.27. The installed module initialized the i386 prefix
successfully on the Debian 10 base. The launcher selects CPU04 with dynamic
recompilation and disables user Box64 configuration files for this runtime.
AMD64 execution bypasses this i386 backend and is a separate compatibility
effort.

The build source and license are published as
`source/goty-20261124/box64-wine-0.4.5-1+chinaos1+cpu04b10-source.tar.gz`
in `ChinaOSProject/binaries`. The archive SHA-256 is
`a8b53983bde627b31793363f8e39f4a039e4cc1dc352e71c70097f7aa39c5b73`.
