---
title: 'Determinism, Fault Injection, and the Evidence Chain: One "Exam Philosophy" Shared by Three Industries'
description: "Chips, AI agents, and automotive electronics: the SUT is too complex to exhaust and failure too costly to improvise — so all three grew the same methodology."
series: "Cross-Industry Notes · Part II"
tn: "TN-07"
date: 2026-09-10
zh_url: "/blog/cross-industry-verification/"
cta_text: "Download the scenario pack"
cta_url: "/en/#scenariopack"
---


Chips, AI agents, automotive electronics — three industries that seem to have nothing in common. But walk into their respective verification labs and you will see a strikingly similar scene: engineers everywhere are answering the same question, with the same methodology.

## 1. The Same Question

The three industries' "exam papers" look different; the question is the same:

- **Chips**: one tape-out costs tens of millions and takes half a year. Before the silicon comes back, how do you know a design with billions of transistors is correct?
- **AI agents**: run an LLM twice on the same input and you may get two different answers. How do you prove an autonomous agent is reliable?
- **Automotive electronics**: the controller sits inside a two-ton steel box moving at 120 km/h. Before it goes into a vehicle, how do you prove it still reacts correctly under extreme conditions?

The commonality is obvious: **the SUT is too complex to exhaust, and the cost of failure too high for trial and error.** Faced with this situation, each industry independently grew — look closely — almost the same answer.

## 2. Three Exam Rooms, Sketched

**The chip exam room**: the design exists as RTL (register-transfer-level code), first run in a software simulator; when that gets too slow, it moves to a hardware emulator (a cabinet full of dedicated chips that "burns in" the design to run at near-real speed). Stimulus comes from constrained-random generation, every test carries a random seed, and verification convergence is marked by coverage-closure sign-off.

**The AI agent exam room**: public benchmarks (SWE-Bench, AgentBench, τ-bench, etc.) provide the "question bank"; verdicts come in three schools — execution verdicts (actually run the code the agent wrote; it only scores if tests pass), trajectory verdicts (don't just look at the final answer; inspect the tool-call process step by step), and LLM-as-judge (let another model grade it, then find ways to correct its bias). Offline evaluation runs daily; after launch, A/B tests watch the real metrics.

**The automotive exam room**: scenario replay (feeding sensor data to the controller software) + fault injection (deliberately dropping frames, adding latency, tampering with fields) + statistical verdicts (not "run a few rounds and look at the pass rate" but computing confidence intervals) + an evidence chain (every verdict's inputs, process, and conclusion can be recomputed by a third party). Verification climbs the MiL→SiL→PiL→HiL ladder toward real hardware, rung by rung.

## 3. Four Shared Pillars

Strip away the industry jargon, and the three exam rooms stand on the same four pillars.

### Pillar 1: Determinism — a bug must be reproducible, or it was never fixed

- Chips: **seed replay** in constrained-random testing — same seed, identical stimulus sequence, failures replayed precisely;
- AI: **record and replay** every model response keyed by input hash — a stochastic model placed inside a deterministic exam room;
- Automotive: **virtual clock + deterministic scheduling** — rerun the same scenario N times and the verdict and critical timing are bit-identical.

All three industries understand: without reproducibility, a "fix" is just a wish.

### Pillar 2: Injection and Adversarial Testing — fair-weather tests don't count

- Chips: functional-safety standards demand **fault-injection campaigns** — inject stuck-at faults and transient soft errors (SEU) into the design, verify the detection rate of safety mechanisms, directly feeding FMEDA quantification;
- AI: **adversarial cases** and chaos engineering — tool timeouts, API errors, malicious input injection, deliberately hitting the system where it hurts;
- Automotive: **bus fault injection** — frame drops, latency, field tampering, verifying that the function holds its safety floor under degraded perception.

The goal is the same: not to prove the system "works," but to map "**where it starts to stop working**."

### Pillar 3: Statistical Verdicts — "27 passes out of 30 runs" is not a pass

All three industries have been burned by point estimates, and all converged on the same conclusion: **a verdict must be based on the confidence interval, not the surface pass rate.**

27 successes in 30 trials gives a surface success rate of 0.90, but the lower bound of the 95% confidence interval is only 0.74 — that sample size has no business claiming "success rate ≥ 0.90." To get the lower bound above 0.99, you need evidence on the order of 300 consecutive passes. When samples are scarce, sequential testing (SPRT) lets the system keep asking "is the evidence enough?" as it runs — stopping early when it is, running on when it isn't.

### Pillar 4: Evidence and Attribution — a report is not a conclusion, but a recomputable chain

- Chips: coverage data, failure waveforms, sign-off records;
- AI: full trajectory recordings, version comparisons, failure attribution;
- Automotive: scenario hash + log digest + failure snapshot — a third party can take the evidence package and **recompute the same verdict**.

Facing an auditor (a tape-out sign-off review, a platform review board, a regulator), "we tested it" is worthless; "you can recompute it yourself" is priceless.

## 4. The Same Ladder: Trading Fidelity for Speed

The three industries even share an organizational structure — the **verification ladder**:

| Ladder principle | Chips | AI agents | Automotive |
|---|---|---|---|
| Fast and abstract | Functional simulation (ISS) | Offline recorded replay | MiL / SiL |
| Middle rung | Cycle-approximate simulation | Shadow mode / canary | PiL |
| Slow and real | FPGA prototype → real silicon | Online A/B | HiL rig |

The pattern is identical: the further down you go, the more real, the slower, the more expensive; **fast rungs run daily as the quality gate, slow rungs get periodic sign-off**. Put the gate on the wrong rung and you either go broke or let bugs through.

## 5. What You Can't Copy Over

Isomorphic does not mean identical. The biggest difference across the three industries is the **determinism of the SUT**:

| Industry | SUT determinism | Verdict weighting |
|---|---|---|
| Chip RTL | Fully deterministic (same input, same output) | Bit-level comparison dominates |
| Automotive software | Nearly deterministic (scheduling and floating point must be controlled) | Bit-level + statistical combined |
| LLM agent | Stochastic by nature | Statistical verdicts only |

The fault models differ too: chips get physical faults, automotive gets bus faults, AI gets semantic faults. **The methodology transfers; the tooling must be localized** — transplant someone else's exam room unchanged and you will fail on the interfaces and the fault model.

## 6. Landing This Methodology on the Automotive Software Layer

Cyclone Core is our automotive answer to this cross-industry map:

- Virtual clock + 1 ms deterministic scheduling; rerunning the same scenario is bit-identical (Supported);
- Bus fault injection: frame drops / latency / field tampering (Supported); CRC corruption and node freeze (Planned);
- A verdict engine combining Wilson confidence intervals with SPRT sequential verdicts, thresholds calibrated through margin analysis (Supported);
- Evidence chain: scenario hash + log digest + failure snapshot + JUnit output; third parties can recompute the verdict (Supported).

An exam philosophy validated independently over decades in three industries deserves to be implemented once more, carefully, at the automotive software layer.

*(Automotive-side capability descriptions reflect the current version of Cyclone Core; "Planned" items are not delivered capabilities. Example figures are demo data.)*
