# Plants vs. Zombies GOTY launcher 1.2.0.1096-1+chinaos3

The public launcher accepts a locally obtained GOTY game directory with
`--game-dir DIRECTORY`. It imports `main.pak` and the installed game
files and accepts either `PlantsVsZombies.exe` or `popcapgame1.exe`.
With the local 1.2.0.1073 game package installed, the application-menu
entry opens that version by default. The bundled and imported game
profiles use separate Wine prefixes and writable data.

The application-menu entry lives in the launcher package. The public
package carries a generic gamepad icon in `/usr/share/pixmaps`.
The optional local game package provides the original icon through the
standard `hicolor` theme. It installs game files and icons without a
desktop entry. The public package contains no game executables, game
data, or original artwork.

The launcher retains the previously tested Wine, i386 guest libraries,
and Box64 dependencies. The launcher script matches the script used in
the earlier English GOTY title-screen run byte for byte.
`desktop-file-validate` succeeded, and GTK resolved the generic icon
with the launcher alone and the original icon with both packages.
An offline LoongArch Debian 10 container upgraded both packages through
APT. `dpkg --audit`, `dpkg --verify`, and `apt-get check` passed, and
`dpkg -S` confirmed that the game package owns the original icon.

- Debian package `plants-vs-zombies-goty-launcher`
- Version `1.2.0.1096-1+chinaos3`, architecture `all`
- Size 3,324 bytes
- SHA-256 `60ccdb722fcd636d8a457ed237255eb5d06038af66edb59fdef02d6d7920f34d`
- [Binary publication](https://github.com/ChinaOSProject/binaries/tree/56959321e964ae170713339c55c07c5caf60f00e/plants-vs-zombies-goty-launcher/all)
