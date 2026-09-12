---
title: "Test Tiering: The Cost Logic Behind Smoke, Regression, and Full Suites"
description: "Smoke, regression, and full suites aren't three test sets — they're three cost-budgeted ways to run one case library."
series: "Test Engineering Series · Part 1"
tn: "TN-08"
date: 2026-09-12
cta_text: "View all notes"
cta_url: "/en/#blog"
zh_url: "/blog/test-tiering-system/"
---


A freshly soldered board doesn't start with functional tests. First you **power it on and see whether it smokes** — only if it doesn't is it worth testing further. Software borrowed the idea: on every commit, run the smallest set of cases that answers one question — "**is this build obviously broken?**" If yes, don't waste resources on anything else.

But smoke only answers "what to check in five minutes." A real project's case library has hundreds or thousands of cases: what runs at night? On weekends? Before a release? This article connects three things: **how to design the smoke tier, how to do the cost math behind tiering, and the mechanism that lands it — how one case library gets run N different ways.**

## 1. Smoke Tests: Power It On, See If It Smokes

Smoke answers exactly one question: "is this build obviously broken?" Around that question, it has five design principles:

| Principle | Requirement | Anti-pattern |
|---|---|---|
| **Fast** | Hard cap of 5–10 minutes total (if MR feedback takes over 10 minutes, developers will route around the gate) | The smoke suite quietly inflates to 40 minutes |
| **Broad, not deep** | **Touch** every key module once; don't chase corners | Testing 50 boundary conditions of one module inside smoke |
| **Catch only "disaster-class" failures** | Target failure types: won't boot, won't connect, core path broken, data all wrong | Trying to catch subtle logic errors (that's the full regression's job) |
| **Zero tolerance for flaky** | One unstable case in smoke → isolate or fix it immediately — a red smoke must mean "really broken" | The team builds a "if smoke goes red, just re-run it" habit, and the gate dies |
| **Runs on every commit** | It is a gate: fail it and you can't merge | Degrading into an "informational check" that still merges when red |

How do you pick the cases? To filter those ~20 cases out of the full library, ask three questions (a case enters smoke only if all three are "yes"):

1. **If this fails, is the build basically unusable?** (P0 core journeys: boots, communicates, core functionality works)
2. **Has it caught real bugs historically?** (a case that has never failed is decoration — it goes to the back of the line)
3. **Is it fast and stable?** (≤30 seconds per case, zero variance over a hundred consecutive runs)

Then top up by **coverage**: at least one case per key module/interface — you're covering the "surface," not the "points." A typical mix: 20% startup/build, 50% core happy path, 20% key interfaces, 10% one end-to-end.

## 2. What a 5-Minute Smoke Looks Like

A time-budget example for the embedded/automotive scenario (the 5-minute tray):

```
0:00-0:30  Pull code + incremental build (ccache / distributed build cache hits)
0:30-1:00  Fast-pass static checks (high-risk MISRA subset, not the full rule set)
1:00-2:00  Unit-test smoke tier (GTest filtered by tags, ~200 cases, in parallel)
2:00-2:30  vECU boot test: image loads, scheduler starts ticking, self-check passes
2:30-4:00  Bus/diagnostics sanity: one frame over vcan, UDS 0x10 session switch +
           0x22 version read + 0x27 unlock round (three services answering = the diag stack is alive)
4:00-5:00  One end-to-end SIL scenario: replay 10 s of data → verdict → report
```

Each failure directly names "which major organ is broken" — that's what broad-not-deep means.

The engineering means that make it "fast":

- **Parallelization**: everything without inter-case dependencies runs in parallel (GTest `-j`, pytest-xdist)
- **Incremental build + compile cache**: ccache/sccache; smoke never does a full clean build
- **Environment pre-warming**: pre-built container images, a standing pool of test environments — smoke setup time often exceeds the test time itself, so this is the first optimization target
- **fail-fast**: one failure terminates the run and reports red immediately, no waiting for the rest
- **Stubbed dependencies**: smoke never calls real external services or real rigs — everything is stubs
- **Timeout circuit-breakers**: every case gets a hard timeout; a stuck case is treated as failed and flagged

