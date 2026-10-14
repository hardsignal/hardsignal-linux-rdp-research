# Patch set

Apply patches in numeric order to the matching upstream source versions.

| Patch | Purpose | Upstream licensing boundary |
| --- | --- | --- |
| `0001-use-local-lamco-pipewire.patch` | Redirect the server to the verified local `lamco-pipewire` 0.6.13 source and update only that lockfile entry. | Touches Lamco RDP Server 1.4.5, licensed by Lamco under BUSL-1.1. |
| `0002-capture-diagnostics.patch` | Add low-impact counters shared by A and B and remove the allocating pixel preview. | Modifies `lamco-pipewire` 0.6.13, upstream MIT OR Apache-2.0. |
| `0003-variant-b-remove-driver.patch` | Remove only `StreamFlags::DRIVER` for Variant B. | Modifies `lamco-pipewire` 0.6.13, upstream MIT OR Apache-2.0. |

The repository-level Apache-2.0 licence applies to Hardsignal-authored material. It does not relicense Lamco RDP Server or third-party patch context. The `lamco-pipewire` modifications are offered under Apache-2.0, one of the upstream project's licensing options. The server patch remains subject to [`LICENSE-LAMCO-RDP-SERVER-BUSL-1.1`](LICENSE-LAMCO-RDP-SERVER-BUSL-1.1).

No vendored dependency, complete upstream source file, generated binary, or private build configuration is included.
