# COSMIC portal memory growth case study

## Status

Investigation record. The COSMIC desktop portal process was observed holding approximately 12 GiB of private anonymous memory that continued to grow while no RDP client was connected. Operator-supplied terminal observations from a controlled maintenance window report that restarting the portal reclaimed the memory and that remote desktop service was restored.

**The underlying leak has not been fixed.** It has not been conclusively attributed to a specific allocation, object type, code path, or component. Restarting the portal removed the accumulated memory, but a post-maintenance measurement shows idle growth resuming at a similar rate (5.14 MiB/min against 4.87 MiB/min before maintenance).

This case study is separate from the [PipeWire DRIVER A/B research](findings.md). It does not change that research's results, its supported conclusion, or its pending work: the final controlled Variant A repeat and static-desktop validation of Variant B.

## Environment

| Component | Recorded identity |
| --- | --- |
| `xdg-desktop-portal-cosmic` | `1.10.0~1791324839~24.04~9d35e63` (upstream commit `9d35e6300b69cf06da723a502a9a56604a022786`) |
| `cosmic-comp` | `0.1~1791227611~24.04~41497b4` |
| `xdg-desktop-portal` | 1.18.4 |
| PipeWire | 1.6.8 |
| WirePlumber | 0.5.12 |
| Lamco RDP Server | 1.4.5-based local deployment with unpublished local modifications |
| Portal allocator | glibc `malloc`; no alternative allocator was mapped into the process |
| Captured output | One 1280×720 output at 60 Hz |

The RDP server held one long-lived COSMIC RemoteDesktop/ScreenCast portal session and one PipeWire capture stream. Clients connected and disconnected over the session's lifetime, including from an iPad.

## Evidence classes

This document uses the repository's existing evidence classes, with three additions for this investigation (source observation, operator-supplied terminal observation, and proposed):

- **Diagnostic** — read-only measurements from `/proc`, package metadata, or explicit log events.
- **Source observation** — behaviour read from upstream source at the identified version. It shows what the code does, not what happened at runtime.
- **Operator-observed** — a human report that no retained diagnostic record independently substantiates.
- **Operator-supplied terminal observation** — a value the operator read from terminal output and reported. The output is not retained here, and the value has not been independently reproduced.
- **Inferred** — an explanation or derived figure consistent with the evidence but not directly measured.
- **Proposed** — an experiment that has not yet been run.

## Direct measurements

All measurements were taken read-only. No debugger, `ptrace` attach, signal, restart, or configuration change was used during the investigation.

| Measurement | Value | Class |
| --- | --- | --- |
| Portal private anonymous memory at investigation | ≈12 GiB (point readings at different times: 11.8 GiB RSS; 12.19 GiB private dirty anonymous) | Diagnostic |
| Share in large anonymous arena mappings | ≈11.5 GiB, as 92 fully resident 128 MiB regions | Diagnostic |
| Main `[heap]` mapping | ≈11 MiB | Diagnostic |
| GPU, file-backed and `memfd` mappings | Negligible | Diagnostic |
| Swap used by the portal | 0 | Diagnostic |
| Active RDP clients during sampling | 0 (last client disconnected ≈14.7 h earlier) | Diagnostic |
| Initial idle growth check | +10.9 MiB over 2 min (≈5.5 MiB/min) | Diagnostic |
| Idle growth, first 150 s of the sampled window | 5.47 MiB/min | Diagnostic |
| Idle growth, full sampled window (31 samples at 10 s, 300 s) | **4.87 MiB/min** | Diagnostic |
| Portal sessions / PipeWire streams over the observed lifetime | 1 / 1; not recreated across 11 client reconnects | Diagnostic |
| Stream lifetime / cumulative client-connected time | ≈46.4 h / ≈10.4 h | Diagnostic |
| Frame arrival at the RDP server while idle | ≈84 frames/s | Diagnostic (heartbeat PTS-interval statistics) |
| Retained memory per captured frame | ≈1.1 KB | Inferred (idle rate ÷ idle frame rate) |
| RDP server console log | 3.09 GB | Diagnostic |
| IronRDP per-packet DEBUG lines in that log | ≈13.4 M lines, 1.61 GB (52%) | Diagnostic |
| Display-pipeline heartbeat lines | ≈35 k lines, 11.8 MB (0.4%) | Diagnostic |
| Duplicate application log file | ≈1.9 GB (≈5 GB combined) | Diagnostic |
| Log growth rate | ≈290 MB/h with a client connected; ≈6 MB/day idle | Diagnostic |
| Portal RSS before maintenance | ≈11.9 GiB | Operator-supplied terminal observation |
| Portal RSS immediately after restart | ≈169 MiB | Operator-supplied terminal observation |
| Memory reclaimed | ≈11.7 GiB (derived from the two RSS readings) | Inferred from operator-supplied values |
| Available system RAM before / after | 9.8 GiB / 21 GiB | Operator-supplied terminal observation |
| Post-maintenance service | Original wheel controller SHA-256 verified; restored RDP server identity audit passed; iPad remote desktop confirmed | Operator-supplied terminal observation |
| **Post-maintenance** idle growth, full window (restarted portal, ≈24 min after restart; 31 samples at 10 s, 301 s; 0 RDP clients connected throughout) | **5.14 MiB/min** (`RssAnon` +25.8 MiB) | Diagnostic |
| Post-maintenance idle growth, first / second half | 4.71 / 5.58 MiB/min | Diagnostic |
| Post-maintenance `VmRSS` / `Private_Dirty` change over the same window | +25.8 MiB / +25.8 MiB; `VmSwap` 0 throughout | Diagnostic |
| PipeWire capture nodes during the post-maintenance window | Portal screencast source and RDP server input stream both in `running` state | Diagnostic |