Two disciplines that decide whether smoke lives or dies:

1. **Runtime inflation monitoring**: the smoke suite's total runtime goes on a dashboard; if it blows the weekly budget, cut cases — the number-one cause of smoke death is "everyone wants to stuff their case in," so there must be an explicit in/out review: to add a case, you must state which disaster-class failure it catches;
2. **Failure-attribution hygiene**: a smoke failure must be 100% attributable to a product problem. If an environment wobble caused it, fix the environment — never paper over it with a "passed on re-run." Otherwise, three months later nobody trusts smoke anymore.

Landing it in CI is straightforward: hang this one smoke command on the MR pipeline with an 8-minute timeout as backstop, and give the exit codes gate semantics — 0 lets it through, 2 blocks a product problem, 3 blocks a tooling problem (expose 3 separately to the on-call; don't let it pollute the "product is broken" signal).

One sentence to sum up: **the craft of smoke is not "what to test" but "what to resist testing"** — it's a sieve, and the mesh size sets the throughput of the whole pipeline; too coarse and disasters leak through, too fine and MRs jam.

## 3. The Essence of Tiering: An Economic Structure

Smoke only covers "the 5 minutes on every commit." When does everything else run? To answer that, first see tiering for what it is — not "some cases matter more than others," but an economic structure:

> **A case's cost scales linearly with execution frequency, while the information value it provides decays with waiting time — tiering is scheduling cases by "information yield per unit cost."**

Cost first. A case's true price is far more than "the machine time of one run":

- **Execution cost**: rig hours (HIL is the most expensive) or CPU time;
- **Opportunity cost**: while it occupies the rig/CI runner, other cases are queuing;
- **Analysis cost**: a failure needs a human to look at it. **Flaky failures are the most expensive** — they burn machine time *and* engineers' trust in red lights;
- **The frequency multiplier**: all of the above × execution frequency. A gate case that runs 20 times a day is a major line item no matter how cheap it is per run; a full-suite case that runs once a year can be as expensive as it likes.

Now the yield. The value of running a case right now = **probability of catching a regression × severity of the bug × timeliness**:

- **Detection probability varies enormously**: a core-path case can break on every change; a corner case breaks once a year — the "expected yield per execution" differs by orders of magnitude;
- **Timeliness is the hidden master variable**: a bug found 5 minutes after introduction vs one found a week later differs in repair cost by an order of magnitude — the author still has the context, and the change hasn't been buried under later commits. Automotive amplifies this one notch further: finding it at HIL = a rig-scheduling disaster;
- So **the same case is worth more in the gate than in the nightly, and more in the nightly than in the weekend run** — information depreciates.

## 4. What Each of the Three Tiers Buys

| Tier | Budget | What it buys | Selection principle |
|---|---|---|---|
| Smoke (gate / every commit) | 5–10 minutes | **Instant feedback**: tell the committer "you broke trunk" within 5 minutes | Highest failure-probability density + fastest; historical bug hotspots |
| Regression (nightly) | One nightly window | **Daily baseline**: is the system as a whole still OK today | All trunk cases + historical defect cases + requirement coverage |
| Full (weekend / release) | Weekend / release window | **Coverage evidence** and the long tail | Everything, including corners that fail once a year |

One counterintuitive point: **the tier boundary is "feedback latency," not "case importance."** The smoke set isn't "the most important cases" — it's "the most cost-effective early-warning system." An important case that takes 40 minutes is a net negative in smoke (it drags down the feedback loop) and a perfect fit for the nightly.

So don't pick the smoke set by gut: sort your cases by "historical failure count ÷ execution time" and take the top 20% as candidates — a boundary drawn from real data beats one negotiated in a review meeting.

## 5. Three Engineering Corollaries

1. **The gate's time limit is engineers' patience, not a technical constraint.** 5–10 minutes is a psychological threshold — beyond it, the gate gets routed around (no local run, push anyway, skip checks). Once the gate is bypassed, the whole system collapses. So when smoke overruns, the fix is **removing cases**, not extending the threshold;
2. **Tiering is alive; cases must flow between tiers**: flaky cases get demoted (they burn trust — keeping them is a liability), cases for newly caught bugs get promoted into regression (historical defects are the most accurate "failure probability" predictor), corners quiet for years get demoted to the full suite. The KPI is the **defect escape rate**: the more bugs escape to the next tier, the more the upper tier's filter is failing;
3. **Compliance constraints outrank economic ones**: high-ASIL cases don't participate in demotion — their yield doesn't show up in "expected bugs found" but in "execution evidence you must present at audit." So the full tier cannot be cut: tiering is "everything in its place," not "cut the expensive stuff."

## 6. Automotive: The Same Math, Amplified by Rig Hours

The math already holds at the SIL level; at the HIL level, rig hours amplify it by 10–100× — the HIL "full run" gets compressed into a "weekend/release" event. So automotive tiering is **steeper and more rigid** than internet-industry tiering: full SIL every night, a curated HIL set every night, the full HIL run saved for pre-release — the same cost logic, just different execution parameters per environment.

## 7. The Landing Mechanism: Tag Every Case

The math is done; how do you land it? The answer is **tiered tagging: give every case multiple orthogonal tags, and "which cases to run" becomes a filter expression over tags.** The SIL full set, the HIL subset, the CI smoke, the release regression are just different views of the same case library — not four case libraries. Tags aren't taxonomy; they're the shift schedule.

SIL/HIL scenarios need this especially:

- **HIL rig time is expensive**: rigs are booked in shifts and can't afford full runs, so you must be able to filter out the "must-run-on-HIL set";
- **Cases are reused across environments**: most cases that run in MIL also run in SIL, but not every SIL case can run on HIL (real ECU, fault-injection hardware) — you need environment tags to separate "can run" from "should run";
- **CI layering**: 5-minute gate smoke, nightly regression, release full — three granularities sharing one library;
- **ISO 26262 traceability**: every case must link to a requirement ID.

## 8. Tag Dimensions: Orthogonal, Not One Big Enum

| Dimension | Typical values | Question it answers |
|---|---|---|
| level | unit / component / integration / system | Which V-model level it sits at |
| env | mil / sil / pil / hil (**multi-valued**) | Which environments it can execute in |
| scope | smoke / nightly / regression / release | When it runs |
| req | REQ-AEBS-003 | Which requirement it traces to |
| asil | QM / A / B / C / D | Review and coverage obligations |

Filtering is a boolean expression: `env:hil AND scope:nightly AND NOT known_issue`. env must be multi-valued — the same AEB case often has to run on both SIL and HIL.

## 9. Implementation, from Rustic to Fancy

**(a) What the test framework gives you**

- **GoogleTest**: no native tags; the industry convention is **encoding tags in suite names** + `--gtest_filter` wildcards:

  ```bash
  # Suite naming: TEST(Smoke_Engine, Start), TEST(Hil_Aebs, ...)
  ./tests --gtest_filter='Smoke_*:*Hil_*-*Slow*'   # positive:negative patterns
  ```

  The key trick is `GTEST_SKIP()`: **probe the environment at runtime; if the HIL rig is offline, skip instead of fail** — this is the core technique for "one binary that runs in both environments." fail means "the product is wrong," skip means "preconditions not met," and the two mean entirely different things in CI statistics;
- **Catch2 / doctest**: native tags: `TEST_CASE("...", "[smoke][hil]")`, run with `./tests "[smoke]~[slow]"`;
- **pytest**: `@pytest.mark.hil` + `-m "sil and not slow"`;
- **Home-grown runner**: the framework doesn't care; define your own — usually paired with the manifest below.

**(b) A manifest (the engineering mainstream)**

Keep case metadata separate from code, in a single `tests/manifest.yaml`:

```yaml
- id: TC-ADAS-0042
  path: tests/system/adas/test_aebs.lua    # or a gtest suite name
  level: system
  env: [sil, hil]
  scope: [nightly, regression]
  req: [REQ-AEBS-003]
  asil: B
```

The runner reads the manifest → filters by expression → generates a gtest filter / case list → executes → **writes req/asil into the JUnit XML `<properties>` to report back to the ALM** (every `<testcase>` in the report must carry its requirement ID).

Benefits: changing tags doesn't touch code, and you can add lint rules (e.g. "asil B and above must have a req field, or CI rejects it" — the codification of "compliance outranks economics": compliance cases don't flow with economic sorting). On a Lua stack there's a lighter option: a header comment `-- @env sil,hil` parsed by the runner — pick one of the two.

