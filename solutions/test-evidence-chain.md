---
layout: solution
lang: zh
docno: CY-SOL-003
title: "可审计测试证据链：ISO 26262 / SOTIF 审核的机器证据"
description: "功能安全审核要的不是截图拼接的 Word 文档。Cyclone 证据链四件套：case hash + SHA-256 日志摘要 + 失败快照 + JUnit XML，每条 PASS/FAIL 可回溯、第三方可复算，直接对接 CI 回归看板。"
permalink: /solutions/test-evidence-chain/
en_url: /en/solutions/test-evidence-chain/
---

<h2><span class="sec-no">§1 · PROBLEM</span>审核证据的现状：截图与 Word</h2>

ISO 26262 / SOTIF 审核要回答的问题只有一个：**"这个结论，我怎么信？"** 而大多数团队交上去的证据是：测试报告（Word）、波形截图（PNG）、以及"我们跑过了"的口头承诺。这种证据有三个结构性缺陷：

1. **不可复核**。审核员无法拿着截图重新验证结论——截图本身就是结论的全部载体；
2. **不可回溯**。报告上的 PASS 对应哪个版本的用例、哪份原始数据？链条断的；
3. **不可复现**。同样的测试再做一遍，结果是否一致？没人敢当场演示。

<h2><span class="sec-no">§2 · SOLUTION</span>证据四件套，机器可验证</h2>

Cyclone 的每一次判定自动产出一套**证据链（Evidence Chain）四件套** <span class="st"><i class="sdot g"></i>已支持</span>：

| 证据件 | 作用 | 验证方式 |
|---|---|---|
| `case hash` | 用例与场景的唯一内容标识 | 内容寻址，改一个参数 hash 即变 |
| `JSONL SHA-256 digest` | 证据日志的完整性摘要 | 同 seed 重跑，摘要逐字节一致 |
| `failure snapshot` | 失败现场的结构化快照 | 直接定位判定翻转时刻 |
| `JUnit XML` | 机器可读的标准报告 | Jenkins / GitLab CI / 测试管理平台原生消费 |

每条 verdict 通过 `manifest.json` 回溯到具体的用例版本与原始数据——**不是"支持追溯"，是结构上无法切断**。

<h2><span class="sec-no">§3 · VERIFY IT YOURSELF</span>第三方复算：不信？你自己验</h2>

证据链的检验标准不是"看起来正规"，而是**第三方能否独立复算出同一个结论**：

1. 同一个用例跑两次：`cyclone run cases/aeb.yaml`；
2. 比对两次输出的 SHA-256——逐字节一致，机器可证；
3. 打开 `manifest.json`：每条 verdict 回溯到用例版本与原始数据。

因为确定性内核保证"同输入、不同结果"在结构上不可能，所以**复算不是信任投票，是机械验证**。当报告要递给监管机构、审核员甚至法庭的时候，"我们测过了"不值钱，"你可以自己复算"才值钱。

<h2><span class="sec-no">§4 · FOR SAFETY AUDITS</span>为功能安全审核设计</h2>

- **DV/PV 报告合订**：环境试验的软件伴随项——样件在气候箱里烤，台架上同步跑故障战役，两本证据合订进同一份 DV/PV 报告；
- **剂量-响应曲线**：扫描故障强度得到鲁棒性剖面，是安全档案的高级素材（见[《裕度分析三步法》](/blog/margin-analysis/)）；
- **证据格式可定制**：按审核方要求调整证据链格式，属 POC 服务项。

<h2><span class="sec-no">§5 · HONEST BOUNDARIES</span>诚实边界</h2>

- 证据链证明的是**测试过程的可信**，不替代**需求与覆盖率的论证**——那是你的安全案例（safety case）本体；
- Cyclone 不替换 CANoe / dSPACE / 程控电源体系，而是作为确定性故障注入与证据层**叠加其上**；
- 站内示例数值均为演示数据；正式证据在你的 SUT 与总线环境中产生。
