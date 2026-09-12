---
layout: solution
lang: en
docno: CY-SOL-001
title: "Deterministic Fault Injection: Five Fault Primitives, Reproduced at Tick N"
description: "Cyclone deterministic fault injection: five primitives (drop, delay, corrupt, freeze, CRC error) in declarative YAML — same fault, same tick, every run."
permalink: /en/solutions/fault-injection/
zh_url: /solutions/fault-injection/
---

<h2><span class="sec-no">§1 · PROBLEM</span>The Three Deadlocks of Manual Fault Injection</h2>

Almost every ADAS / robotics test team has done fault injection, and in a strikingly uniform way: pull the cable, patch the code, write a one-off script. That road has three deadlocks you cannot untangle:

1. **No precise reproduction.** Inject the same fault a second time and the timing, intensity, and duration window all differ from the first. Want to re-run it to verify the fix after the FAIL? Sorry — "that fault" no longer exists.
2. **Conclusions don't align.** The FAIL one engineer produces won't reproduce on a colleague's machine — the code didn't change, the "luck" did. In the review meeting, nobody can convince anybody.
3. **Scenarios are not assets.** One-off scripts sit on personal laptops: the owner can't take a vacation, nobody dares touch the script — let alone review, version, or diff it.

<h2><span class="sec-no">§2 · SOLUTION</span>Cyclone's Approach: Fault as Code</h2>

Cyclone's fault-injection layer is called FaultPipe, and its core idea fits in one sentence: **faults are declared, not handcrafted**.

<h3>Five Fault Primitives <span class="st"><i class="sdot g"></i>Supported</span></h3>

| Primitive | Behavior | Typically Simulates |
|---|---|---|
| `Drop` | Frame drops | Sensor packet loss, bus overload |
| `Delay` | Delayed delivery | Network congestion, ECU scheduling jitter |
| `CorruptField` | Field corruption | Sensor drift, signal anomalies |
| `Freeze` | Freezing | Hung node, watchdog never fires |
| `CorruptCrc` | CRC errors | EMC interference, poor harness contact |

<h3>Tick-Precise Triggering</h3>

Declare the trigger by time window, frame count, or conditional expression. On a 1 ms schedule, **the same fault fires at tick N, guaranteed** — not "somewhere around then," but the exact slot the deterministic scheduler promises. Rerun with the same seed and the evidence logs are byte-identical (SHA-256 verifiable).

<h3>Declarative YAML Configuration</h3>

```yaml
# cases/aeb_frame_drop.yaml — a regression test case
fault:
  type: Drop
  target: front_camera.frames
  trigger: { at_tick: 500, duration: 120 }
  pattern: burst        # burst drops, simulating a loose connector
verdict:
  metric: recall_near
  threshold: 0.98       # threshold derived via margin analysis, see §4
```

Fault scenarios as code: reviewable, versionable, diffable. A new engineer taking over doesn't need "the old hand's touch."

<h2><span class="sec-no">§3 · WHY IT MATTERS</span>Only Deterministic Injection Makes a Verdict Possible</h2>

The finish line of fault injection isn't "injected" — it's "judged." Cyclone welds injection and statistical verdict into a single pipeline: fault injection → metric degradation → verdict flip → evidence output. The verdict engine uses the Wilson lower bound and SPRT sequential testing, turning "ran it a few times, felt fine" into "fails at 95% confidence."

- [30 Runs, 27 Passes: Why That Doesn't Count as Verified →](/en/blog/30-runs-27-pass/)
- [Wilson Lower-Bound Verdicts: An Honest Ruler for Pass Rates →](/en/blog/wilson-lower-bound/)
- [SPRT and LLR: Sequential Testing That Delivers the Verdict as Soon as the Evidence Is In →](/en/blog/sprt-llr/)

<h2><span class="sec-no">§4 · HONEST BOUNDARIES</span>Honest Boundaries</h2>

- **Physical environmental testing is not software's job.** High/low temperature, vibration, EMC, and salt spray (ISO 16750 / GB/T 28046) belong to climate chambers and rigs. What Cyclone does is the half of environmental stress **projected onto the bus**: drift, frame drops, bit flips, node freezes (some injection methods <span class="st"><i class="sdot a"></i>Prototype</span> / <span class="st"><i class="sdot o"></i>Planned</span>, see [Homepage §3.5](/en/#testmap)).
- **Interface status**: SocketCAN / UDP / YAML DSL / JUnit XML <span class="st"><i class="sdot g"></i>Supported</span>; DDS <span class="st"><i class="sdot a"></i>Prototype</span>; SOME/IP, UDS, ASAM XIL API <span class="st"><i class="sdot o"></i>Planned</span>. During the POC phase we evaluate priorities against your bus environment.
- **Thresholds need calibration.** The 0.98 in the example above is a demo value; the formal acceptance threshold must be derived via [margin analysis](/en/blog/margin-analysis/) against your SUT baseline — a standard POC service item.
