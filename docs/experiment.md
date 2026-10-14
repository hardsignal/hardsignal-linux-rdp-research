# PipeWire DRIVER A/B methodology

## Research question

Does removing `StreamFlags::DRIVER` from the Lamco PipeWire input stream allow the stream's `process()` callback to run and frames to reach the RDP display pipeline in the observed environment?

## Compared variants

Both variants use Lamco RDP Server 1.4.5 and `lamco-pipewire` 0.6.13 with the same diagnostic instrumentation.

```text
Variant A: AUTOCONNECT | MAP_BUFFERS | DRIVER | RT_PROCESS
Variant B: AUTOCONNECT | MAP_BUFFERS | RT_PROCESS
```

These are source-derived `pw_stream_connect` expressions, not values parsed from runtime logs. Both binary artifacts retain an unchanged hardcoded message that names `DRIVER`; that message is accurate for A but stale for B and must not be used to identify B's effective flags.

The sole intended A-to-B source change is removal of `StreamFlags::DRIVER` from the `pw_stream_connect` flags. The server trees, lockfiles, dependency graph, build features, toolchain protocol, and diagnostic code are controlled between arms.

## Instrumentation

The shared patch adds manager-wide cumulative `AtomicU64` counters:

| Counter | Meaning |
| --- | --- |
| `process_entries` | Entries into the PipeWire `process()` callback. |
| `buffers_dequeued` | Successful calls returning a PipeWire buffer. |
| `dequeues_empty` | Callback entries for which no buffer was available. |
| `handoffs_ok` | Frames successfully placed on the existing downstream channel. |
| `handoffs_full` | Frame handoffs rejected because the channel was full. |
| `handoffs_disconnected` | Frame handoffs rejected because the receiver was disconnected. |
| `receiver_delivered` | Frames popped by either manager receive API, before downstream corrupted-frame filtering. |

The real-time callback performs only relaxed native 64-bit atomic operations on the x86_64 experiment target and inspects the existing handoff result. Reporting occurs about every five seconds on the PipeWire main-loop thread, including when `process()` never runs. No logging, allocation, lock, wait, or trigger operation was added to the real-time callback.

The shared patch also removes an existing 32-byte pixel preview that allocated strings in the callback. This removal is identical in both arms and is not the experimental variable.

## Controls

- Application source: Lamco RDP Server 1.4.5 at the recorded source revision.
- Capture crate: `lamco-pipewire` 0.6.13 from the verified crate archive.
- Lockfiles: byte-identical across A and B.
- Intended feature set: `h264,vaapi,opus-libopus,libei,gui` with default features disabled.
- Target protocol: Linux x86_64 with the documented Flatpak 25.08 pins; no retained post-build record independently verifies those Flatpak commits.
- Compiler mode: both binaries confirm the same Rust 1.98.1 compiler through embedded strings; Cargo and clang remain documented protocol pins without retained post-build verification records.
- Client and display conditions: same RDP client, selected source, resolution, and observation interval for each completed pair.
- Logging: the narrow `hardsignal_capture_diag=info` target only; no broad trace logging or frame dumps.

The experiment must use fresh, isolated runtime state for each arm. It must not overwrite, stop, restart, or reuse the working Variant B installation or its configuration.

## Planned sequence

1. Verify source, patch, lockfile, toolchain, and binary hashes before launch.
2. Start Variant A in a disposable environment and record several diagnostic intervals before and after the client connects.
3. Stop only the disposable Variant A process.
4. Repeat with Variant B using the same client and conditions.
5. Perform a final Variant A repeat to test reversibility and reduce ordering or transient-state confounds.
6. Retain only sanitised counter and pipeline excerpts in a publication repository.

Steps 1–4 produced the observations described in this repository. Step 5, the final A repeat, remains pending.

The completed runs use two distinct forms of artifact attribution. A pre-launch SHA-256 check identifies the executable selected for a run. After launch, the server's startup banner reports its embedded build stamp. The embedded stamp supports build attribution but is not a hash and does not replace the pre-launch verification.

The successful B observation did not validate sustained delivery while the desktop remained unchanged. The upstream source says DRIVER is intended to keep the graph clock active in that condition, so a separate static-desktop check is required to assess whether Variant B introduces a regression.

## Outcome criteria

Capture success requires all of the following:

- `process_entries` increases;
- `buffers_dequeued` increases;
- `handoffs_ok` and `receiver_delivered` increase;
- the server display pipeline reports processing or sending frames; and
- the operator confirms visible desktop content at the intended RDP client.

A zero-callback result requires repeated summaries during an otherwise active capture attempt with `process_entries=0`, not merely an absence of verbose callback messages.

## Evidence classes

- **Diagnostic** — numeric counters or explicit server state transitions.
- **Operator-observed** — a human observation, such as visible output on the iPad, not independently encoded in server logs.
- **Inferred** — an explanation consistent with the A/B result but not directly measured.

These classes are kept separate in the findings.
