# Linux RDP PipeWire DRIVER research

This repository records a controlled A/B investigation into a Linux RDP capture stall involving `lamco-rdp-server` 1.4.5, `lamco-pipewire` 0.6.13, and the PipeWire `StreamFlags::DRIVER` flag.

The repository contains documentation, minimal source patches, hashes, and short sanitised evidence excerpts. It deliberately excludes binaries, vendored dependencies, full logs, private configuration, certificates, credentials, tokens, and machine-specific paths or addresses.

## Status

Publication-ready research record. Final controlled Variant A repeat and static-desktop validation of Variant B are pending.

The evidence supports this bounded result:

| Variant | Source-derived PipeWire stream flags | Result | Evidence status |
| --- | --- | --- | --- |
| A | `AUTOCONNECT | MAP_BUFFERS | DRIVER | RT_PROCESS` | The observed run reached the relevant capture state but recorded zero `process()` callbacks and delivered no frames. | Observed result; the final controlled A repeat is still pending. |
| B | `AUTOCONNECT | MAP_BUFFERS | RT_PROCESS` | Capture callbacks, dequeues, and receiver delivery advanced; a remote client became active; the operator confirmed the desktop was displayed on an iPad. | Diagnostic counters and server pipeline events recorded; iPad display is an operator observation. |

These flag lists come from the respective `pw_stream_connect` source expressions, not from runtime log messages. Both binary artifacts retain the same hardcoded log message naming `DRIVER`; it describes A but is stale and incorrect for B.

Variant B produced a first five-second diagnostic sample of 392 callback entries, 392 dequeued buffers, 392 successful handoffs, and 392 receiver deliveries, with no empty dequeues, full handoffs, or disconnected handoffs. Later samples continued increasing. See [the findings](docs/findings.md) and [sanitised excerpts](evidence/).

## Interpretation

Verified: removing `StreamFlags::DRIVER` was the only intended source difference between the two prepared arms, and Variant B captured and delivered frames in the observed environment.

Hypothesis: advertising this capture stream as a PipeWire graph driver prevented a producer-driven capture graph from scheduling the process callback correctly in this environment.

Not yet established: general causality across PipeWire implementations, compositors, portals, or hardware. A final repeat of Variant A under the same controlled conditions remains necessary before treating the A/B sequence as complete. Variant B has also not been validated while the desktop remains unchanged, so removal of DRIVER remains a candidate experimental change rather than a universally validated upstream fix.

## Objectives

- Preserve a reviewable account of the DRIVER A/B experiment.
- Make verified observations visibly distinct from hypotheses and operator reports.
- Publish only the smallest source changes needed to reproduce the comparison.
- Preserve toolchain, dependency, source, and binary provenance without redistributing build outputs or dependency trees.
- Provide a safe reproduction plan that does not disturb an installed or working RDP service.

## Repository map

- [`docs/experiment.md`](docs/experiment.md) — experimental design and counter definitions.
- [`docs/findings.md`](docs/findings.md) — results, inference boundary, and pending work.
- [`docs/build-provenance.md`](docs/build-provenance.md) — source identity, Rust 1.98.1 substitute toolchain, dependency pins, and hashes.
- [`docs/reproduction.md`](docs/reproduction.md) — safe, isolated reproduction procedure.
- [`patches/`](patches/) — minimal source changes and their licensing boundary.
- [`evidence/`](evidence/) — sanitised, non-verbatim diagnostic excerpts only.
- [`THIRD_PARTY_NOTICES.md`](THIRD_PARTY_NOTICES.md) — upstream attribution.

## Scope and exclusions

This is a research record, not a Lamco distribution and not an installation bundle. It does not include or grant rights to Lamco products, third-party dependencies, binaries, private build caches, raw logs, live configuration, or secrets. Obtain upstream sources and dependencies from their authorised distribution channels and comply with their licences.

No command in this repository should be run against an active installation. Reproduction requires a disposable environment and separate authorisation for compilation and live testing.

## Licence

Hardsignal-authored documentation and original repository material are provided under the [Apache License 2.0](LICENSE). Patch hunks derived from upstream projects remain governed by the applicable upstream licences; see [`patches/README.md`](patches/README.md) and [`THIRD_PARTY_NOTICES.md`](THIRD_PARTY_NOTICES.md).
