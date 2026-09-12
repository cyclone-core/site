---
title: "Margin Analysis in Three Steps: Thresholds Are Derived, Not Picked"
description: "Why is the acceptance line at 0.98? Baseline, margin, two-way validation — turn thresholds from decoration into evidence."
series: "Statistical Verdicts Series · Part 4"
tn: "TN-04"
date: 2026-09-07
zh_url: "/blog/margin-analysis/"
cta_text: "Book a POC consultation"
cta_url: "/en/#contact"
---


The question that most reliably silences a review meeting: "Why is this acceptance line set at 0.98, and not 0.95 or 0.99?"

If the answer is "experience," "industry practice," or "close enough" — then the line is decoration. Set it too loose and it won't catch regression; set it too tight and it wrongly kills healthy builds. **A threshold should not be "picked." It should be derived.** The derivation method is called margin analysis, and it has three steps.

## 1. Set the Scene: The Blade Sits Between Two Worlds

A test verdict faces two worlds:

- **The healthy world**: no fault injected; the metric fluctuates around its baseline (with noise);
- **The degraded world**: a fault is injected; the metric drops (also with noise).

The threshold τ is a blade placed in the gap between the two worlds. Its distance to each side has a name:

```
Metric value (e.g., near-range recall)  low ──────────────────────────► high

      0.87            0.98             0.9931
───────●───────────────│─────────────────●────────
   degraded world    τ threshold      healthy world
              │◄detection►│◄──health──►│
               margin        margin
```

- **Health margin = baseline − τ**: how far a healthy build is from being wrongly killed;
- **Detection margin = τ − degraded measurement**: how far a degraded build is from slipping through.

**Both margins must be significantly larger than the noise of their respective worlds** — otherwise the line means nothing.

## 2. The Three Steps (Demonstrated with a Real Tuning Session on a Blind-Spot Monitoring Scenario)

**Step 1: Run the baseline.** No fault injection; run several rounds to get the metric baseline — note that it is a **distribution**, not a number. Measured: near-range bin recall ≈ **0.9931**.

**Step 2: Set the margin.** Threshold = baseline − acceptable degradation. The engineering argument is "how much may blind-spot monitoring degrade under brief radar frame loss, per the business?" Say we tighten by 1.3 percentage points: **τ = 0.98**.

**Step 3: Two-way validation.**
- In the fault scenario (radar frame loss), the measured metric drops below 0.98 → what should FAIL did FAIL ✓
- Re-run the baseline; it stays above 0.98 → what shouldn't FAIL wasn't wrongly killed ✓

**Two-way validation is the dividing line from picking numbers out of thin air**: verifying only that "the fault will FAIL" is not enough — you must also verify that "the baseline won't falsely FAIL."

## 3. Two Ways to Fail

| Threshold mistake | Consequence | Common symptom |
|---|---|---|
| **Too loose** (e.g., τ = 0.80; a drop to 0.87 still PASSes) | Fault injection tests for nothing; all-green gives you false confidence | Testing degenerates into compliance theater |
| **Too tight** (e.g., τ = 0.995; even the 0.9931 baseline fails) | CI is red every day; the team learns to ignore red lights | Alarm fatigue; the test system dies |

The rule of thumb for the sweet spot: **make each margin divided by its world's noise large enough** — the distance from baseline to τ ≫ run-to-run baseline jitter; the distance from the degraded measurement to τ ≫ degraded-world jitter.

## 4. Statistical Backup: When There Is No Line to Draw at All

The baseline comes from sampling, so it carries uncertainty of its own (bracketed by a Wilson interval — see [Part 2 of this series](/en/blog/wilson-lower-bound/)). This yields the most important **feasibility criterion** in margin analysis:

> A threshold that separates the two worlds exists only if **the lower bound of the healthy world's interval > the upper bound of the degraded world's interval**.
> If the two intervals overlap → no threshold works → three ways out: strengthen the fault, add samples to tighten the intervals, or honestly admit "this fault is undetectable on this metric."

