---
title: "SPRT and LLR: Sequential Testing That Delivers the Verdict as Soon as the Evidence Is In"
description: "How many runs are enough? SPRT gives the verdict a scoreboard: stop as soon as the evidence is in, cutting execution volume by 30–50% on average."
series: "Statistical Verdicts Series · Part 3"
tn: "TN-03"
date: 2026-09-06
zh_url: "/blog/sprt-llr/"
cta_text: "Download the scenario pack and run it yourself"
cta_url: "/en/#scenariopack"
---


The first two parts solved "what to judge by": look at the floor score (the Wilson lower bound), not the surface pass rate. But one question stayed open: **how many runs count as enough?**

The first two parts answered: "run enough samples — say, 300." Anyone who has done regression testing knows how expensive that answer is: 300 runs per scenario, a hundred scenarios makes thirty thousand runs, and your CI queue stretches into tomorrow.

Worse is the sense of waste: **most of the time you don't need 300 runs at all** — inject a fault and the system crashes four times in a row, plainly broken; or it passes 12 straight, smooth as silk, plainly fine. Forcing the full 300 before stamping the verdict means punishing a question that already has an answer with queue time.

Is there a way to **watch as you run, and close the case early once the evidence is in**? Yes — that's SPRT (Sequential Probability Ratio Test). Its engine is called the LLR (log-likelihood ratio).

## 1. Start with a Picture: Two Worlds Face Off

The essence of verdicting a pass rate is making two "world hypotheses" confront each other on the spot:

- **H₀ (the healthy world)**: the SUT is fine; true pass rate = 0.90;
- **H₁ (the degraded world)**: the SUT is broken; true pass rate = 0.70.

Every run produces a result that looks more like the product of one world than the other. A string of failures? Looks like the degraded world. A flawless streak? Looks like the healthy world. What SPRT does is **put a scoreboard on this confrontation**: each run adds or subtracts points, and once enough points pile up, the verdict is declared on the spot.

## 2. LLR: How Many Points Each Run Is Worth

The **likelihood ratio** answers this question: "How many times more likely is this run's result in the H₁ world than in the H₀ world?" Take the logarithm (multiplication becomes addition — easier bookkeeping), and you get the LLR — **the "evidence points" of each run**.

Compute the scoring rules for the two worlds above (derivation skipped; conclusions below):

| This run's result | Scoreboard change | Why |
|---|---|---|
| Success | **−0.25** (a small push toward "healthy") | A healthy system is supposed to succeed; success is no news |
| Failure | **+1.10** (a big push toward "degraded") | Failures are rare in the healthy world; when one shows up, it's big news |

Note the asymmetry: **one failure carries roughly the information of four successes**. This matches every test engineer's intuition — when nothing is wrong, no amount of smooth running is more than "as expected"; the failing run is the information bomb.

## 3. SPRT: Draw Two Trip Lines on the Scoreboard

The rules are simple enough to fit on a sticky note:

```
scoreboard ≥ +2.94  →  verdict: H₁ holds, FAIL (genuinely degraded)
scoreboard ≤ −2.94  →  verdict: H₀ holds, PASS (it held up)
in between          →  not enough evidence, keep running
```

±2.94 isn't pulled from thin air — it's computed directly from the misjudgment rates you can tolerate (5% room for error on each side corresponds to log 19 ≈ 2.94; if you want it stricter, draw the lines farther out, at the cost of a few more runs).

Walk through two example trajectories to get a feel for it (computed with the scoring rules above):

**Scenario A: the system is clearly degraded after fault injection** — four runs in a row: fail, fail, pass, fail

```
+1.10 → +2.20 → +1.95 → +3.05  ≥ +2.94 ✅
```

**Case closed in 4 runs, verdict FAIL.** A fixed-sample-size plan would still be queuing at run 4/300.

**Scenario B: the system is healthy** — 12 consecutive successes

```
12 × (−0.25) = −3.01  ≤ −2.94 ✅
```

**Case closed in 12 runs, verdict PASS.**

What about the general case? On expectation, a healthy scenario gets its verdict in about 25 runs, a degraded one in about 19 — compared to a fixed-sample-size plan at the same misjudgment rates, **that's 30–50% saved on average, and over 90% on one-sided scenarios**. This is no back-alley trick: Wald proved in 1943 that at equal misjudgment rates, SPRT has the smallest expected sample size of all testing methods — **the savings are mathematically guaranteed savings**.

## 4. Why This and Fault Injection Are a Match Made in Heaven

A fault campaign has a distinctive statistical signature: **most scenarios are one-sided**.