Notes on the measurements:

- **Growth rate range.** The 4.87–5.5 MiB/min range reflects different windows rather than different methods. An earlier report quoted 5.47 MiB/min from the first part of the sample; across the complete 300-second window the rate was 4.87 MiB/min, and the second half was slower (≈4.3 MiB/min). The growth was steady but not strictly linear.
- **Memory layout.** The large regions are consistent with glibc per-thread malloc arenas. The active growth was observed in a 64 MiB reserved arena whose writable portion expanded during sampling. That the 128 MiB regions are adjacent full arenas merged by the kernel is an inference from their size and layout.
- **Idle growth.** Growth continued with no RDP client connected. If memory had grown only while clients were connected, the accumulated total would have required about 19 MiB/min during connected time. At the measured idle rate, the stream's ≈46.4 h lifetime predicts about 11.6 GiB, consistent with the observed total. This consistency check is inferred, not a measurement of the full history.
- **Logs.** The portal's journal showed no panics or protocol errors during the observed period. PipeWire and WirePlumber logged nothing after startup. The compositor logged a small number of texture-import errors with no established relation to the growth.
- **Maintenance outcome.** The before and after RSS, available RAM, hash verification, identity audit, and iPad values are operator-supplied terminal observations. They have not been independently reproduced, and their raw terminal output is not published.
- **Post-maintenance rate.** The post-maintenance sample was taken read-only against the restarted portal process, whose PID and start time stayed unchanged for the whole window. No RDP client connection was established at any sample. Its 5.14 MiB/min average is about 6% above the pre-maintenance 4.87 MiB/min and within the half-window variation seen before maintenance (≈4.3–5.5 MiB/min). `RssAnon`, `VmRSS`, and `Private_Dirty` rose by the same amount, so the new growth is again private anonymous memory. The idle frame rate was not recorded in this window, so the per-frame figure was not recomputed.

## Source-code observations

These observations describe upstream source at the identified versions. No upstream source is reproduced here.

### xdg-desktop-portal-cosmic (GPL-3.0), `src/screencast_thread.rs` at `9d35e63`

- The PipeWire `process` callback dequeues an available buffer and asks the compositor to capture a new frame into it. Capture is therefore driven by buffers becoming available to the portal, which happens when the consumer returns them.
- The negotiated video format offers a fixed 60/1 framerate, and a maximum-framerate property is marked as not yet implemented (`TODO max framerate`). Nothing in this path was found to cap capture to a rate below how fast buffers are returned.

### Lamco RDP Server 1.4.5 (BUSL-1.1), display handler

- When no client is active, the display loop continues to receive frames from the PipeWire capture stream and discards them, sleeping 1 ms per iteration. The stream is not paused or disconnected when the last client leaves.
- The display-pipeline heartbeat is emitted every 1,000 loop iterations rather than on a timer. With the 1 ms idle sleep, the loop ran about 220 iterations per second, giving a heartbeat roughly every 4.5 s with unchanged counters. The heartbeat is noisy but is not the main source of log volume.

### Logging configuration

The deployment's launch environment set a `RUST_LOG` filter with `ironrdp_server=debug` and several `trace` targets. This overrode the `info` level in the server's configuration file. The configuration and launcher are private and are not published.

## Engineering hypotheses

