# Findings

## Verified observations

### Variant A: DRIVER present

The observed Variant A run's connect expression used:

```text
AUTOCONNECT | MAP_BUFFERS | DRIVER | RT_PROCESS
```

This flag list is derived from the Variant A source code; it is not a runtime log excerpt.

The retained raw log records the stream entering streaming state and the RDP client becoming active. Across 91 periodic diagnostic summaries, all seven counters remained zero: 43 summaries preceded the client-active event and 48 followed it. The server continued to report that no video frames were received, and no captured desktop image was delivered to the RDP client during that observation.

This is a verified observation from the conducted experiment, but the final controlled repeat of A has not yet been completed. The result should therefore be described as an observed A failure, not a completed reversal proof.

### Variant B: DRIVER removed

Variant B changed only the effective connect expression:

```text
AUTOCONNECT | MAP_BUFFERS | RT_PROCESS
```

This flag list is likewise derived from source code, not runtime output. Both A and B binaries contain the same hardcoded message claiming `AUTOCONNECT | MAP_BUFFERS | DRIVER | RT_PROCESS`; the message is accurate for A but stale and incorrect for B because the Variant B patch changes the connect expression without changing the message.

The stream entered streaming state. The first retained five-second summary reported:

```text
process_entries=392
buffers_dequeued=392
dequeues_empty=0
handoffs_ok=392
handoffs_full=0
handoffs_disconnected=0
receiver_delivered=392
```

Later retained samples continued to increase, including a sample with 14,952 callback entries, 14,952 dequeued buffers, 14,952 successful handoffs, and 14,952 receiver deliveries. Server events also recorded an active remote client and continuing frame processing and sending.

The operator confirmed that the Linux desktop was visibly displayed on the iPad. That visual confirmation is an operator observation; the server diagnostics independently establish callback activity and downstream frame processing, but cannot identify what the human saw on the client screen.

## Comparison

| Measurement | Variant A | Variant B |
| --- | ---: | ---: |
| `StreamFlags::DRIVER` | Present | Absent |
| `process()` callback entries | 0 in the observed run | Increasing |
| Successful buffer dequeues | 0 | Increasing |
| Successful downstream handoffs | 0 | Increasing |
| Receiver deliveries | 0 | Increasing |
| Server frame pipeline | No captured frames delivered | Active and sending frames |
| iPad display | No desktop image observed | Desktop image observed |

## Supported conclusion

Within this experiment, removing `StreamFlags::DRIVER` was associated with the transition from zero capture callbacks to successful capture, downstream delivery, RDP frame processing, and visible output on the iPad.

The source comparison establishes that DRIVER removal was the sole intended A/B code variable. It does not by itself prove a general PipeWire defect or guarantee that the same change is appropriate for every graph or application.

## Hypothesis

The upstream source comment explains the rationale for `StreamFlags::DRIVER`: keep the graph clock active so frames continue at the negotiated rate even when the desktop is static, rather than relying only on damage-driven updates. That rationale identifies a separate risk in Variant B: removing DRIVER may restore capture in the observed case yet regress delivery when the desktop does not change.

In the observed run, the capture source was expected to be producer-driven. A plausible explanation is that making the Lamco input stream a driver created a scheduling role mismatch in this graph, leaving the stream nominally connected or streaming without invoking its process callback. Removing DRIVER allowed the upstream producer's scheduling to drive callbacks normally.

This mechanism is a hypothesis. The retained counters show where progress stopped and resumed; they do not directly measure PipeWire's scheduling decision.

## Limitations and pending work

- The final controlled Variant A repeat is pending.
- Variant B has not been validated with an unchanged desktop, so the upstream rationale for DRIVER and the corresponding regression risk remain untested.
- The iPad display result is a documented operator observation, not video or screenshot evidence.
- The experiment covers one observed software and desktop environment; it does not establish cross-compositor or cross-version behaviour.
- The rebuilt A/B arms used Rust 1.98.1 as a substitute for the original installed build's Rust 1.98.0. This does not confound A versus B because both rebuilt arms use the same substitute, but it does limit comparison with the installed binary.
- Build metadata includes time and Git-derived values, so binary bit-identity with any original build is not claimed.
- The diagnostic counters are cumulative relaxed-atomic samples, not transactional snapshots. A one-frame difference between producer and receiver counters can occur while a frame is in flight.

Until the final A repeat and static-desktop testing are complete, the strongest accurate wording is: **Variant B succeeded after DRIVER removal in the observed conditions, while the observed Variant A run recorded zero callbacks; reversal confirmation remains pending, and the change is not a universally validated upstream fix.**
