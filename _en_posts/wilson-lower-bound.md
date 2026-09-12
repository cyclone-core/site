---
title: "Wilson Lower-Bound Verdicts: An Honest Ruler for Pass Rates"
description: "Wald, Clopper-Pearson, Wilson: three rulers, three floor scores. Why acceptance promises must stand on the lower bound, with a cheat table and 5-line code."
series: "Statistical Verdicts Series · Part 2"
tn: "TN-02"
date: 2026-09-05
zh_url: "/blog/wilson-lower-bound/"
cta_text: "Learn about the Cyclone verdict engine"
cta_url: "/en/#pillars"
---


[The last post](/en/blog/30-runs-27-pass/) left a loose thread: we said a verdict should look at the "floor score" (the lower bound of the confidence interval), not the surface pass rate. The obvious follow-up: how is the floor score actually computed? There are several ways to put an interval on a pass rate, and **the rulers themselves are not equally good**. This post settles the question of rulers.

## 1. Three rulers, three "floor scores"

Given the same evidence (27 passes in 30 runs), three common methods give three different lower bounds:

| Method | Interval for 27/30 | Character |
|---|---|---|
| **Wald (normal approximation)** | ≈ [0.793, 1.000] | The most optimistic, and the easiest to be fooled by |
| **Clopper-Pearson (exact method)** | ≈ [0.735, 0.979] | The most conservative; the scale is deliberately stretched |
| **Wilson (score method)** | [0.744, 0.965] | In between, and stable even on small samples |

Wald gives a floor score of 0.793; Wilson gives 0.744 — **different rulers, different honesty in the conclusion**. Wald is the most common method in textbooks, but it is a product of the big-sample era and starts distorting as soon as samples get scarce.

The most telling case is a perfect record — 30 runs, 30 passes:

- **Wald computes [1.00, 1.00]**: zero width, meaning "the true pass rate is 100%, dead certain." Daring to claim that after 30 runs — this ruler is plainly lying;
- Wilson computes [0.887, 1.000]: 30 out of 30, floor score 0.887 — **honest, and it matches intuition**: 30 runs is still a long way from "a rock-solid 99%."

## 2. Why verdicts only watch the lower bound

An interval has two endpoints, so why does acceptance only stare at the left one? Because **an acceptance promise is one-directional**.

"Pass rate ≥ 0.90" is a one-way promise: you only care whether the true level might fall below 0.90, not how high it could go. Each endpoint has its own audience:

- the **upper bound** is for the optimists: how good things could ideally get;
- the **center** is for the press-release writers: the prettiest-looking number;
- the **lower bound** is for the people signing off: the line that won't be breached even in the most conservative case — **a promise can only stand on the lower bound**.

Sign-off uses the lower bound. That is the entire case for lower-bound verdicts.

## 3. Wilson's intuition: when evidence runs thin, pull toward fifty-fifty

You don't need to memorize the Wilson formula; remember its temper: **the smaller the sample, the harder it pulls your score toward 50%**.

The reasoning is plain: two runs, two passes — is the true level 100%? Wilson says: too little evidence, so for now treat it as an ordinary coin (50%), and believe you once the sample grows. The larger the sample size, the weaker the pull, and the closer the estimate hugs the measured value. That is why it doesn't distort near small samples or perfect scores — precisely the two places where Wald falls on its face.

(For the formula and a five-line Python implementation, see Appendix A at the end.)

## 4. A bit of gossip: Reddit uses this ruler too

In 2009, engineer Evan Miller wrote the famous article "How Not to Sort by Average Rating," pointing out a widespread stupidity: sorting by average rating ranks a product with "one 5-star review" above a product with "100 reviews averaging 4.8" — the former's sample is too small to deserve it.

The fix is exactly the Wilson lower bound: **don't rank by average; rank by floor score**. Reddit's "best" comment sorting uses it.

Notice that this is **the same math problem** as test verdicts: a case with 1 run and 1 pass (100% pass rate) versus a case with 100 runs and 96 passes (96%) — whose evidence is harder? On surface numbers the former wins; on floor score the latter wins. You already guard against "pumped-up five-star ratings" when you shop; don't forget to use the same ruler when you measure pass rates.

## 5. A cheat table: N runs, all pass — what is the floor score?

Perfect runs are the case you meet most often, so look it up directly (Wilson, 95%):

