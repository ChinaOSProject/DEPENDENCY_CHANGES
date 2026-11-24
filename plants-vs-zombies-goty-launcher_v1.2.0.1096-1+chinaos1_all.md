# Plants vs. Zombies GOTY launcher for LoongArch

The `plants-vs-zombies-goty-launcher` Debian package runs GOTY 1.2.0.1096
from a licensed Windows installation supplied by the user. The public
package contains the shell launcher, desktop entry, and documentation.

## Installation

Obtain the versioned `.deb` files in the accompanying package manifest
`goty-launcher-20261124/packages.json` from `ChinaOSProject/binaries`.
The two `clash-` libraries are existing
packages at the listed version. Use Git LFS to retrieve the package
payloads. Put `goty-launcher-20261124/SHA256SUMS` in the same directory
as the nine listed files, then run:

```sh
sha256sum -c SHA256SUMS
sudo apt install ./*.deb
plants-vs-zombies-goty-launcher --game-dir "/path/to/GOTY"
```

The source directory needs `popcapgame1.exe`, `main.pak`, and the rest
of the installed game data. On first use the launcher copies that
directory and creates an i386 Wine prefix. Later launches use the
desktop entry or `plants-vs-zombies-goty-launcher` without arguments.
The launcher stores its state under
`${XDG_DATA_HOME:-$HOME/.local/share}/plants-vs-zombies-goty-launcher`.

The Wine and CPU packages use the device's Debian `loongarch64`
architecture. `wine-proton-runtime` provides native Wine 11.0,
`wine-proton-i386` provides the Windows guest modules, and `box64-wine`
provides the CPU04 batch10 backend. These packages use a private
runtime prefix and the dependencies declared in their control metadata.

## Observed results

On November 24, 2026, the nine packages installed together with
networking disabled in a Debian 10 LoongArch Podman container.
`dpkg --audit` was empty and `apt-get check` succeeded. The installed
launcher initialized a fresh prefix, copied the game from a read-only
mount, and opened an 800x600 game window in Xvfb with llvmpipe and
`WINE_DISABLE_FULLSCREEN_HACK=1`. The game's WineD3D Present logs
averaged approximately 24.70 FPS over 39.83 seconds of sampled title
and intro rendering, using 26 complete intervals from the game process.
The harness stopped the game after sampling. Audio was outside this
test.

The separate local full-game package reached the 1-1 seed selection
tutorial at 29.70 FPS over 28.85 seconds in Xvfb. Before packaging,
the real desktop test placed a Peashooter and averaged 33.30 FPS over
28.79 seconds. The measurements cover different scenes or display setups.
