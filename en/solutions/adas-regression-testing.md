---
layout: solution
lang: en
docno: CY-SOL-002
title: "ADAS Scenario Regression Testing: One Pipeline from Replay to the CI Quality Gate"
description: "Cyclone runs ADAS regression testing in one deterministic pipeline: one YAML per case, statistical verdicts, JUnit reports native to Jenkins/GitLab CI."
permalink: /en/solutions/adas-regression-testing/
zh_url: /solutions/adas-regression-testing/
---

<h2><span class="sec-no">§1 · PROBLEM</span>The Typical State of ADAS Regression Testing</h2>

The scenario library lives on the simulation engineer's machine, the regression scripts on the test engineer's machine, the reports in Word, and the conclusions in meeting minutes. Every pre-release regression is, at its core, a **manual pipeline**: run a batch of scenarios, eyeball the logs, pick a few screenshots, write up a "pass." Three direct consequences:

1. **Regression cycles are measured in weeks**, and the release cadence is held hostage by testing;
2. **Nobody can state the strength of the conclusion** — "50 runs, all passed" only supports a "failure rate < 6%" claim (the rule of three), yet all the review board hears is "no problems";
3. **A FAIL cannot be characterized** — product defect or test flakiness? Without a deterministic kernel, that question never gets an answer.

<h2><span class="sec-no">§2 · SOLUTION</span>One Pipeline: Replay → Verdict → Report</h2>

Cyclone folds ADAS scenario regression into a single deterministic pipeline:

`case YAML → data-source replay → fault injection → SUT → statistical verdict → evidence report`

<h3>One YAML Is One Regression Case <span class="st"><i class="sdot g"></i>Supported</span></h3>

The scenario, the faults, and the verdict thresholds are all declared in a single YAML. Change a parameter and you have a new scenario — the public catalog already ships **22 ADAS scenarios whose measured verdicts match expectations** (active safety ×13, driving assistance ×5, parking ×1, shared driving ×2, L3 admission ×1), covering typical functions such as AEB / LKA / ACC.

<h3>Dual-Mode Clock: Same Case, Two Physics <span class="st"><i class="sdot g"></i>Supported</span></h3>

Offline mode uses a virtual clock: accelerated replay on an ordinary server, byte-for-byte identical across reruns with the same seed. Online mode uses a real clock: connect to the live bus over SocketCAN/UDP with hardware-timestamped logging. The scenario YAML stays untouched — flip the clock mode and you have switched. See [Dual-Mode Clock](/en/blog/dual-clock/) for details.

<h3>CI-Native, Not "Integration-Ready"</h3>

- **JUnit XML out of the box** <span class="st"><i class="sdot g"></i>Supported</span> — Jenkins / GitLab CI consume it directly, zero rework on your regression dashboard;
- **rules_cyclone** <span class="st"><i class="sdot a"></i>Prototype</span>: `bazel test //...` puts scenario regression and unit tests in the same graph, the same cache, and the same dashboard;
- Single-file binary, download and run <span class="st"><i class="sdot o"></i>Planned</span>.

<h2><span class="sec-no">§3 · CASE</span>A Real Verdict Flip</h2>

Regression of an AEB function: after injecting clustered frame-drop faults into the front-view camera, the statistical verdict flips automatically —

| Run Scenario | Near-Range Recall | Mid-Range Recall | Far-Range Recall | False-Alarm Rate | System Verdict |
|---|---|---|---|---|---|
| Baseline (no fault) | 0.9924 | 0.7040 | 0.3453 | 0.0000 | PASS |
| Frame-drop fault injected | 0.8724 | — | 0.0000 | 0.1033 | **FAIL (verdict flipped)** |

<blockquote><p>Data from the demo environment, illustrating the full closed loop of "fault injection → metric degradation → verdict flip → evidence output"; a formal POC follows the same methodology on your real bus and ECU environment.</p></blockquote>

<h2><span class="sec-no">§4 · HONEST BOUNDARIES</span>Honest Boundaries</h2>

- **Cyclone is not a simulator**. CarMaker / Carla own "how real the world is"; Cyclone owns "how trustworthy the test is" — the two are complementary.
- **It does not replace your existing toolchain**. It layers on top of CANoe / dSPACE / in-house scripts, and the JUnit output feeds straight into your existing CI.
- Reliability claims must go through statistics: [only 300/300 supports a ≥ 99% reliability claim at 95% confidence](/en/blog/30-runs-27-pass/), and quality-gate thresholds must be calibrated against the SUT baseline ([margin analysis](/en/blog/margin-analysis/), a standard POC service item).