| All-pass runs | Floor score | Claim it can support |
|---|---|---|
| 10 | 0.723 | failure rate < 28% (basically unconvincing) |
| 30 | 0.887 | failure rate < 11% |
| 50 | 0.929 | failure rate < 7% |
| 100 | 0.963 | failure rate < 3.7% |
| 300 | 0.987 | **threshold zone for reliability ≥ 0.99** |
| 1000 | 0.996 | 0.996 |

Bookmark this table. The next time someone uses "ran it 30 times, all passed" to prove 0.99 reliability, you will see at a glance that they are off by an order of magnitude.

## 6. Three common objections, answered up front

**"Isn't this too strict?"** Not strict — honest. The floor score is not a punishment; it rises naturally with sample size — bank enough evidence and the line clears itself. People who call it strict usually mean "I don't want to run more samples."

**"Why 95% and not 99%?"** The higher the confidence level, the wider the ruler's marks, and the lower the floor score. 95% is the industry's default balance point; for safety sign-off you can go to 99%, but budget for "needs more samples."

**"Can we loosen up a bit with a one-sided interval?"** Yes, and it is defensible: acceptance is one-directional, so a one-sided 95% lower bound (z = 1.645 instead of 1.96) is slightly looser for a legitimate reason. But get fluent with the two-sided version first, then talk about that optimization.

## 7. This ruler is already built into the engine

The default verdict method of the Cyclone Core verdict engine is the **Wilson lower-bound verdict** (Supported): you write `{metric: task_success, op: gte, value: 0.90}` in the scenario YAML, and the engine checks "95% lower bound ≥ 0.90" instead of the surface pass rate — **engineers write the threshold, statistics owns the ruler**, and nobody gets to pass off luck as evidence.

Next post: a verdict can be not just "accurate" but "fast" — how [SPRT sequential testing](/en/blog/sprt-llr/) ends the exam early as soon as the evidence is in, saving 30–50% of execution on average.

---

## Appendix A: The formula and a five-line implementation (for the hands-on)

Wilson interval (z depends on the confidence level; z = 1.96 for 95%):

```
        p̂ + z²/(2n)  ±  z·√( p̂(1−p̂)/n + z²/(4n²) )
CI  = ───────────────────────────────────────────────
                      1 + z²/n
```

where p̂ = passes ÷ total runs, and n = total runs. A verdict only takes the minus half (the lower bound).

```python
def wilson_lower(k, n, z=1.96):
    """k passes out of n trials → 95% confidence lower bound (floor score)"""
    p = k / n
    denom = 1 + z * z / n
    center = (p + z * z / (2 * n)) / denom
    half = z * ((p * (1 - p) / n + z * z / (4 * n * n)) ** 0.5) / denom
    return center - half

print(round(wilson_lower(27, 30), 3))   # 0.744
print(round(wilson_lower(300, 300), 3)) # 0.987
```

## Glossary (in order of appearance)

| Term | Plain explanation |
|---|---|
| **Floor score (lower bound)** | The line the true pass rate will not breach even in the most conservative case; verdicts should look at it |
| **Confidence interval** | A range computed from samples; the true value most likely lives inside it |
| **Wald (normal approximation)** | The classic interval method; accurate only on large samples, and lies on small samples / perfect scores (e.g. 30 out of 30 yields [1.00, 1.00]) |
| **Clopper-Pearson (exact method)** | An "exact" method guaranteed not to understate risk, at the cost of being conservative and wider |
| **Wilson (score method)** | The method this post recommends: stable on small samples, no distortion near perfect scores; the intuition is "thin evidence pulls toward fifty-fifty" |
| **One-sided / two-sided interval** | Two-sided gives both an upper and a lower bound; when you only care about "not below X," a one-sided lower bound works and is slightly looser |
| **z value** | The conversion factor for the confidence level: 1.96 for 95%, 2.58 for 99%, 1.645 for one-sided 95% |
| **Shrinkage** | The practice of pulling the estimate toward a neutral value (50%) when samples are few — guards against small-sample bragging |
| **Cheat table** | The lookup table of floor scores for N all-pass runs; handy during reviews |
| **Lower-bound verdict** | A verdict style where the acceptance line applies to the floor score rather than the surface pass rate |
| **SPRT (sequential testing)** | The star of the next post: compute evidence as you run, and hand down the verdict early once there is enough |

*(Interval values in this article are the statistical results of the respective algorithms; the Clopper-Pearson value is approximate. Product capabilities are subject to the current version of Cyclone Core.)*
