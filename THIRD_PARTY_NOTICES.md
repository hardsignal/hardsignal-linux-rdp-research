# Third-party notices

This repository is an independent Hardsignal Labs research record. It is not an official Lamco, PipeWire, IronRDP, Flatpak, Rust, or Apple publication, and no endorsement is implied.

## Lamco RDP Server

`lamco-rdp-server` 1.4.5 is developed by Lamco Development LLC and is licensed under the Business Source License 1.1 with the parameters supplied by Lamco. The small manifest and lockfile patch in this repository is provided solely to describe the experiment and remains subject to the upstream licence, reproduced at [`patches/LICENSE-LAMCO-RDP-SERVER-BUSL-1.1`](patches/LICENSE-LAMCO-RDP-SERVER-BUSL-1.1). No Lamco server source tree or binary is redistributed here.

- Upstream: <https://github.com/lamco-admin/lamco-rdp-server>
- Product and licensing information: <https://www.lamco.ai/>

## lamco-pipewire

`lamco-pipewire` 0.6.13 is developed by Lamco Development LLC as part of the lamco-wayland ecosystem. The upstream crate is offered under MIT OR Apache-2.0. The diagnostic and DRIVER modifications are offered under Apache-2.0, one of the upstream licensing options. No complete crate archive or source tree is redistributed here.

- Upstream: <https://github.com/lamco-admin/lamco-wayland>
- Crate: <https://crates.io/crates/lamco-pipewire>

## IronRDP

The tested Lamco source pinned IronRDP to the Lamco fork revision `b6be611ccb8776f1133679ab3193cd578a17e58a`. IronRDP identifies Devolutions Inc. and Teleport as authors and is licensed under MIT OR Apache-2.0. No IronRDP source or binary is included.

- Upstream project: <https://github.com/Devolutions/IronRDP>
- Tested fork: <https://github.com/lamco-admin/IronRDP>

## Other named components

PipeWire, XDG Desktop Portal, Flatpak, Rust, Cargo, LLVM/clang, and the NVIDIA Video Codec SDK binding are named to identify the tested stack and build inputs. Their names and marks belong to their respective owners. No source or binary from these projects is included.

- PipeWire: <https://pipewire.org/>
- XDG Desktop Portal: <https://flatpak.github.io/xdg-desktop-portal/>
- Flatpak: <https://flatpak.org/>
- Rust and Cargo: <https://www.rust-lang.org/>
- LLVM/clang: <https://llvm.org/>
- libfuse: <https://github.com/libfuse/libfuse>
- NVIDIA Video Codec SDK Rust binding: <https://github.com/ViliamVadocz/nvidia-video-codec-sdk>

The tested dependency graph also contains transitive open-source packages governed by their own licences. Those packages are identified by the upstream Lamco lockfile and build metadata, neither of which is redistributed here. Anyone reconstructing the build must retain and review the licence files supplied with the dependencies they obtain.

The RDP client used for the visual check ran on an Apple iPad. “iPad” is used only to identify the client device class; Apple is not affiliated with this research.
