---
layout: solution
lang: en
docno: CY-SOL-003
title: "Auditable Test Evidence Chain: Machine Evidence for ISO 26262 / SOTIF Audits"
description: "Safety audits don't want screenshot collages. Cyclone evidence chain: case hash + SHA-256 digest + failure snapshot + JUnit XML — recomputable, CI-ready."
permalink: /en/solutions/test-evidence-chain/
zh_url: /solutions/test-evidence-chain/
---

<h2><span class="sec-no">§1 · PROBLEM</span>Audit Evidence Today: Screenshots and Word</h2>

An ISO 26262 / SOTIF audit asks exactly one question: **"Why should I trust this conclusion?"** What most teams hand over is: a test report (Word), waveform screenshots (PNG), and a verbal "we ran it." This kind of evidence has three structural defects:

1. **Not reviewable**. An auditor cannot take a screenshot and re-verify the conclusion — the screenshot is the entire carrier of the conclusion;
2. **Not traceable**. Which case version and which raw dataset does that PASS on the report map to? The chain is broken;
3. **Not reproducible**. Run the same test again — will the results match? No one dares demonstrate it live.

<h2><span class="sec-no">§2 · SOLUTION</span>A Four-Part Evidence Set, Machine-Verifiable</h2>

Every Cyclone verdict automatically produces a four-part **Evidence Chain** <span class="st"><i class="sdot g"></i>Supported</span>:

| Artifact | Purpose | How it's verified |
|---|---|---|
| `case hash` | Unique content identifier of the case and scenario | Content-addressed; change one parameter and the hash changes |
| `JSONL SHA-256 digest` | Integrity digest of the evidence log | Rerun with the same seed — byte-for-byte identical digest |
| `failure snapshot` | Structured snapshot of the failure scene | Pinpoints the exact moment the verdict flipped |
| `JUnit XML` | Machine-readable standard report | Natively consumed by Jenkins / GitLab CI / test management platforms |

Every verdict traces back through `manifest.json` to the exact case version and raw data — **not "traceability supported," but structurally impossible to sever**.

<h2><span class="sec-no">§3 · VERIFY IT YOURSELF</span>Third-Party Recompute: Don't Trust It? Verify It Yourself</h2>

The test of an evidence chain is not "does it look official" — it's **whether a third party can independently recompute the same conclusion**:

1. Run the same case twice: `cyclone run cases/aeb.yaml`;
2. Compare the two SHA-256 outputs — byte-for-byte identical, machine-provable;
3. Open `manifest.json`: every verdict traces back to a case version and raw data.

Because the deterministic kernel makes "same input, different result" structurally impossible, **recompute is not a vote of confidence — it's mechanical verification**. When the report goes to a regulator, an auditor, or even a court, "we tested it" is worthless; "you can recompute it yourself" is what counts.

<h2><span class="sec-no">§4 · FOR SAFETY AUDITS</span>Designed for Functional Safety Audits</h2>

- **DV/PV report binding**: the software companion to environmental testing — samples bake in the climate chamber while the rig runs fault campaigns in sync, and both bodies of evidence bind into the same DV/PV report;
- **Dose–response curves**: sweep fault intensity to obtain a robustness profile — premium material for the safety dossier (see [Margin Analysis in Three Steps](/en/blog/margin-analysis/));
- **Customizable evidence formats**: evidence-chain formats adjusted to the auditor's requirements, available as a POC service item.

<h2><span class="sec-no">§5 · HONEST BOUNDARIES</span>Honest Boundaries</h2>

- The evidence chain proves the **trustworthiness of the test process**; it does not replace the **argument over requirements and coverage** — that is the body of your safety case;
- Cyclone does not replace CANoe / dSPACE / programmable-power-supply setups; it **layers on top of them** as the deterministic fault-injection and evidence layer;
- All example figures on this site are demo data; formal evidence is produced on your SUT and bus environment.
