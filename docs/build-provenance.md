# Build provenance

This document records provenance for already-prepared experiment artifacts. No binary, dependency cache, vendored source, or private build configuration is included in this repository.

## Source identity

| Item | Recorded identity |
| --- | --- |
| Lamco RDP Server | version 1.4.5; source revision `2ca842213437b58af6202ed92bdc87526986d8a0` |
| `lamco-pipewire` | version 0.6.13 |
| Published crate archive SHA-256 | `f184ada837636b715661cb97aaf646e18f758a544f2bd5b2554c1cadf7cac8e9` |
| Original source-tree record SHA-256 | `54ff4be43113a936ad80637f9d3fab1b81b7d22ae65fbf30ec3e0ef432ae20a7` |
| Prepared A source-tree record SHA-256 | `7606220633cb3ed6156e56594f014fb59191854aeb64f890b5421fe118b97a60` |
| Prepared B source-tree record SHA-256 | `69a32b3cdd6bb3605c09988bf0b52cae845fa8d3ca34e462423d160069f8dd1d` |
| A and B server `Cargo.lock` SHA-256 | `83129adcdc35861b94618c8e1313e402bc989749583044f9b86404ed07b3248d` |

The source-tree values are hashes of sorted records in the form `<file-sha256>  <relative-path>\n`, not tarball hashes.

## A/B source boundary

Both arms contain the same local `lamco-pipewire` override, lockfile adjustment, and diagnostic instrumentation. The only intended A-to-B source difference is the removal of `StreamFlags::DRIVER` in [`patches/0003-variant-b-remove-driver.patch`](../patches/0003-variant-b-remove-driver.patch).

## Rust 1.98.1 substitute toolchain and protocol pins

The original installed-build record identified Rust 1.98.0 with observed commit prefix `88d9e12ae`. That exact historical toolchain was not used for the controlled rebuild. The experiment instead defined a substitute mode in which both A and B use the same Rust 1.98.1 protocol pin:

| Tool | Protocol identity | Retained post-build verification |
| --- | --- | --- |
| rustc | `rustc 1.98.1 (48a229cea 2026-09-01)`; full commit `48a229ceaefd4985c50990b14116b6d856af0985` | Confirmed by embedded compiler version and commit strings in both binaries. |
| Cargo | `cargo 1.98.1 (797e8a9bc 2026-08-05)`; full commit `797e8a9bca276c1c9f9f738d2a20f484fa4eea9d` | No retained post-build verification record; documented protocol pin only. |
| clang | `clang version 20.1.8`; target `x86_64-unknown-linux-gnu` | No retained post-build verification record; documented protocol pin only. |

Documented Flatpak x86_64/25.08 protocol pins:

| Component | Commit |
| --- | --- |
| `org.freedesktop.Sdk` | `b90ed309cc1d505dea48b6a2121c5dcfac22868120eee643b0596d31f96b9bb8` |
| `org.freedesktop.Sdk.Extension.rust-stable` | `09784a9989ae5a7fda6a2280033ca9aa32aac6907961da8b18be4590dd261507` |
| `org.freedesktop.Sdk.Extension.llvm20` | `1fac8133086b2ee599d277ad48373c99e64acee7611fc05fbb15beb485b36702` |
| `org.freedesktop.Platform` | `d27f7a6a974e40b061070bec1be9e1b52a7a6872b271e6a45c2c34a48bf6fedf` |

No retained post-build record independently verifies the Cargo executable, clang executable, or Flatpak commit identities used for the resulting binaries. They are documented protocol pins, not post-build observations. Rustc 1.98.1 is the exception because its version and full commit strings are embedded in both binaries.

The substitution controls the confirmed rustc identity between A and B. It does not reproduce the installed build's Rust 1.98.0 identity, so a rebuilt-arm-to-installed-binary comparison cannot isolate DRIVER from compiler-version effects.

## Dependency pinning

- Both feature-specific Linux x86_64 graphs contained 717 packages and were identical after normalising the arm path.
- Registry packages were locked by `Cargo.lock`; 884 registry archives were checksum-verified during input preparation.
- The offline vendor set contained 906 packages and 44,746 verified source files. It is intentionally not redistributed here.
- IronRDP dependencies were pinned to revision `b6be611ccb8776f1133679ab3193cd578a17e58a`.
- The optional NVIDIA Video Codec SDK binding source was locked to revision `f0f0743afcb0019f00f647c482d0699e64ee1b6f`.
- `lamco-pipewire` was redirected to the verified local 0.6.13 source; only that package's registry source and checksum fields were removed from the staged lockfile.
- Build features were `h264,vaapi,opus-libopus,libei,gui` with default features disabled.
- Dependency upgrades and lockfile regeneration were outside the experiment protocol.

## Existing binary hashes

The following SHA-256 values were rechecked against the existing build outputs while preparing this repository. The binaries themselves are not included. They were executed during the experiments described here, but they were not executed as part of publication-repository preparation.

| Arm | File | Size | SHA-256 |
| --- | --- | ---: | --- |
| A | `lamco-rdp-server` | 26,252,152 bytes | `766d49b24e699cb170482492ba43aabccb5165bd430bf17be61d69123a3be589` |
| B | `lamco-rdp-server` | 26,252,152 bytes | `6a89a28c8a5c907c27288dc715a15d8156b6031b8cb58be03446bd95e943827c` |

Distinct hashes are expected because the effective connect flags differ and the unchanged build script embedded different build timestamps in A and B. They are artifact identifiers, not proof that DRIVER is the only possible source of every binary difference.

## Reproducibility limits

- The retained preparation records predate the later build outputs; this repository records the verified output hashes but does not publish raw build logs.
- The original installed application's complete source and lockfile identity was not proven.
- Rust 1.98.1 is an explicit substitute for the original Rust 1.98.0 compiler.
- Cargo, clang, and Flatpak identities are protocol pins without retained post-build verification records.
- The unchanged build script embeds ordinary date/time and Git command output, so bit-identical rebuilding is not claimed.
- Hashes identify artifacts but do not establish runtime behaviour. Runtime claims are supported separately by sanitised diagnostics and operator observation.