This is the same idea as MSA (Measurement System Analysis) in manufacturing: first ask "can my ruler tell good parts from bad," then talk about inspection. **The verdict engine owns the ruler (Wilson/SPRT); margin analysis owns the scale markings.**

## 5. The Hidden Fourth Dimension: Window Phase

While tuning the blind-spot scenario, we hit a pitfall subtler than any number: the fault window must land in **the phase where the metric is actually decided**. In the synthetic data, the target approaches over time; a frame-loss window early in the run never touches the near-range bin. Only after we moved the window to when the target enters the near-range zone did the fault actually "bite" the metric.

The implication: **margin analysis is not just picking a number — it also means confirming that "fault window × metric binning × scenario phase" are aligned**. Otherwise you will misread "the test didn't see it" as "the system is robust" — the most dangerous false negative of all: not a test failure, but test blindness.

## 6. Advanced Form: The Dose-Response Curve

A single fault intensity gives you a single degradation point. **Sweep the fault intensity** (frame loss of 100 ms / 200 ms / 500 ms ...) → the metric degrades with intensity → you get the SUT's **robustness profile curve**. A threshold is just choosing one point on that curve — and the curve itself is premium material for the safety case:

- "We characterized the system's degradation curve under fault class X, and placed the threshold before the knee" — far more convincing than "we picked 0.98";
- This is the same craft as a calibration engineer finding the knock boundary or the lambda window: **finding the maximum-margin operating point on the plant**. Margin analysis ports that thinking to the test system — and a test system is also a measuring instrument that needs calibration.

## 7. The Threshold Derivation Note: Write the Answer into the Evidence Chain

That silence-inducing review question now gets answered like this:

> "Baseline 0.9931 (n frames, Wilson lower bound 0.99); tightened by 1.3 percentage points per the business-acceptable degradation, so τ = 0.98; fault scenario measured 0.87, a detection margin of 11 percentage points; derivation attached."

This "threshold derivation note" is part of the evidence chain, archived alongside the verdict report (Cyclone Core verdict engine and evidence chain: Supported). **A picked threshold is decoration; a derived threshold is evidence.**

Threshold calibration is also a standard line item in our POC: scenario templates included, with baseline measurement, margin justification, and fault validation run against your SUT — come talk to us.

---

## Appendix: Glossary (in Order of Appearance)

| Term | Plain-language explanation |
|---|---|
| **Margin** | The distance from the threshold to the healthy/degraded world; split into health margin and detection margin |
| **Threshold τ** | The PASS/FAIL dividing line; this article argues it must be derivable |
| **Baseline** | The metric's reference level with no fault injected; a distribution, not a number |
| **Acceptable degradation** | The performance drop the business will tolerate; threshold = baseline − this |
| **Two-way validation** | The calibration step that verifies both "faults must FAIL" and "the baseline won't falsely FAIL" |
| **Feasibility criterion** | A valid threshold exists only if the healthy interval's lower bound > the degraded interval's upper bound; otherwise the fault is undetectable at that intensity |
| **MSA (Measurement System Analysis)** | Manufacturing method: first verify the measurement system itself can tell good from bad, then talk about inspection |
| **Window phase** | The alignment between the fault-injection time window and the metric's sensitive stage; misalignment causes "test blindness" |
| **Dose-response curve** | The degradation curve obtained by sweeping fault intensity — i.e., the SUT's robustness profile |
| **Threshold derivation note** | The document recording "baseline + acceptable degradation + validation results," archived with the evidence chain |
| **Alarm fatigue** | Red lights so frequent that the team habitually ignores them; the test system exists in name only |
| **SUT** | System Under Test |

*(The numbers in this article come from demonstration scenarios and illustrate the method only; product capabilities are governed by the current Cyclone Core release.)*
