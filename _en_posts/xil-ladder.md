---
title: "The XiL Ladder, Side by Side: Automotive and Silicon Are Drawing the Same Picture"
description: "Automotive says SiL/HiL; silicon says ISS/cycle-approx — on paper, they are the same ladder from fast-and-fake to slow-and-real."
series: "Cross-Industry Notes · Part I"
tn: "TN-06"
date: 2026-09-09
zh_url: "/blog/xil-ladder/"
cta_text: "View all articles"
cta_url: "/en/#blog"
---


Put an automotive test engineer and a chip verification engineer at the same table, and they will likely fail to understand each other: one says SiL and HiL, the other says ISS and cycle-approx. But draw both worlds on paper, and you will find **it is the same picture** — a ladder from "fast and fake" to "slow and real."

The ladder has a unified name: **XiL (X-in-the-Loop)**.

## 1. First Principles of the Ladder: Trading Fidelity for Speed

Every XiL ladder obeys the same physics:

> **The more real, the slower and more expensive; the more abstract, the faster and cheaper.**

So the standard play for an engineering organization is: **use the fast levels as the quality gate, run them daily; use the slow levels for sign-off, run them periodically.** Put the gate on the wrong level and there are only two ways to die — gate on a level that is too slow and you go broke; gate on a level that is too abstract and you ship leaks.

## 2. The Automotive Ladder: MiL → SiL → PiL → HiL

| Level | Full name | What runs | Characteristics |
|---|---|---|---|
| **MiL** | Model-in-the-Loop | The control algorithm model (e.g. a Simulink diagram) | Earliest, fastest; checks whether the algorithm logic is right |
| **SiL** | Software-in-the-Loop | Generated C code running on a PC | Checks that "the code implements the model"; pure software, massively parallel |
| **PiL** | Processor-in-the-Loop | Code running on the target processor/eval board | Exposes compiler, word-length, and timing issues |
| **HiL** | Hardware-in-the-Loop | Real ECU wired to a simulation rig | Closest to the vehicle; rigs are expensive and scarce |

## 3. The Silicon Ladder: Model/Operator → Compile → ISS → cycle-approx → FPGA → Real Silicon

| Level | What runs | Characteristics | Automotive counterpart |
|---|---|---|---|
| Model/operator | ONNX model, individual operators | Algorithm correctness | ≈ MiL |
| Compile | The instruction sequence the compiler emits | Verifies the compiler didn't "mistranslate" the model | ≈ code-generation checks |
| **ISS** (functional simulation) | Instruction-set simulator: behaviorally correct, **untimed** | Fast; the workhorse of functional verification | **≈ SiL** |
| **cycle-approx** (performance simulation) | Simulator with a timing model | Measures performance/bandwidth/latency; slower than ISS | ≈ PiL/timing analysis |
| **FPGA prototype** | The design burned into an FPGA cabinet | Near-real speed; drivers can start early | Prototype controller |
| **Real silicon / hardware-in-the-loop** | Actual silicon back from the fab | Final sign-off | **≈ HiL** |

## 4. One Table, Two Industries, the Same Rules

```
fast/cheap/abstract ────────────────────► slow/expensive/real

Auto:    MiL ────── SiL ────── PiL ────── HiL
Silicon: model/op ── ISS ── cycle-approx ─ FPGA ── real silicon
                  ▲                         ▲
                  │                         │
            where the gate lives      where sign-off lives
```

The shared rules, point by point:

1. **Each level is more real than the one before it — and slower by one to several orders of magnitude**;
2. **Every level has its own "golden reference"**: the automotive scenario baseline ↔ the chip's golden/baseline outputs;
3. **Every level needs a regression gate**: scenario regression in automotive ↔ daily CI regression in silicon;
4. **The further right, the scarcer the resource**: queuing for HiL rigs ↔ scheduling FPGA/real-silicon boards.

## 5. A Two-Way Translation Glossary

**Chip terms for the automotive engineer:**
- ISS: instruction-set simulator — think of it as a "pure-software virtual ECU": behaviorally exact, time is optional;
- cycle-approx: an ISS plus a timing model — roughly a PiL that has started caring about deadlines;
- tape-in (silicon back from the fab): the chip returns from the foundry — the equivalent of your first B-sample delivery;
- golden: the reference answer for every test — your baseline data.

**Automotive terms for the chip engineer:**
- SiL: running the entire controller software on a PC — what your ISS level does;
- HiL rig: real ECU + simulated environment — your chip validation board + tester, except the "environment" is virtual vehicles and roads instead of a waveform generator;
- scenario: a replayable stream of sensor/bus data — your testbench stimulus.

## 6. Where Cyclone Core Sits on the Ladder

Cyclone Core positions itself as the **"deterministic verdict layer" on the ladder**:

- At the **SiL level**: offline mode (virtual clock + replay) provides bit-exact reproducible scenario regression and fault injection (Supported);
- At the **HiL level**: online mode connects to real buses, with statistical verdicts + evidence chain (Supported);
- **Across all levels**: the same scenario assets, the same verdict engine, the same evidence format — eliminating the most common accident between ladder levels ("it passed on this level but not that one, because the two sides weren't running the same thing").

The ladder is a product of industry division of labor, but **verdicts and evidence should not be stratified** — that is the lesson this side-by-side picture leaves for tool designers.

---

## Appendix: Glossary (in order of appearance)

| Term | Plain explanation |
|---|---|
| **XiL (X-in-the-Loop)** | Umbrella term for "X-in-the-loop" testing: putting something (model/software/processor/hardware) inside a simulated closed loop |
| **MiL / SiL / PiL / HiL** | The automotive ladder: Model- → Software- → Processor- → Hardware-in-the-Loop |
| **ISS (instruction-set simulator)** | A chip simulator that guarantees instruction behavior only, untimed; fast |
| **cycle-approx (cycle-approximate simulation)** | A chip simulator with a timing model; can measure performance, slower |
| **FPGA prototype** | The chip design burned into programmable hardware — a "near-real" chip |
| **Tape-in** | The chip returns from the foundry after tape-out |
| **golden / baseline** | The golden reference output / baseline data for each test |
| **testbench** | The chip verification platform (stimulus + checking environment) |
| **quality gate / sign-off** | Gate: high-frequency automated checking (cheap levels); sign-off: low-frequency formal acceptance (real levels) |
| **B-sample** | The second round of engineering samples in automotive development |
| **evidence chain** | The full set of records that lets a third party recompute the verdict |

*(Product capabilities are subject to the current version of Cyclone Core.)*