**(c) Centralized test-management tools**

ECU-TEST package attributes, TestRail / Polarion custom fields — essentially the database version of a manifest, with the executor pulling filter results from the tool. Small teams are fine with (b); OEMs go (c).

Starting principle: **begin with just env + scope + req** — any more dimensions and nobody maintains them.

## 10. Tags and XIL: One Decides "Should It Run," the Other "Can It Run"

XIL standards govern "how scripts talk to the rig," not case management; tiered tagging sits in the case-description layer **above** XIL. The two complement each other: **tags decide "should it run"; XIL's environment abstraction decides "can it run"** — the same script connects to stubs in SIL and to the rig in HIL, and `env: [sil, hil]` runs it on both sides. (There's also the ASAM ATX format for exchanging case descriptions across the supply chain — good to know it exists.)

---

Looking back at the whole chain: **smoke is the sieve, tiering is the shift schedule, and tags are the schedule written down.** These aren't three separate topics — they're three layers of one cost logic, from idea to implementation.

---

## Appendix: Glossary (in order of appearance)

| Term | Plain explanation |
|---|---|
| **smoke test** | The smallest set of cases that quickly answers "is this build obviously broken"; named after powering on fresh hardware to see if it smokes |
| **quality gate** | An automated checkpoint in the CI pipeline: fail it and you can't merge |
| **MR** | Merge Request; the gate usually hangs on the MR pipeline |
| **happy path** | The main-flow case where all inputs are normal, no exception branches |
| **MISRA** | The automotive embedded C/C++ coding standard that static-check rule sets are usually based on |
| **vECU** | Virtual ECU: the controller software stack simulated on a PC |
| **UDS** | Unified Diagnostic Services (ISO 14229); 0x10 session switch, 0x22 read data, 0x27 security unlock are its service IDs |
| **SIL / HIL** | Software-in-the-Loop / Hardware-in-the-Loop; siblings: MIL (Model-) and PIL (Processor-) |
| **flake / flaky** | A "coin-flip" case that passes and fails without any code change |
| **defect escape rate** | The share of bugs that slip past an upper tier and get caught at a later one; measures whether the tier filter works |
| **ASIL** | Automotive Safety Integrity Level (QM/A/B/C/D, D strictest), a core ISO 26262 concept |
| **ISO 26262** | The automotive functional-safety standard; requires cases to trace back to requirements |
| **GTEST_SKIP()** | GoogleTest's runtime skip macro: when preconditions aren't met, record skip rather than fail |
| **JUnit XML** | The common test-result reporting format understood by CI systems and test-management tools |
| **ALM** | Application Lifecycle Management tooling; owns requirement–case–result traceability |
| **manifest** | A case-metadata file kept separate from code; all the tags live in it |
| **ASAM ATX** | A standard format for exchanging test-case descriptions across the supply chain |
