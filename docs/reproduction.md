# Safe reproduction

This procedure is intentionally conservative. It describes how to reproduce the research without changing an installed Lamco deployment, the working Variant B process, system services, or production credentials.

## Safety boundary

Do not perform the experiment on the active installation. Use a disposable VM, disposable user account, or separately namespaced Flatpak build and runtime environment. Compilation and live capture require explicit local authorisation beyond merely reading this repository.

Never copy a production configuration, private key, certificate, portal token, credential store, raw log, or dependency cache into the experiment directory. Generate experiment-only TLS material and delete it after the test. Do not expose an unauthenticated RDP listener on a routable interface.

## 1. Acquire authorised sources

Obtain these exact upstream versions through authorised channels and review their licences:

- Lamco RDP Server 1.4.5;
- `lamco-pipewire` 0.6.13; and
- the dependencies locked by the matching Lamco `Cargo.lock`.

Verify the crate archive and source identities in [build provenance](build-provenance.md). Do not use or publish the private caches from the original experiment.

## 2. Prepare isolated arms

Create two fresh arm directories outside any installed application directory. Within each arm, the server and capture sources must be sibling directories with these exact names:

```text
variant-a/
├── lamco-rdp-server/
└── lamco-pipewire-0.6.13/

variant-b/
├── lamco-rdp-server/
└── lamco-pipewire-0.6.13/
```

The patch paths start with `a/` and `b/`, so apply every patch from the parent containing those two siblings and use strip level `-p1`. Apply in numeric order. For Variant A:

```sh
cd /path/to/variant-a
patch --dry-run -p1 < /path/to/publication/patches/0001-use-local-lamco-pipewire.patch
patch -p1 < /path/to/publication/patches/0001-use-local-lamco-pipewire.patch
patch --dry-run -p1 < /path/to/publication/patches/0002-capture-diagnostics.patch
patch -p1 < /path/to/publication/patches/0002-capture-diagnostics.patch
```

For Variant B, start from another fresh pair of sibling directories and apply `0001`, then `0002`, then `0003`:

```sh
cd /path/to/variant-b
patch --dry-run -p1 < /path/to/publication/patches/0001-use-local-lamco-pipewire.patch
patch -p1 < /path/to/publication/patches/0001-use-local-lamco-pipewire.patch
patch --dry-run -p1 < /path/to/publication/patches/0002-capture-diagnostics.patch
patch -p1 < /path/to/publication/patches/0002-capture-diagnostics.patch
patch --dry-run -p1 < /path/to/publication/patches/0003-variant-b-remove-driver.patch
patch -p1 < /path/to/publication/patches/0003-variant-b-remove-driver.patch
```

Do not apply `0003` before `0002`, and do not run these commands from inside either sibling directory. After patching, confirm that a normalised recursive diff between A and B contains only the DRIVER-line change. Do not edit the server configuration or installed Flatpak during source preparation.

## 3. Reconstruct the pinned build environment

Use a separate user-scoped Flatpak installation or equivalent disposable environment with the exact SDK, runtime, Rust, Cargo, and clang identities recorded in [build provenance](build-provenance.md).

Resolve exclusively from the matching locked inputs. Build with `--frozen`, with downloads disabled, default features disabled, and this feature list:

```text
h264,vaapi,opus-libopus,libei,gui
```

Build A and B into different state and output directories. Record complete tool fingerprints and SHA-256 hashes. Never install either build over the working Variant B application.

The original experiment used a local Flatpak build protocol. This publication repository omits its vendored dependency tree and private build paths; it is not a turnkey build bundle.

## 4. Create a safe runtime configuration

- Use a new, experiment-only TLS certificate and private key.
- Prefer a host-only network or an isolated trusted test network.
- Require authentication if the listener is reachable beyond the test host.
- Use an unused port and a new application data directory.
- Enable only `hardsignal_capture_diag=info`; avoid broad trace logging and any raw or encoded frame dump.
- Keep machine names, user names, addresses, portal object paths, and certificate paths out of retained excerpts.

An iPad test requires a path from the test client to the disposable server. Constrain that path with the test network and host firewall; do not weaken the active server's security settings.

## 5. Run the controlled sequence

For each arm, verify its binary hash before launch, then use the same selected display source, client, resolution, and observation window.

1. Run A and retain at least three periodic counter summaries before and after client connection.
2. Stop only the disposable A process.
3. Run B under the same conditions and retain equivalent summaries plus client/frame-pipeline events.
4. Stop only the disposable B process.
5. Repeat A as the final reversal check.

Do not restart PipeWire, a desktop portal, the installed Lamco service, or the working Variant B process as part of this sequence. If the disposable environment cannot proceed without changing shared service state, stop and redesign the isolation boundary.

## 6. Evaluate

Use the outcome criteria in [the methodology](experiment.md). Do not equate a `Streaming` state alone with frame delivery. Counters must show callback, dequeue, handoff, and receiver progress, and the client display must be confirmed separately.

The current research record has an observed A zero-callback result and a successful B run. The final A repeat remains pending, and B has not yet been validated with an unchanged desktop.

## 7. Sanitize before retention

Create short excerpts rather than copying complete logs. Remove or generalise:

- user and host names;
- home and application-data paths;
- IP addresses, ports, and network identifiers;
- portal request/session object paths and restore tokens;
- certificate and key paths;
- credentials, authentication material, environment dumps, and command histories;
- device serial numbers or unrelated hardware details; and
- raw pixel, audio, encoded-frame, or clipboard data.

Run a secret scanner and a manual sensitive-data review over every proposed file. Keep only evidence necessary to substantiate the stated counters, state transitions, and operator observation.
