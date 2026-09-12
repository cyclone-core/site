---
layout: solution
lang: zh
docno: CY-SOL-001
title: "确定性故障注入：五种故障原语，Tick 级复现"
description: "故障注入还在拔线、改代码、写一次性脚本？Cyclone 确定性故障注入：丢帧/延迟/字段损坏/冻结/CRC 错误五种原语，YAML 声明式配置，同一故障在第 N 个 tick 必现。"
permalink: /solutions/fault-injection/
en_url: /en/solutions/fault-injection/
---

<h2><span class="sec-no">§1 · PROBLEM</span>手工故障注入的三个死结</h2>

几乎所有 ADAS / 机器人测试团队都做过故障注入，方式也惊人地一致：拔线、改代码、写一次性脚本。这条路有三个解不开的死结：

1. **无法精确复现**。同一个故障，第二次注入的时机、强度、持续窗口都和第一次不一样。FAIL 之后想回归验证修复？对不起，"那个故障"已经不存在了。
2. **结论无法对齐**。小张跑出来的 FAIL，小李的机器上跑不出来——不是代码变了，是"手气"变了。评审会上谁也说服不了谁。
3. **场景不是资产**。一次性脚本躺在个人电脑里，人不能休、脚本不敢动，更谈不上评审、版本化和 diff。

<h2><span class="sec-no">§2 · SOLUTION</span>Cyclone 的做法：故障即代码</h2>

Cyclone 的故障注入层叫 FaultPipe，核心思想一句话：**故障是声明出来的，不是手工做出来的**。

<h3>五种故障原语 <span class="st"><i class="sdot g"></i>已支持</span></h3>

| 原语 | 行为 | 典型模拟对象 |
|---|---|---|
| `Drop` | 丢帧 | 传感器丢包、总线负载过高 |
| `Delay` | 延迟 | 网络拥塞、ECU 调度抖动 |
| `CorruptField` | 字段损坏 | 传感器漂移、信号异常 |
| `Freeze` | 冻结 | 节点卡死、看门狗未触发 |
| `CorruptCrc` | CRC 错误 | EMC 干扰、线束接触不良 |

<h3>Tick 级精确触发</h3>

按时间窗、帧计数或条件表达式声明触发点。1ms 调度周期下，**同一故障在第 N 个 tick 必现**——不是"大概那时候"，是确定性调度器保证的那一格。同 seed 重跑，证据日志逐字节一致（SHA-256 可验）。

<h3>YAML 声明式配置</h3>

```yaml
# cases/aeb_frame_drop.yaml —— 一个回归用例
fault:
  type: Drop
  target: front_camera.frames
  trigger: { at_tick: 500, duration: 120 }
  pattern: burst        # 成簇丢帧，模拟接插件松动
verdict:
  metric: recall_near
  threshold: 0.98       # 阈值经裕度分析推导，见 §4
```

故障场景即代码：可评审、可版本化、可 diff。新人接手不需要"老师傅的手感"。

<h2><span class="sec-no">§3 · WHY IT MATTERS</span>注入确定性之后，判定才成立</h2>

故障注入的终点不是"注了"，而是"判了"。Cyclone 把注入和统计判定焊在同一条流水线里：故障注入 → 指标劣化 → 判定翻转 → 证据产出。判定用 Wilson 置信区间下界和 SPRT 序贯检验，把"跑了几次感觉还行"变成"在 95% 置信度下不通过"。

- [跑 30 把过 27 把，为什么不算验证通过 →](/blog/30-runs-27-pass/)
- [Wilson 下界判定：给通过率配一把诚实的尺子 →](/blog/wilson-lower-bound/)
- [SPRT 与 LLR：证据攒够就提前下判 →](/blog/sprt-llr/)

<h2><span class="sec-no">§4 · HONEST BOUNDARIES</span>诚实边界</h2>

- **物理环境试验不是软件能做的**。高低温、振动、EMC、盐雾（ISO 16750 / GB/T 28046）是气候箱和台架的活。Cyclone 做的是环境应力**投影到总线上的那一半**：漂移、丢帧、位翻转、节点冻结（部分注入方式 <span class="st"><i class="sdot a"></i>原型验证中</span> / <span class="st"><i class="sdot o"></i>规划中</span>，见[首页 §3.5](/#testmap)）。
- **接口现状**：SocketCAN / UDP / YAML DSL / JUnit XML <span class="st"><i class="sdot g"></i>已支持</span>；DDS <span class="st"><i class="sdot a"></i>原型验证中</span>；SOME/IP、UDS、ASAM XIL API <span class="st"><i class="sdot o"></i>规划中</span>。POC 阶段可按你的总线环境评估优先级。
- **阈值需要标定**。上例的 0.98 是演示值；正式验收阈值要按你的 SUT 基线做[裕度分析](/blog/margin-analysis/)推导，这属于 POC 标准服务项。