- The camera drops frames for 200 ms and AEB recall falls from 0.99 to 0.87 — one-sided degradation, obvious within a few runs;
- Ultrasonic frame drops leave some function completely unfazed — one-sided health, cleared in a dozen runs.

The scenarios that genuinely need a few hundred careful runs are the contested ones, "where the degradation sits right at the threshold." A fixed-sample-size plan runs all three kinds to completion, **handing the obvious scenarios and the contested ones the same budget**; SPRT automatically shifts the budget toward the contested ones — the machine time saved goes to where the knife's edge is.

For CI quality gates this is hard cash: the gate's wall clock shrinks, feedback comes faster, and developers stop treating red lights as background noise.

## 5. How SPRT Relates to Wilson

No fight — different jobs:

| | In charge of what | When it shows up |
|---|---|---|
| **SPRT + LLR** | **When to stop** — deciding the closing time online, run by run | During execution |
| **Wilson lower bound** | **What number goes in the report** — giving the final pass rate an honest interval | After the verdict, written into the evidence pack |

So the Cyclone Core verdict engine uses both (Supported): SPRT handles "deliver the verdict early once the evidence is in," and Wilson turns the verdict into a floor-score number that third parties can recompute. One is the throttle, the other is the dashboard.

## 6. Three Engineering Truths (Don't Step on These Rakes)

1. **SPRT needs the two worlds set up front (p₀ and p₁).** 0.90 vs 0.70, or 0.90 vs 0.85? The closer the two worlds, the longer the verdict drags on. p₀/p₁ shouldn't be pulled from thin air — they should come from baseline measurements and margin analysis (covered in [Part 4](/en/blog/margin-analysis/) of this series) — **SPRT saves samples, but not thinking**.
2. **You'll occasionally hit a "tug-of-war"**: when the true pass rate lands right between the two worlds, the scoreboard swings back and forth for a long time. The engineering answer is **truncated SPRT** — set a sample-size cap, judge on the evidence at hand when the cap is hit, and honestly write "a close call" into the report.
3. **SPRT is no get-out-of-jail card**: it optimizes "how many runs"; it doesn't change "what to judge by" or "where the threshold comes from" — those two belong to [Part 2](/en/blog/wilson-lower-bound/) and [Part 4](/en/blog/margin-analysis/) respectively. Only together do the three form a complete verdict system.

## 7. Once It's in the Engine

In Cyclone Core you never touch a scoring rule: declare the two worlds (e.g. `p0: 0.90, p1: 0.70`) and the misjudgment budget (α/β) in the scenario YAML, and the verdict engine keeps the books run by run, judging the moment a trip line is touched — the verdict, along with the full LLR trajectory, lands in the evidence pack (Supported). **You write the scenarios; the engine handles the closing.**

Want to feel it firsthand? The scenario pack ships with ready-made fault-injection scenarios — run one and watch how many runs it takes the scoreboard to touch a line.

Next in the series: the three-step margin-analysis method — how p₀, p₁, and the threshold should actually be set, and where the dividing line falls between "pulled from thin air" and "derivable."

---

## Appendix: Glossary (in order of appearance)

| Term | Plain-English explanation |
|---|---|
| **SPRT (Sequential Probability Ratio Test)** | A method that judges while sampling: close the case the moment the evidence is in, keep running otherwise; proposed by Wald in 1943 |
| **LLR (log-likelihood ratio)** | The "evidence points" each run contributes: which world this run's result more likely came from; in log form it can be accumulated run by run |
| **Likelihood** | The probability of observing the data at hand under a given hypothesis |
| **H₀ / H₁ (the two worlds)** | The two hypotheses in confrontation: in this article H₀ = healthy (pass rate 0.90), H₁ = degraded (0.70) |
| **Scoreboard / trip line** | The accumulated value of the LLR; the two verdict lines above and below (±2.94 in this article) — touch one and the verdict drops |
| **Misjudgment rates α / β** | α: the probability of misjudging healthy as degraded; β: the probability of misjudging degraded as healthy. The trip-line distance is computed from them |
| **Expected sample size** | The average number of runs needed to reach a verdict; SPRT achieves the minimum among all methods at the same misjudgment rates |
| **Fault campaign** | A testing activity that executes fault scenarios in batches to see whether the system holds its bottom line |
| **Truncated SPRT** | A practical variant that adds a sample-size cap to sequential testing, preventing a "tug-of-war" from dragging on forever |
| **p₀ / p₁** | The pass-rate settings of the two worlds; should be derived from baseline measurements and margin analysis |
| **Margin analysis** | The star of the next article: a method for deriving thresholds and verdict parameters from the baseline |

*(The scoring rules and trajectories in this article are illustrative data computed with the standard LLR/SPRT formulas; product capabilities are subject to the current version of Cyclone Core.)*
