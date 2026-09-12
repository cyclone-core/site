---
layout: solution
lang: zh
docno: CY-SOL-002
title: "ADAS 场景回归测试：从回放到 CI 门禁的一条流水线"
description: "ADAS 回归测试散落在脚本、台架和截图里？Cyclone 把场景回放、统计判定、JUnit 报告收进一条流水线：一个 YAML 就是一个回归用例，JUnit 原生进 Jenkins/GitLab CI，Bazel 规则包让场景测试成为一等公民。"
permalink: /solutions/adas-regression-testing/
en_url: /en/solutions/adas-regression-testing/
---

<h2><span class="sec-no">§1 · PROBLEM</span>ADAS 回归测试的典型现状</h2>

场景库在仿真工程师的电脑里，回归脚本在测试工程师的电脑里，报告在 Word 里，结论在会议纪要里。每次发版前的回归，本质是一次**人肉流水线**：跑一批场景、肉眼看日志、挑几张截图、写一份"通过"。三个直接后果：

1. **回归周期以周计**，发版节奏被测试卡住；
2. **结论强度没人说得清**——"跑过 50 把全过"只能支撑"失败率 < 6%"的结论（三分律），但评审听到的只是"没问题"；
3. **FAIL 无法定性**——产品缺陷还是测试抖动？没有确定性内核，这个问题永远没有答案。

<h2><span class="sec-no">§2 · SOLUTION</span>一条流水线：回放 → 判定 → 报告</h2>

Cyclone 把 ADAS 场景回归收进一条确定性流水线：

`用例 YAML → 数据源回放 → 故障注入 → SUT → 统计判定 → 证据报告`

<h3>一个 YAML 就是一个回归用例 <span class="st"><i class="sdot g"></i>已支持</span></h3>

场景、故障、判定阈值全部声明在一个 YAML 里。改参数即得新场景——公开目录里已有 **22 个实测判定符合预期的 ADAS 场景**（主动安全 ×13、行车辅助 ×5、泊车 ×1、人机共驾 ×2、L3 准入 ×1），覆盖 AEB / LKA / ACC 等典型功能。

<h3>双模时钟：同一用例，两种物理 <span class="st"><i class="sdot g"></i>已支持</span></h3>

离线模式用虚拟时钟：在普通服务器上加速回放，同 seed 重跑逐字节一致；在线模式用真实时钟：SocketCAN/UDP 接入真实总线，硬件时间戳打点。场景 YAML 不动，拨一下时钟模式即可切换。详见[《双模时钟》](/blog/dual-clock/)。

<h3>CI 原生，不是"支持集成"</h3>

- **JUnit XML 开箱即用** <span class="st"><i class="sdot g"></i>已支持</span>，Jenkins / GitLab CI 直接消费，回归看板零改造；
- **rules_cyclone** <span class="st"><i class="sdot a"></i>原型验证中</span>：`bazel test //...` 把场景回归和单元测试放进同一张图、同一个缓存、同一块看板；
- 单文件二进制，下载即跑 <span class="st"><i class="sdot o"></i>规划中</span>。

<h2><span class="sec-no">§3 · CASE</span>一次真实的判定翻转</h2>

某 AEB 功能回归：对前视摄像头注入成簇丢帧故障后，统计判定自动翻转——

| 运行场景 | 近距召回 | 中距召回 | 远距召回 | 虚警率 | 系统判定 |
|---|---|---|---|---|---|
| 基线（无故障） | 0.9924 | 0.7040 | 0.3453 | 0.0000 | PASS |
| 注入丢帧故障 | 0.8724 | — | 0.0000 | 0.1033 | **FAIL（判定翻转）** |

<blockquote><p>演示环境数据，用于说明"故障注入 → 指标劣化 → 判定翻转 → 证据产出"的完整闭环；正式 POC 在你的真实总线与 ECU 环境中按同样方法论执行。</p></blockquote>

<h2><span class="sec-no">§4 · HONEST BOUNDARIES</span>诚实边界</h2>

- **Cyclone 不是仿真器**。CarMaker / Carla 负责"世界有多真"，Cyclone 负责"测试信不信得过"——两者是互补关系。
- **不替换现有工具链**。叠加在 CANoe / dSPACE / 自研脚本之上，JUnit 输出直接进现有 CI。
- 可靠性声明要走统计学：[300/300 才能以 95% 置信声明 ≥ 99% 可靠性](/blog/30-runs-27-pass/)，门禁阈值需按 SUT 基线标定（[裕度分析](/blog/margin-analysis/)，POC 标准服务项）。
