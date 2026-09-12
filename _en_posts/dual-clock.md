---
title: "Dual-Mode Clock: How to Have Both Offline Reproducibility and Online Realism"
description: "One engine, two physics: the dual-mode clock gives you both offline reproducibility and online realism — virtual clock for replay, real clock for the field."
series: "Architecture Note"
tn: "TN-05"
date: 2026-09-08
zh_url: "/blog/dual-clock/"
cta_text: "Download the scenario pack and run it yourself"
cta_url: "/en/#scenariopack"
---


Test platform architecture has one dilemma you can't route around:

- If you want **reproducibility**, you have to make the environment "fake" — data replay, controlled time — but that drifts away from the real world;
- If you want **realism**, you have to connect the real environment — real buses, real ECUs, a real clock — but every run is different, and the bug you caught can never be reproduced.

Most platforms pick one side of this choice. Cyclone Core's answer is **don't choose — build a clock**: a dual-mode clock that lets the same engine walk in two kinds of physics. This post lays out that architectural decision.

## 1. First, Separate the Two Knobs: Clock × Environment

"Offline/online" is actually a combination of two independent knobs:

| | Virtual clock | Real clock |
|---|---|---|
| **Replayed data** | ① Offline mode: a fully controlled lab | ③ Rarely used (not covered here) |
| **Live bus** | ④ Meaningless (real data won't wait for you) | ② Online mode: the real world, live |

Only two cells really matter: **offline mode = virtual clock + replay files**, **online mode = real clock + live bus**.

## 2. Offline Mode: Time Is Injected by the Platform

In offline mode, **time doesn't pass — it is injected**. The scheduler advances the entire world at a fixed step (one tick per 1 ms): emit one frame of data, advance the system under test by one step, write one log entry. Sensor data comes from file replay; the whole world is closed.

This earns offline mode its crown: **bit-level determinism**. The same scenario YAML, the same seed, run a thousand times — the logs are byte-for-byte identical, and the verdict is necessarily identical. Three engineering dividends follow:

1. **Bugs are reproducible forever** — caught means pinned; there is no "works on my machine";
2. **CI-friendly** — results are cacheable (content-addressed PASS), the pipeline doesn't flap;
3. **Free large-scale sampling** — run a seed matrix 300 times for a statistical verdict (see Parts 1–3 of this series: [Part 1](/en/blog/30-runs-27-pass/), [Part 2](/en/blog/wilson-lower-bound/), [Part 3](/en/blog/sprt-llr/)); machine time is the entire cost.

## 3. Online Mode: Time Is Injected by Physics

In online mode, the clock obeys the physical world (the OS clock, the real cadence of the bus), and data comes from real nodes on SocketCAN/Ethernet. What you get is what offline can never give you:

- **Real timing**: end-to-end latency, missed deadlines, jitter — in a virtual clock "1 tick is always 1 tick," which proves nothing about how fast the CPU actually ran;
- **Real hardware behavior**: the startup twitch of a real ECU, bus load, the temperament of the protocol stack;
- **Field evidence**: the problem happened in a real environment, and the evidence chain is collected from that real environment.

## 4. The Honest Cost: Why Online + Real ECU Must Give Up Bit-Level Determinism

This is the most important section of this post, because many platforms lie about exactly this.

Once you connect a real ECU, **bit-level determinism is physically impossible**. There are at least five leak points:

1. **Sensor and sampling noise**: ADC quantization, thermal noise — two samplings of the same physical quantity will not be bit-identical;
2. **Clock drift**: every node's crystal has its own ppm-level deviation; a distributed system has no single "now";
3. **Bus arbitration**: CAN arbitration depends on each node's instantaneous state, so the precise arrival order of frames drifts;
4. **Scheduling jitter**: the scheduling latency of a non-hard-real-time OS is a random variable;
5. **Physical effects like temperature**: hardware behavior drifts with temperature — even the lab air conditioning is a variable.

**Only by admitting this can we talk about the three things online mode genuinely preserves:**

- **Evidence authenticity**: the logs are collected from a real bus; the source of the evidence chain is real;
- **Statistical verdict stability**: a single run is not reproducible, but the Wilson lower bound and the SPRT verdict are statistically stable — the verdict engine was designed for a noisy world in the first place (Parts [1](/en/blog/30-runs-27-pass/)–[3](/en/blog/sprt-llr/));
- **Verdict logic consistency**: online and offline run the same verdict engine and the same threshold derivation document — not two separate teams.

In one sentence: **offline gives you "same input, same output"; online gives you "different outputs, same verdict."** The former rests on determinism, the latter on statistics.

## 5. Closing the Loop: capture once, replay forever

The two modes are not two isolated islands — there is a conveyor belt between them:

```
capture live once ──► save as replay file ──► replay offline forever
      ▲                                          │
      └──── verify the fix online ◄── offline regression passes
```

A bug that shows up in the field gets captured once, and from then on it becomes a **regression asset** in the offline scenario library — on every release iteration, that bug is executed all over again. This is why the scenario library grows thicker with use: it isn't made up by engineers out of thin air; it is "contributed," piece by piece, by the real world.

## 6. Architectural Implication: Switching Modes Is Configuration, Not a Rewrite

The single most important sentence for an architect: **the same scenario YAML, the same verdict engine, the same evidence format — across both modes.** Switching modes changes two configuration items, the clock source and the data source — not the code. That means:

- Zero forking of scenario assets: scenarios developed offline go straight onto the live bus;
- Zero forking of verdict assets: the threshold derivation document works in both modes;
- Zero forking of evidence assets: one report format, so auditors only have to learn it once.

## 7. Where It Lands

The dual-mode clock is an implemented core mechanism in Cyclone Core (virtual clock + 1 ms deterministic scheduling: Supported; online mode adapting SocketCAN/UDP: Supported). Every scenario in the scenario pack is labeled with the mode it runs in — download it, start with offline determinism, then connect the same scenario to a real bus.

**The deepest architectural decision in a test platform is not choosing real or fake — it is deciding what must be deterministic (the verdict logic and the evidence chain) and what is allowed to be random (the physical world) — and then separating the two with a clock.**

---

## Appendix: Glossary (in order of appearance)

| Term | Plain explanation |
|---|---|
| **Dual-mode clock** | A design where the platform supports two time sources: a virtual clock and a real clock |
| **Virtual clock** | Time is injected by the platform at a fixed step (rather than passing physically); the world is fully controlled |
| **Offline mode** | Virtual clock + data replay: a closed world, bit-level reproducible |
| **Online mode** | Real clock + live bus: real timing and real hardware behavior |
| **Bit-level determinism** | Rerun with the same input: logs are byte-for-byte identical and the verdict is necessarily identical |
| **Content-addressed PASS** | If the input hash is the same, the verdict can be reused from cache; CI doesn't have to rerun |
| **Seed matrix** | The same scenario executed in bulk with multiple random seeds, feeding the statistical verdict |
| **Leak point** | A source of physical noise in the real world that breaks bit-level determinism (five categories in total) |
| **Bus arbitration** | The mechanism by which CAN resolves priority when multiple nodes transmit at once; the outcome drifts with instantaneous state |
| **ppm** | Parts per million; the common unit for crystal frequency deviation |
| **capture once, replay forever** | The asset loop: capture once online, replay offline forever |
| **SocketCAN** | The CAN bus networking framework on Linux |

*(Product capabilities are subject to the current version of Cyclone Core.)*