None of the following is established.

1. **Per-frame retention in the portal's capture path (most likely).** The memory is private anonymous heap in glibc arenas, not GPU, dmabuf, or file-backed memory. It grows while idle, and the stream was never recreated. Together these are consistent with a small allocation being retained for each captured frame on a portal worker thread. No specific object or code path has been identified. The retained memory's contents were not inspected because doing so would have required elevated debugging privileges on the production host.
2. **Lamco's frame handback contributes unnecessary capture activity.** The server drains and returns buffers at full speed with no client connected, and the portal captures whenever a buffer is returned. Together these would keep the portal capturing (≈84 frames/s observed) while idle. If hypothesis 1 holds, this would make idle periods accumulate memory at roughly the same rate as connected periods. Pausing capture while idle would be a mitigation, not a fix: growth would still occur while clients are connected.
3. **Lowering Lamco's requested framerate may not reduce growth proportionally.** The portal does not apply a maximum framerate, and capture follows buffer return. A server-side framerate reduction that does not slow buffer return may therefore have little effect. Advertising a lower fixed rate could also break format negotiation. This is untested.
4. **Relation to the DRIVER research.** Whether the Lamco capture stream acts as a PipeWire graph driver affects who schedules capture callbacks. Any idle-pause or rate change interacts with the pending [static-desktop validation](findings.md#limitations-and-pending-work) of DRIVER removal and must not be evaluated in isolation.

Not supported by the measurements: growth in GPU, dmabuf, or file-backed mappings; swap use; and repeated portal session or stream recreation.

## Unresolved questions

- Which allocation or object is retained per frame, and in which component: the portal itself, a Wayland or PipeWire client library, or the compositor protocol interaction?
- Does growth scale with captured frame count or with wall-clock time?
- Growth resumed at a similar idle rate after the restart. Is it still about 1.1 KB per captured frame, and is the connected-session rate the same?
- Does the growth also occur with other portal consumers, such as browser screen sharing, or only with a long-lived RemoteDesktop session?
- Does it occur on later `xdg-desktop-portal-cosmic` builds?
- Is the ≈84 frames/s idle capture rate, above the output's 60 Hz refresh, expected portal behaviour or an artefact of the handback loop?

## Proposed experiments

All experiments below are proposed and have not been run. Allocation profiling, capture-pausing changes, or allocator tuning belong in a disposable session or machine, never on the production host.

1. **Connected-session baseline.** The post-maintenance idle rate has been measured. Use the read-only sampler below to measure the restarted portal's growth rate with a client connected, and record the RDP server's frame rate over both idle and connected windows.
2. **Frame-count correlation.** In a disposable environment, vary how fast the consumer returns buffers and test whether growth per captured frame stays near constant.
3. **Pause on idle.** Pause the RDP server's capture stream when the last client disconnects and resume it on connect. Check that idle growth stops, the portal session survives without a new permission prompt, and reconnection and input still work. Run this together with the DRIVER static-desktop check.
4. **Allocation profiling.** Run the portal under an allocation profiler such as heaptrack in a disposable session and identify the retaining call stack.
5. **Allocator behaviour.** Test whether the memory is genuinely retained or only unreturned free memory, for example by limiting glibc arenas or triggering `malloc_trim` in a disposable session. A steady rise in anonymous resident memory under constant load suggests live retention, but this has not been proven.
6. **Logging reduction.** Lower `ironrdp_server` to `info`, make the heartbeat time-based or change-triggered, write logs to one destination, and add size-based rotation. IronRDP per-packet DEBUG lines alone accounted for about half of the console log. Any binary or controller change produces new artifact hashes and needs new acceptance evidence.
7. **Upstream report.** File an issue against `xdg-desktop-portal-cosmic` with the memory-layout summary, the idle rate, and the per-frame estimate, with the pre- and post-maintenance idle rates. Strengthen it with experiment 1 or 4, or with a check on a current build.

## Read-only monitoring methodology

This sampler reads only `/proc` for a process owned by the same user. Run it as the desktop session user, and save its output to `samples.txt`, for example `sh portal-mem-sample.sh 10 31 > samples.txt`. The process is matched on its full command line because the kernel truncates process names to 15 characters, so `pgrep -x xdg-desktop-portal-cosmic` never matches. It does not attach to, signal, or modify the portal, and it prints only timestamps and sizes.

```sh
#!/bin/sh
# Usage: portal-mem-sample.sh [interval_seconds] [samples]
# Output columns: unix_time  rss_anon_kib  private_dirty_kib  swap_kib
interval=${1:-10}
samples=${2:-31}
pid=$(pgrep -u "$(id -u)" -f '(^|/)xdg-desktop-portal-cosmic( |$)' | head -n 1)
[ -n "$pid" ] || { echo "portal not running" >&2; exit 1; }

i=0
while [ "$i" -lt "$samples" ]; do
    anon=$(awk '/^RssAnon:/ {print $2}' "/proc/$pid/status")
    dirty=$(awk '/^Private_Dirty:/ {print $2}' "/proc/$pid/smaps_rollup")
    swap=$(awk '/^VmSwap:/ {print $2}' "/proc/$pid/status")
    echo "$(date +%s) $anon $dirty $swap"
    i=$((i + 1))
    if [ "$i" -lt "$samples" ]; then
        sleep "$interval"
    fi
done
```

Compute the rate from the first and last samples, and report the window length with it. The published idle rates were derived from `RssAnon` (column 2). `Private_Dirty` (column 3) is recorded as a cross-check.

```sh
awk 'NR==1 {t0=$1; m0=$2} {t=$1; m=$2}
     END {if (t > t0) printf "%.2f MiB/min over %d s\n", (m-m0)/1024/((t-t0)/60), t-t0}' samples.txt
```

Guidance:

- Record the number of connected RDP clients for each window. Compare idle and connected windows separately, and report windows of at least five minutes.
- For layout analysis, aggregate `/proc/<pid>/smaps` by mapping type and size. Do not publish raw mappings: they expose addresses, paths, and process metadata.
- Record frame rate from the RDP server's own diagnostics over the same window to derive memory per frame.
- Do not use debuggers, `gcore`, `ptrace`-based tools, or allocation profilers on a production portal. Do not truncate or rotate active logs during measurement.
- Before retaining any output, apply the sanitisation rules in the [reproduction guide](reproduction.md#7-sanitize-before-retention).

## Maintenance outcome

The operator ran a separately approved, controlled maintenance window and supplied the following terminal observations. They are reported here as supplied and have not been independently reproduced:

| Observation | Before | After |
| --- | ---: | ---: |
| Portal RSS | ≈11.9 GiB | ≈169 MiB, immediately after restart |
| Available system RAM | 9.8 GiB | 21 GiB |

- The SHA-256 of the original wheel controller (the deployment's sealed launch controller) was verified. The hash value is not published.
- An identity audit of the restored RDP server passed.
- Remote desktop operation from the iPad was confirmed afterwards.

The before and after RSS readings imply about 11.7 GiB reclaimed. The restart restored memory headroom only. No code, logging configuration, or capture behaviour was changed, and **the memory leak is not fixed**. A read-only post-maintenance measurement confirms the growth has resumed: 5.14 MiB/min with no client connected, against 4.87 MiB/min before maintenance (see [Direct measurements](#direct-measurements)). At that rate, the portal would be expected to return to about 12 GiB in roughly 40 hours of continuous capture, consistent with the ≈46 h accumulation observed before maintenance. This projection is inferred and assumes the rate stays constant.

## Upstream references

- `xdg-desktop-portal-cosmic`: <https://github.com/pop-os/xdg-desktop-portal-cosmic>
- Screencast thread at the tested commit: <https://github.com/pop-os/xdg-desktop-portal-cosmic/blob/9d35e6300b69cf06da723a502a9a56604a022786/src/screencast_thread.rs>
- Issue #384, a screen-share freeze and portal panic on the same 1.10.0 build involving the per-frame capture lifecycle. It concerns crashes, not memory growth, but appears to involve the same capture code path: <https://github.com/pop-os/xdg-desktop-portal-cosmic/issues/384>
- Issue #351, a large memory leak following a screenshot-portal panic. No panic was observed here, so this is a distinct trigger: <https://github.com/pop-os/xdg-desktop-portal-cosmic/issues/351>
- PipeWire DMA-BUF sharing and buffer negotiation: <https://docs.pipewire.org/devel/page_dma_buf.html>
- Lamco RDP Server: <https://github.com/lamco-admin/lamco-rdp-server>
- IronRDP: <https://github.com/Devolutions/IronRDP>

No matching upstream report of a slow, panic-free idle leak was found at the time of investigation.

## Publication boundary

This document omits process identifiers, memory addresses, user and host names, network addresses, private paths, launcher and configuration contents, session identifiers, and raw logs. The retained figures are rounded summaries of private read-only samples and log classifications. That raw evidence stays outside this repository.
