---
title: "30 Runs, 27 Passes: Why That Doesn't Count as Verified"
description: "A 90% pass rate meets the 90% acceptance line, so why can't you sign off? Coin flips, a spoonful of soup, and one sample-size table explain point estimates, confidence intervals, and floor scores."
series: "Statistical Verdicts Series · Part 1"
tn: "TN-01"
date: 2026-09-04
zh_url: "/blog/30-runs-27-pass/"
cta_text: "Explore the Cyclone verdict engine"
cta_url: "/en/#pillars"
---


You've seen this sentence before. You may have written it yourself:

> "AEB scenario regression testing: 30 runs executed, 27 passed, pass rate 90%, meets the 90% acceptance threshold. Test passed."

Confident tone, standard format, stamped and archived. Unfortunately, **this sentence doesn't hold up to scrutiny** — and it isn't even close. This article makes the case with three pieces of everyday common sense, no formulas.

## 1. The coin-flip lesson: what you see ≠ what is true

Flip a coin 10 times, get 6 heads — can you conclude the coin is biased? You wouldn't dare — 10 flips is too few; luck plays too big a role.

Test pass rates follow exactly the same logic. 27 passes out of 30 runs gives you 0.90, but that number is only **the luck of this particular sample**. The software's true pass rate objectively exists (it's determined by code quality), but you can't test every possible situation — you can only sample a handful of runs. Take another 30 runs and you might get 25 passes, or 29.

Statistics has a name for this "number you see": the **point estimate** — an estimate of the true value based on a small handful of samples. Its flaw: **it reports the answer, but not how trustworthy the answer is**.

## 2. The spoonful-of-soup lesson: the true value lives in a range

To find out whether a pot of soup is salty enough, you taste a spoonful. The more spoonfuls you taste, the more confident you feel — but no matter how many you taste, your conclusion should be "**approximately** this salty," never "exactly this salty."

Pass rates are the same. The honest statement is not "the pass rate is 0.90," but "**the true pass rate is roughly within some range**." That range is called a **confidence interval**, and the trustworthiness of the range is called the **confidence level** (say, 95%).

Run the 27/30 evidence through the standard method (Wilson):

> 95% confidence interval for the true pass rate = **[0.744, 0.965]**

Stare at the lower bound: **0.744**. In other words, based on these 30 runs alone, the true pass rate could easily be just over 74% — a dozen-plus points below the 90% acceptance line. Imagine a student whose average score is 90 but whose **floor score is only 74**. Would you sign a document guaranteeing "never below 90"?

And what "95% confidence level" actually means: not "the true value has a 95% probability of being in the interval," but — **this interval-computing procedure, used 100 times, will capture the true value about 95 times**. The true value is a motionless fish; the interval is a net cast each time; 95% describes the hit rate of the net-casting technique.

## 3. A table worth keeping: how many runs do you actually need

Keep adding spoonfuls and watch how the conclusion changes (all at 95% confidence level):

| Observed result | Apparent pass rate | Floor score (interval lower bound) | Dare claim "≥ 0.90"? |
|---|---|---|---|
| 27 of 30 runs pass | 0.90 | **0.744** | No |
| 90 of 100 runs pass | 0.90 | **0.844** | Still no |
| 285 of 300 runs pass | 0.95 | 0.926 | Barely |
| 300 of 300 runs pass | 1.00 | 0.987 | Yes — and close to 0.99 |

Three counterintuitive lessons:

1. **30 runs isn't enough. Neither is 100.** When the apparent pass rate hugs the acceptance line, the required sample size is ten times what intuition suggests;
2. **Perfect-score claims are the most expensive.** The industry has a rule of thumb called the "rule of three": N consecutive runs with zero failures only guarantees a failure rate below 3/N. Want to prove a failure rate under 1%? **You need 300 consecutive runs without a single failure.** Want to prove 0.999? Roughly 3,000 failure-free runs;
3. So "we ran 50 times, all passed, no problem" — that statement actually only supports the conclusion "failure rate < 6%," but what the review board hears is "no problem."

## 4. Three common traps in test reports

**Trap 1: Treating the average as the floor score.** Apparent pass rate ≥ acceptance line means release. But as the table shows, 0.90 over 30 runs and 0.90 over 300 runs carry wildly different evidentiary weight — yet look identical on the report.

**Trap 2: Retaking the exam until you pass.** A test case FAILs, someone hits rerun, and once it passes, everything is fine. The cost is diluted results: a flaky case with a true pass rate of only 50% has a 75% chance of "eventually passing" after two retakes. Worse, **flakiness itself is an important signal** (randomness in the environment, race conditions in the code, wrong timing assumptions) — and the rerun button buries that signal.

