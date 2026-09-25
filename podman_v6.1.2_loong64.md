# Podman v6.1.2 loong64 dependency adaptation report

Dependency: conmon
Version: 2.2.1; upstream commit c8cc2c4db27531bd4e084ce7857f73cd21ee639d
Type: OCI runtime monitor
Reason: Podman 6.1.2 passes `--full-attach` to conmon during container creation.
Root Cause: Kylin V10 SP1 provides conmon 2.0.9-1, which rejects `--full-attach`. The configured Kylin apt index has no newer conmon candidate.
Changes: Built the unmodified upstream v2.2.1 source on the target with GCC 8.3.0 and installed it at `/usr/local/libexec/podman/conmon` for initial validation. Then unpacked the conmon `.deb` into the system root, placing v2.2.1 at `/usr/libexec/podman/conmon`. dpkg still records Kylin's 2.0.9-1 package because the local package was unpacked without dpkg registration.
Compatibility Impact: The binary is LoongArch ELF and requires at most `GLIBC_2.27`; the target glibc is 2.28. Podman's default conmon path resolves to the installed v2.2.1 executable.
Upstream Status: No upstream source modifications. Built from the official `containers/conmon` v2.2.1 tag.
Validation: Build succeeded. `conmon --version` reported 2.2.1. Rootful and rootless Podman container runs completed and printed `Hello from Docker!`.
Project-side Changes: None.

Dependency: GLib development files
Version: libglib2.0-dev 2.64.6-1~kylin20.04.4k5.11
Type: Build dependency
Reason: The conmon source build requires GLib development headers and pkg-config metadata.
Root Cause: The target did not have `glib-2.0.pc` or the GLib development headers installed.
Changes: Installed `libglib2.0-dev` and its 15 required packages from the existing Kylin apt indexes with `--no-upgrade`. Apt reported 0 upgrades, 16 new packages, and 0 removals.
Compatibility Impact: Build-only headers and metadata were added. No installed package was upgraded by this apt transaction.
Upstream Status: Used Kylin-provided development packages; no source modifications.
Validation: The conmon v2.2.1 build completed with journald and seccomp support enabled.
Project-side Changes: None.

Dependency: Netavark
Version: 2.1.0
Type: Runtime network backend
Reason: Podman's configured network backend is Netavark, and the target did not have the executable files from the supplied package installed in the system paths.
Root Cause: Kylin's package verification rejected the supplied local package during dpkg installation.
Changes: With user authorization, unpacked the data archive from `netavark_v2.1.0_loongarch64.deb` into the system root. Reloaded systemd unit metadata without restarting services. The package remains unregistered in dpkg. Detailed build and ABI records are in `netavark_v2.1.0_loongarch64.md`.
Compatibility Impact: Netavark reports version 2.1.0 and Podman reports the Netavark backend. The container verification used `--network=host`; bridge and firewall configuration were not exercised.
Upstream Status: No Netavark source modification in this work.
Validation: `/usr/libexec/podman/netavark --version` reported 2.1.0. Podman's `info --debug` reported Netavark 2.1.0. The hello-world container ran successfully with host networking.
Project-side Changes: None.

Dependency: passt/pasta
Version: upstream commit 588b545dae741bec6fd7622a33c7852c06d72a59
Type: Rootless networking runtime
Reason: Podman 6.1.2 defaults to pasta for rootless networking. Kylin apt indexes do not provide a `passt` package.
Root Cause: The target had no `pasta` executable. The available `slirp4netns` package is not accepted by Podman 6.1.2.
Changes: Built upstream passt on the target with GCC 8.3.0 and installed `passt` at `/usr/local/bin/passt` with `pasta` symlinked to it. The target's Linux 5.4 headers lacked `linux/close_range.h`; the official Linux v5.10 UAPI header was placed in a build-local include directory. System headers were left unchanged.
Compatibility Impact: The LoongArch ELF requires at most `GLIBC_2.27`. Linux 5.4 returns `ENOSYS` for `close_range`; upstream passt warns and continues. Rootless networking was verified on this kernel.
Upstream Status: No passt source modifications. Built from the official upstream commit above.
Validation: `pasta --version` reported commit `588b545`. Podman `info --debug` reports `/usr/local/bin/pasta`; the rootless hello-world container ran successfully with the default pasta network.
Project-side Changes: None.

Dependency: Rootless user namespace and network utilities
Version: uidmap 1:4.8.1-1kylin5.20.04.5k1.2; slirp4netns 0.4.3-1 (installed, then removed)
Type: Rootless runtime utilities
Reason: Rootless Podman requires `newuidmap` and `newgidmap`; rootless networking uses pasta.
Root Cause: The target had no uidmap tools or pasta executable, and Kylin apt indexes had no passt package.
Changes: Installed uidmap and slirp4netns from Kylin apt with `--no-upgrade` (2 new packages, 0 upgrades, 0 removals). Podman 6.1.2 rejected slirp4netns networking, so slirp4netns was then removed. Pasta was built from upstream and installed separately.
Compatibility Impact: No pre-existing package was upgraded or removed. The final apt addition is uidmap.
Upstream Status: Used Kylin packages; no source modifications.
Validation: `newuidmap` and `newgidmap` are setuid-root executables. `/etc/subuid` and `/etc/subgid` already assign `100000:65536` to `laevatein`.
Project-side Changes: None.

Validation environment: Kylin V10 SP1, LoongArch64 (`loong64` in Podman and `loongarch64` in dpkg), glibc 2.28, kernel 5.4, unified cgroup v2.
Rootless user configuration: `runtime = "crun"`, `cgroup_manager = "cgroupfs"`, storage driver `vfs`, and a user-level policy accepting only `cr.loongnix.cn/hello-world`. VFS was selected because the Kylin `fuse-overlayfs` install plan would remove the existing `fuse` package.
Rootless validation command: `podman run --rm --pull=always cr.loongnix.cn/hello-world`, executed as `laevatein` without sudo. It printed `Hello from Docker!` and exited successfully.
Package: `../binaries/podman/loong64/podman_v6.1.2_loong64.deb`; `../binaries/conmon/loong64/conmon_v2.2.1_loong64.deb`.
Package SHA-256: Podman `b952f169334a738dba218fc4df83d59256c15eff3c684ac8d8b7bf1230464dea`; conmon `952fd9c5cc993ff5757dd435e47d7e9ba5747c21904ad1ce4b3548829ad30985`.
Podman package recommendations: `aardvark-dns`, `passt`.
Deployment: Unpacked both `.deb` payloads into `/` without dpkg registration. dpkg still records conmon 2.0.9-1 while `/usr/libexec/podman/conmon` reports 2.2.1. Installed crun and passt under `/usr/local/bin`; reloaded systemd unit metadata without restarting services.