**Trap 3: A passing line set by gut feel.** Why is the acceptance line at 90% and not 85%? If nobody can answer, that line is decoration: set it too loose and it can't stop the bad; set it too tight and it wrongly kills the good. The proper approach is to first measure the baseline performance of a known-good version, then derive the line as "baseline minus acceptable degradation" (this method is called margin analysis — [Part 4 of this series](/en/blog/margin-analysis/) covers it in detail).

## 5. The correct posture is just three moves

**Move 1: Look at the floor score, not the average.** The acceptance line may say "pass rate ≥ 0.90," but what should actually be judged is "floor score (95% lower bound) ≥ 0.90." If it falls short, add more samples — **how many runs to execute is determined by statistics, not by the schedule**.

**Move 2: Compute as you go, stop when the evidence is sufficient.** You don't have to run the full 300 every time: each run adds to the evidence; once the evidence is conclusive, render the verdict immediately; if not, keep running. This is called sequential testing (SPRT), and on average it saves 30–50% of executions — quality gates aren't slow; detours are.

**Move 3: Zero-failure claims have a ready-made formula.** To prove "reliability 0.99 at 95% confidence level," the number of zero-failure runs required (about 300) comes straight out of a statistics formula. Put it in the test plan up front, instead of retrofitting a story after the testing is done.

## 6. When reviewing a report, ask three questions

The next time any test report lands in front of you, ask three questions:

1. **How many runs?** (Sample size sets the ceiling on evidentiary strength)
2. **What's the floor score?** (Don't look at the apparent pass rate — look at the 95% lower bound)
3. **How was the acceptance line set?** (One with a derivation is a quality gate; one without is decoration)

Only when all three have answers is the report worth signing. If they don't — to put it politely — what you're holding is a "record of testing activity," not a "verification conclusion."

## 7. This verdict logic is already an engine

The verdict engine in Cyclone Core builds the three moves above into default behavior — it doesn't rely on engineer discipline:

- **Floor-score verdicts** — the acceptance line applies directly to the statistical lower bound (Supported);
- **Sequential testing** — renders PASS/FAIL early once the evidence is sufficient, saving executions (Supported);
- **Verdict evidence chain** — every verdict ships with the scenario hash, log digest, and failure snapshot, so a third party can recompute the same conclusion independently (Supported).

Because when the report has to go to regulators, auditors, or even a court, "we tested it" is worthless — "**you can recompute it yourself**" is what counts.

---

## Appendix: Glossary (in order of appearance)

| Term | Plain-language explanation |
|---|---|
| **Regression testing** | After every code change, rerun the existing test cases to confirm old functionality wasn't broken |
| **AEB** | Automatic Emergency Braking: the vehicle brakes itself when a collision is imminent. The system under test in this article's example |
| **Point estimate** | The estimate computed from a small sample, e.g. 27/30 = 0.90. Reports the answer, not its trustworthiness |
| **True pass rate** | The objectively existing but never fully measurable true value; every sample is a guess at it |
| **Confidence interval** | A range computed from the sample where the true value most likely lives, e.g. [0.744, 0.965] |
| **Confidence level** | How trustworthy the "range-computing method" is. 95% = out of 100 uses, it captures the true value about 95 times |
| **Lower bound (floor score)** | The left endpoint of the confidence interval: even in the most conservative case, the true value won't fall below it. Verdicts should be based on it |
| **Wilson method** | A standard algorithm for computing confidence intervals of pass rates; stable even with small samples |
| **Rule of three** | Rule of thumb: N consecutive runs with zero failures only guarantees a failure rate below 3/N |
| **Flaky case** | A test case that passes sometimes and fails sometimes with no code changes — a "moody" case |
| **Quality gate** | An automated checkpoint in the CI pipeline: fail to meet the bar, and nothing merges or ships |
| **SPRT (sequential testing)** | A verdict method that computes evidence as runs accumulate: stop early when evidence suffices, keep sampling otherwise |
| **Margin analysis** | The method for deriving acceptance lines: measure the baseline first, then set the threshold as "baseline − acceptable degradation" |
| **Evidence chain** | The complete set of records that lets a third party recompute a verdict: scenario hash, log digest, failure snapshot |
| **Recompute** | Take the raw evidence and run the computation again to verify the conclusion wasn't doctored |

*(The interval values in this article are statistical results computed as Wilson 95% confidence intervals; product capabilities are subject to the current version of Cyclone Core.)*
