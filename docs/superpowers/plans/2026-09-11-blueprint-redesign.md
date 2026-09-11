# 官网蓝图风改版实施计划

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [x]`) syntax for tracking.

**Goal:** 将官网重塑为「工程图纸风 × 规格书叙事」：整站即一份《Cyclone 规格书》，Hero 为封面，板块为 §01–§10 条款 + 附录，博客为技术笔记 TN-01…07。

**Architecture:** 设计依据 `docs/superpowers/specs/2026-09-11-blueprint-redesign-design.md`。纯静态 Jekyll，无构建工具、无外部 webfont、无 JS 框架。index.html 单文件自包含（CSS 内联），文章走 `_layouts/post.html`。文案逐字保留现有版本，只改结构、版式与编号。

**Tech Stack:** Jekyll 4（本地 `~/.local/share/gem/ruby/4.0.0/bin/jekyll`），GitHub Pages 部署。

**构建命令（每个任务通用）：**

```bash
export PATH="$HOME/.local/share/gem/ruby/4.0.0/bin:$PATH"
rm -rf _site && jekyll build 2>&1 | grep -iE 'error|warn|done'
```

预期输出：`done in X seconds`，无 error/warn。

**全局语义守恒断言（Task 1–11 每任务执行）：**

```bash
grep -o 'class="zh"' _site/index.html | wc -l   # zh 与 en 数量应相等：
grep -o 'class="en"' _site/index.html | wc -l
grep -o '已支持\|原型验证中\|规划中' _site/index.html | sort | uniq -c
# 基线值（Task 2 封面落地后实测）：已支持 12 · 原型验证中 11 · 规划中 8
# （规划中 含封面验证条新增的「单文件二进制」状态点 +1；其余与改版前相等）
# zh/en span 基线：各 174（Task 6 +1 对，Task 7 +1 对，Task 9 +2 对），应始终保持相等
# 约定：状态点 <i class="sdot"> 为装饰性元素，状态语义由相邻文字携带，不加 aria-hidden（评审 Minor 豁免）
for id in evidence pillars testmap architecture pipeline pains boundary methodology roadmap integrate scenariopack downloads blog contact; do
  grep -q "id=\"$id\"" _site/index.html || echo "MISSING ANCHOR: $id"
done; echo anchors-ok
```

---

### Task 0: 基线提交

**Files:** 无（仅 git）

- [x] **Step 1: 提交改版前全部工作**

```bash
cd /Users/caojian/tech/site
git add -A
git commit -m "feat(site): 博客系统(7篇) + 四类测试地图 + 全站诚实标签 + 场景包/方法论/架构板块"
```

预期：working tree clean（除 .DS_Store 等 ignored）。

---

### Task 1: CSS 设计系统重写

**Files:**
- Modify: `index.html`（替换整个 `<style>` 块，约第 14–200 行）

- [x] **Step 1: 用以下完整样式替换 `<style>` 全部内容**

```css
  :root {
    --paper: #fdfdfa; --ink: #16181a; --hair: #d9d9d2; --mono: #6b6f6b;
    --grn: #0a7a2f; --red: #c01818; --amb: #a06a00; --ambdot: #d9a520;
    --term-bg: #10141a; --term-fg: #d6e2d8;
  }
  * { box-sizing: border-box; }
  html { scroll-behavior: smooth; }
  body {
    font-family: -apple-system, "PingFang SC", "Segoe UI", "Microsoft YaHei", sans-serif;
    color: var(--ink); background: var(--paper); margin: 0; line-height: 1.65;
  }
  .wrap { max-width: 900px; margin: 0 auto; padding: 0 1.4rem; }
  body[data-lang="zh"] .en { display: none; }
  body[data-lang="en"] .zh { display: none; }
  .mono { font-family: ui-monospace, "SF Mono", Menlo, Consolas, monospace; }

  /* ---- 眉线导航 ---- */
  nav {
    display: flex; justify-content: space-between; align-items: baseline;
    padding: 0.8rem 0; border-bottom: 1px solid var(--ink);
    font-family: ui-monospace, "SF Mono", Menlo, monospace;
  }
  .brand { font-weight: 700; letter-spacing: 1.5px; font-size: 0.95rem; }
  .brand small { color: var(--mono); font-weight: 400; margin-left: 8px; letter-spacing: 0.5px; }
  .navlinks { display: flex; gap: 1.3rem; align-items: center; }
  .navlinks a { color: var(--mono); text-decoration: none; font-size: 0.8rem; letter-spacing: 0.5px; }
  .navlinks a:hover { color: var(--grn); }
  @media (max-width: 640px) { .navlinks a { display: none; } .navlinks a:last-of-type { display: inline; } }
  #langbtn {
    border: 1px solid var(--ink); background: transparent; border-radius: 0;
    padding: 3px 12px; cursor: pointer; font-size: 0.78rem; color: var(--ink);
    font-family: inherit;
  }
  #langbtn:hover { background: var(--ink); color: var(--paper); }

  /* ---- 封面 ---- */
  .cover {
    padding: 2.8rem 0 2.2rem;
    background-image:
      repeating-linear-gradient(0deg, transparent 0 23px, rgba(22,24,26,.045) 23px 24px),
      repeating-linear-gradient(90deg, transparent 0 23px, rgba(22,24,26,.045) 23px 24px);
    border-bottom: 2px solid var(--ink);
  }
  .cover .docmeta {
    display: flex; justify-content: space-between; gap: 0.8rem; flex-wrap: wrap;
    font-family: ui-monospace, Menlo, monospace; font-size: 0.68rem;
    letter-spacing: 1.6px; color: var(--mono);
    padding-bottom: 0.7rem; margin-bottom: 1.8rem; border-bottom: 1px solid var(--ink);
  }
  .cover h1 { font-size: 2.1rem; font-weight: 800; letter-spacing: -0.5px; line-height: 1.35; margin: 0 0 0.5rem; }
  .cover .sub-en { color: var(--mono); font-size: 0.92rem; margin-bottom: 1.4rem; }
  .abstract {
    border-left: 3px solid var(--ink); padding: 0.1rem 0 0.1rem 1rem;
    max-width: 44rem; color: #333; font-size: 0.95rem;
  }
  .abstract .lbl {
    display: block; font-family: ui-monospace, Menlo, monospace; font-size: 0.68rem;
    letter-spacing: 1.5px; color: var(--mono); margin-bottom: 0.3rem;
  }
  @media (max-width: 640px) { .cover h1 { font-size: 1.6rem; } }

  /* ---- 条款区块 ---- */
  .clause { padding: 2.2rem 0; border-bottom: 1px solid var(--hair); }
  .clause-head {
    font-family: ui-monospace, Menlo, monospace; font-size: 0.72rem;
    letter-spacing: 2px; color: var(--mono); font-weight: 600; margin: 0 0 0.3rem;
  }
  .clause-name { font-size: 1.25rem; font-weight: 700; margin: 0 0 1.1rem; }
  .clause-name .en { color: var(--mono); font-size: 0.9rem; font-weight: 400; margin-left: 8px; }
  .clause-sub { color: var(--mono); font-size: 0.92rem; margin: -0.6rem 0 1.1rem; max-width: 46rem; }

  /* 条款行：编号 + 内容，顶虚线 */
  .c-row { display: flex; gap: 1rem; padding: 0.65rem 0; border-top: 1px dashed var(--hair); font-size: 0.95rem; }
  .c-row:first-of-type { border-top: none; }
  .c-no { font-family: ui-monospace, Menlo, monospace; color: var(--mono); font-size: 0.78rem; padding-top: 0.2rem; min-width: 3.2rem; flex-shrink: 0; }
  .c-bad { color: var(--red); font-weight: 600; flex: 1; }
  .c-fix { color: #3a3d3a; flex: 1.4; }
  .c-fix::before { content: "→ "; color: var(--grn); }
  @media (max-width: 640px) { .c-row { flex-direction: column; gap: 0.25rem; } }

  /* 卡片（保留网格，改直角发丝线） */
  .cards { display: grid; grid-template-columns: 1fr 1fr; gap: 1rem; }
  .cards.three { grid-template-columns: 1fr 1fr 1fr; }
  @media (max-width: 760px) { .cards, .cards.three { grid-template-columns: 1fr; } }
  .card { border: 1px solid var(--hair); border-radius: 0; padding: 1rem 1.2rem; background: #fff; }
  .card h3 { margin: 0 0 0.4rem; font-size: 1.02rem; }
  .card h3 .en { color: var(--mono); font-weight: 400; font-size: 0.85rem; margin-left: 6px; }
  .card p { margin: 0; color: var(--mono); font-size: 0.92rem; }
  a.card { text-decoration: none; color: var(--ink); display: block; }
  a.card:hover { border-color: var(--ink); }
  .aud-who { font-family: ui-monospace, Menlo, monospace; font-size: 0.72rem; color: var(--mono); letter-spacing: 1px; text-transform: uppercase; }

  /* 状态点：●绿 已支持 / ●琥珀 原型 / ○灰 规划中 */
  .st { display: inline-flex; align-items: center; gap: 5px; font-family: ui-monospace, Menlo, monospace; font-size: 0.72rem; color: var(--mono); white-space: nowrap; }
  .sdot { width: 8px; height: 8px; border-radius: 50%; display: inline-block; flex-shrink: 0; }
  .sdot.g { background: var(--grn); } .sdot.a { background: var(--ambdot); } .sdot.o { background: transparent; border: 1.5px solid #999; }

  /* 图件 */
  .fig { margin: 1.2rem 0 0.6rem; }
  .figtag { font-family: ui-monospace, Menlo, monospace; font-size: 0.68rem; letter-spacing: 1.5px; color: var(--mono); margin-bottom: 0.4rem; }
  .figcap { font-family: ui-monospace, Menlo, monospace; font-size: 0.72rem; color: var(--mono); margin-top: 0.4rem; }

  /* 终端块（深色实物打印件） */
  .term {
    background: var(--term-bg); color: var(--term-fg); border-radius: 0;
    border: 1px solid #22303c; padding: 1rem 1.2rem;
    font-family: ui-monospace, "SF Mono", Menlo, Consolas, monospace;
    font-size: 0.85rem; overflow-x: auto; white-space: pre; line-height: 1.7;
  }
  .term .c { color: #5c6a75; } .term .p { color: #5a8fc9; } .term .ok { color: #3fd08c; }

  /* 表格 */
  table.pos { border-collapse: collapse; width: 100%; margin: 0.6rem 0; }
  table.pos th, table.pos td { border: 1px solid var(--hair); padding: 9px 13px; text-align: left; font-size: 0.92rem; vertical-align: top; }
  table.pos th { background: #f4f4ef; font-family: ui-monospace, Menlo, monospace; font-size: 0.72rem; letter-spacing: 1px; font-weight: 600; }
  table.pos td.no { color: var(--mono); }
  table.pos td.yes { color: var(--ink); }
  table.pos td.yes strong { color: var(--grn); }

  /* 流水线步骤 */
  .flow { display: flex; flex-wrap: wrap; align-items: center; gap: 0.5rem; }
  .flow .step { border: 1px solid var(--hair); border-radius: 0; padding: 0.45rem 0.9rem; font-size: 0.9rem; background: #fff; font-family: ui-monospace, Menlo, monospace; }
  .flow .arrow { color: var(--mono); }

  .facts { color: var(--mono); font-size: 0.9rem; margin-top: 1.1rem; }
  .facts a { color: var(--ink); }

  /* 深色证据带 → 保留深色，作 FIG. 5-1 实物打印件 */
  .evidence { background: var(--term-bg); color: #e6edf3; border-bottom: none; padding: 2.4rem 0; }
  .evidence .clause-head, .evidence .clause-name { color: #e6edf3; }
  .evidence .clause-head { color: #5c6a75; }
  .ev-inner { display: flex; gap: 2rem; align-items: flex-start; }
  .ev-left { flex: 1.2; }
  .ev-right { flex: 1; display: flex; flex-wrap: wrap; gap: 0.6rem; align-content: flex-start; padding-top: 0.4rem; }
  .ev-steps { margin: 0; padding-left: 1.3rem; color: #9fb3c8; font-size: 0.95rem; }
  .ev-steps li { margin-bottom: 0.5rem; }
  .ev-steps code, .ev-note code { color: #3fd08c; font-family: ui-monospace, Menlo, monospace; font-size: 0.85em; }
  .ev-note { color: #9fb3c8; font-size: 0.88rem; margin: 1rem 0 0; }
  .pill { border: 1px solid #22303c; background: #131b22; color: #9fb3c8; border-radius: 0; padding: 0.4rem 0.9rem; font-size: 0.82rem; font-family: ui-monospace, Menlo, monospace; }
  .pill b { color: #3fd08c; font-weight: 600; }
  @media (max-width: 640px) { .ev-inner { flex-direction: column; } }

  /* 话术框（四合一） */
  .map-punch { border-left: 3px solid var(--ink); background: #f4f4ef; padding: 0.8rem 1.1rem; margin-top: 1.4rem; }
  .map-punch p { margin: 0; }

  /* 下载卡 */
  .roadmap-img { width: 100%; border: 1px solid var(--hair); border-radius: 0; display: block; background: #fff; }
  .roadmap-img:hover { border-color: var(--ink); }
  .dl-card { display: flex; gap: 1rem; align-items: center; border: 1px solid var(--hair); border-radius: 0; padding: 1rem 1.2rem; text-decoration: none; color: var(--ink); background: #fff; }
  .dl-card:hover { border-color: var(--ink); }
  .dl-card img { width: 84px; height: 84px; object-fit: cover; border: 1px solid var(--hair); border-radius: 0; flex-shrink: 0; }
  .dl-card h3 { margin: 0 0 0.25rem; font-size: 1rem; }
  .dl-card p { margin: 0 0 0.25rem; color: var(--mono); font-size: 0.88rem; }
  .dl-card .meta { font-size: 0.8rem; color: var(--mono); font-family: ui-monospace, Menlo, monospace; }

  /* 文章列表 */
  .posts { display: flex; flex-direction: column; gap: 0; }
  a.post-row { display: flex; gap: 1rem; align-items: baseline; border-top: 1px dashed var(--hair); padding: 0.8rem 0; text-decoration: none; color: var(--ink); }
  a.post-row:last-child { border-bottom: 1px dashed var(--hair); }
  a.post-row:hover .post-title { color: var(--grn); }
  .post-row .meta { color: var(--mono); font-size: 0.78rem; font-family: ui-monospace, Menlo, monospace; flex-shrink: 0; min-width: 10.5rem; }
  .post-row .post-title { font-weight: 600; }
  .post-row .post-desc { color: var(--mono); font-size: 0.86rem; }
  @media (max-width: 640px) { a.post-row { flex-direction: column; gap: 0.15rem; } }

  /* 按钮：方形墨框 */
  .cta { text-align: center; padding: 2.6rem 0 3rem; }
  .cta p { color: var(--mono); }
  .btn {
    display: inline-block; border: 1px solid var(--ink); border-radius: 0;
    padding: 0.6rem 1.6rem; margin: 0 0.4rem; text-decoration: none; color: var(--ink);
    font-size: 0.85rem; font-family: ui-monospace, Menlo, monospace; letter-spacing: 1px;
    background: transparent; cursor: pointer;
  }
  .btn.primary { background: var(--ink); color: var(--paper); }
  .btn:hover { border-color: var(--grn); color: var(--grn); }
  .btn.primary:hover { background: var(--grn); border-color: var(--grn); color: #fff; }

  .wx-pop { margin-top: 1.2rem; color: var(--mono); font-size: 0.9rem; }
  .wx-pop img { width: 140px; height: 140px; object-fit: cover; border: 1px solid var(--hair); border-radius: 0; display: block; margin: 0.6rem auto; }
  .repo-link { margin-top: 1.2rem; font-size: 0.88rem; font-family: ui-monospace, Menlo, monospace; }
  .repo-link a { color: var(--mono); }
  .repo-link a:hover { color: var(--grn); }

  /* 页脚：文档控制块 */
  footer { border-top: 2px solid var(--ink); padding: 1.4rem 0 2.4rem; color: var(--mono); font-size: 0.85rem; }
  .foot-legend { margin-bottom: 0.9rem; font-size: 0.8rem; }
  .foot-legend p { margin: 0.25rem 0; }

  /* 滚动淡入（reduced-motion 关闭） */
  .reveal { opacity: 0; transform: translateY(10px); transition: opacity .45s ease, transform .45s ease; }
  .reveal.in { opacity: 1; transform: none; }
  @media (prefers-reduced-motion: reduce) { .reveal { opacity: 1; transform: none; transition: none; } }
```

同时删除旧样式中不再使用的：`.hero`、`.slogan`、`.tagline`、`.pain-row/.pain-old/.pain-new`（Task 3 会以新类重新引入，见 Task 3 Step 1）、旧 `.tier` 徽章样式（由 `.st`/`.sdot` 取代）。

- [x] **Step 2: 构建 + 语义守恒断言**

执行「构建命令」和「全局语义守恒断言」（见本计划头部）。此时页面样式会半旧半新（正常，后续任务逐区替换结构）。
预期：构建无 error；zh/en 数量相等；锚点全在。

- [x] **Step 3: Commit**

```bash
git add index.html && git commit -m "refactor(site): 蓝图风设计系统 CSS 基座"
```

---

### Task 2: 封面 Hero + 眉线导航

**Files:**
- Modify: `index.html`（nav 与 hero 区块）

- [x] **Step 1: 替换 nav**

```html
  <nav>
    <div class="brand">CYCLONE<small class="zh">确定性场景测试内核</small><small class="en">Deterministic Scenario Testing</small></div>
    <div class="navlinks">
      <a href="#pillars"><span class="zh">§02 特性</span><span class="en">§02 Features</span></a>
      <a href="#testmap"><span class="zh">§03 测试</span><span class="en">§03 Testing</span></a>
      <a href="#evidence"><span class="zh">§05 证据</span><span class="en">§05 Evidence</span></a>
      <a href="#scenariopack"><span class="zh">附录A 场景包</span><span class="en">App.A Scenarios</span></a>
      <a href="#blog"><span class="zh">附录C 笔记</span><span class="en">App.C Notes</span></a>
      <button id="langbtn" onclick="toggleLang()">EN</button>
    </div>
  </nav>
```

- [x] **Step 2: 用封面替换整个 `.hero` 区块（`<!-- ============ Hero ============ -->` 到 `</div><!-- /hero -->`）**

```html
  <!-- ============ 封面 ============ -->
  <div class="cover">
    <div class="docmeta">
      <span>CYCLONE SPECIFICATION</span>
      <span>DOC NO. CY-SPEC-001 · REV 2026-09-11</span>
    </div>
    <h1><span class="zh">同一个 case hash，<br>永远同一个测试结果。</span><span class="en">Same case hash.<br>Same test result. Every time.</span></h1>
    <p class="sub-en"><span class="zh">确定性 XiL 场景测试内核 · Deterministic XiL Scenario Testing Kernel</span><span class="en">The deterministic XiL scenario testing kernel</span></p>
    <div class="abstract">
      <span class="lbl">摘要 / ABSTRACT</span>
      <span class="zh">Cyclone 把 ADAS / 机器人回归测试收进一条流水线：数据回放 → 故障注入 → 统计判定 → 证据链。PASS 不再是"跑过一次没红"，而是可复现、可审计的机器证明。</span>
      <span class="en">Cyclone turns ADAS / robotics regression testing into one pipeline: replay → fault injection → statistical verdict → evidence chain. A PASS is no longer "it went green once" — it is reproducible, auditable machine proof.</span>
    </div>
    <div class="fig">
      <div class="figtag">FIG. 0-1 · <span class="zh">运行实录（可复算）</span><span class="en">Live run (recomputable)</span></div>
      <div class="term"><span class="p">$</span> cyclone run cases/aeb_cipv_50kph.yaml --out out/
verdict: <span class="ok">PASS</span>
<span class="p">$</span> shasum -a 256 out/AEB_CIPV_50kph_log.jsonl
2e075b42f19031aa…
<span class="p">$</span> cyclone run cases/aeb_cipv_50kph.yaml --out out2/ <span class="c"># 再跑一次 / run it again</span>
<span class="p">$</span> shasum -a 256 out2/AEB_CIPV_50kph_log.jsonl
2e075b42f19031aa…   <span class="c"># 同 seed → 逐字节一致 / same seed → bitwise identical</span></div>
      <div class="figcap"><span class="zh">图 0-1 — 同一用例两次运行，SHA-256 逐字节一致，机器可证</span><span class="en">Fig. 0-1 — Two runs of the same case, bitwise-identical SHA-256. Machine-checkable.</span></div>
    </div>
    <p class="facts" style="margin-top:1.2rem;">
      <span class="st"><i class="sdot g"></i>261 <span class="zh">项自动化测试全绿</span><span class="en">tests green</span></span> ·
      <span class="st"><i class="sdot g"></i><span class="zh">逐字节可复现</span><span class="en">bitwise reproducible</span></span> ·
      <span class="st"><i class="sdot g"></i>Wilson · SPRT <span class="zh">判定</span><span class="en">verdicts</span></span> ·
      <span class="st"><i class="sdot a"></i>rules_cyclone <span class="zh">原型</span><span class="en">prototype</span></span> ·
      <span class="st"><i class="sdot o"></i><span class="zh">单文件二进制 规划中</span><span class="en">single binary planned</span></span>
    </p>
    <p style="margin-top:1.6rem;">
      <a class="btn primary" href="mailto:caojian258@icloud.com?subject=Cyclone%20%E6%BC%94%E7%A4%BA%E9%A2%84%E7%BA%A6"><span class="zh">预约 POC 演示 →</span><span class="en">Book a POC demo →</span></a>
      <a class="btn" href="#scenariopack"><span class="zh">§A · 获取场景包</span><span class="en">App.A · Get the scenario pack</span></a>
    </p>
  </div><!-- /cover -->
```

注：`.cover` 在 `.wrap` 内；方格纹通过 CSS `background-image` 生效，无需额外元素。原 `.hero`  closing 注释 `</div><!-- /hero -->` 一并替换。原 `</div><!-- /wrap（上半） -->` 结构保留。

- [x] **Step 3: 构建 + 断言 + Commit**

```bash
# 构建 + 全局断言（计划头部）后：
grep -q 'class="cover"' _site/index.html && grep -q 'FIG. 0-1' _site/index.html && echo cover-ok
git add index.html && git commit -m "feat(site): 规格书封面 Hero + 眉线导航"
```

---

### Task 3: §01 问题定义（pains + boundary）

**Files:**
- Modify: `index.html`（#pains 与 #boundary 两区块各自条款化，保持 id 不变）

**通用规则（Task 3–10）**：各板块原有的引导段落 `<p class="sub zh" style="margin-top:-0.5rem;">…</p>` / `<p class="sub en" …>…</p>` 统一改为 `<p class="clause-sub zh">…</p>` / `<p class="clause-sub en">…</p>`（去掉内联 style；文案原样）。

**状态点转换两种定式**（沿用 Task 2/4 已确立的写法，不得漂移）：
- 标题/徽章语境（h3 内）：`<span class="st"><i class="sdot g|a|o"></i><span class="zh">已支持</span><span class="en">Supported</span></span>`（zh/en 成对）
- 段落内联语境（段落本身已有语言）：`<span class="st"><i class="sdot g|a|o"></i>已支持</span>`（裸文字，不再嵌 zh/en）
- 映射：tier ok → sdot g，tier proto → sdot a，tier plan → sdot o

- [x] **Step 1: 将 #pains 与 #boundary 合并替换为一个区块**

结构模板如下，**文案逐字取自现有 #pains 四条与 #boundary 两行**（含 zh/en 双语 span）：

```html
  <!-- ============ §01 问题定义 ============ -->
  <section class="clause reveal" id="pains">
    <p class="clause-head">§01 · PROBLEM STATEMENT</p>
    <h2 class="clause-name"><span class="zh">问题定义：老问题，新答案</span><span class="en">Problem statement: old problems, new answers</span></h2>

    <div class="c-row"><span class="c-no">§1.1</span>
      <span class="c-bad">[现有痛点1 zh/en span 原样]</span>
      <span class="c-fix">[现有答案1 zh/en span 原样]</span>
    </div>
    <!-- §1.2 PASS 无证据 / §1.3 CI 胶水 / §1.4 验收线冷场，同构 -->
  </section>

  <section class="clause reveal" id="boundary">
    <p class="clause-head">§01-B · SCOPE / NON-SCOPE</p>
    <h2 class="clause-name"><span class="zh">范围与非范围</span><span class="en">Scope and non-scope</span></h2>
    <!-- 现有 table.pos 边界表原样保留，仅表头文字改为：
         zh: 「非范围（Cyclone 不是）」「范围（Cyclone 是）」
         en: 「Non-scope」「Scope」 -->
  </section>
```

（两区块保持各自 id 不变，锚点不受损。）

- [x] **Step 2: 构建 + 断言**

额外检查：`grep -c 'c-row' _site/index.html` 应 ≥ 4；`grep -q '§01 · PROBLEM STATEMENT' _site/index.html && echo s01-ok`。

- [x] **Step 3: Commit** `git commit -m "feat(site): §01 问题定义条款化"`

---

### Task 4: §02 系统特性（pillars）

**Files:** Modify: `index.html`（#pillars）

- [x] **Step 1: 区块头改为条款头，四卡编号化，`.tier` 徽章改状态点**

```html
  <section class="clause reveal" id="pillars">
    <p class="clause-head">§02 · SYSTEM CHARACTERISTICS</p>
    <h2 class="clause-name"><span class="zh">系统特性</span><span class="en">System characteristics</span></h2>
    <div class="cards">
      <div class="card">
        <h3>§2.1 <span class="zh">确定性回放</span><span class="en">Deterministic Replay</span> <span class="st"><i class="sdot g"></i><span class="zh">已支持</span><span class="en">Supported</span></span></h3>
        <!-- 卡内文案原样 -->
      </div>
      <!-- §2.2 统计判定（g）/ §2.3 证据链（g）/ §2.4 CI 原生（内联状态点：JUnit g / rules_cyclone a / BES a） -->
    </div>
  </section>
```

CI 原生卡内联：`<span class="st"><i class="sdot g"></i>已支持</span>` 等，替换原 `<span class="tier ok">已支持</span>` 写法。

- [x] **Step 2: 构建 + 断言**：`grep -c 'sdot g' _site/index.html` 数量应 ≥ 改版前 tier ok 对应数；标签总数守恒（头部断言）。
- [x] **Step 3: Commit** `git commit -m "feat(site): §02 系统特性条款化 + 状态点"`

---

### Task 5: §03 测试类型矩阵（testmap）

**Files:** Modify: `index.html`（#testmap）

- [x] **Step 1: 区块头条款化，卡片加编号与状态点，表头已是 mono 风格**

- clause-head：`§03 · TEST-TYPE MATRIX`；clause-name：zh「测试类型矩阵：一套引擎，四种姿势」/ en「Test-type matrix: one engine, four test types」（与 Task 3 条款名前缀定式一致）。
- 四卡编号 §3.1–§3.4；卡内 `<span class="tier ok/proto">` 全部改为 `<span class="st"><i class="sdot g/a"></i>…</span>`。
- 环境应力投影子表前导语改为 `§3.5 备注` 样式：`<p class="mono" style="font-size:0.78rem;letter-spacing:1.5px;color:var(--mono);margin:1.4rem 0 0.6rem;">§3.5 · <span class="zh">环境应力的软件投影（注入方式标注能力状态）</span><span class="en">ENV-STRESS PROJECTION (INJECTION METHODS TAGGED BY STATUS)</span></p>`（保留原文括号说明与原段距）；表内 tier 徽章同样改状态点（o=规划中）。
- 四合一 `.map-punch` 保留；底部图例行改为：`● <span class="zh">已支持</span>…`（用 .st 写法），文案不变。

- [x] **Step 2: 构建 + 断言**（头部断言；标签计数守恒）
- [x] **Step 3: Commit** `git commit -m "feat(site): §03 测试类型矩阵条款化"`

---

### Task 6: §04 架构 + FIG. 4-1 流水线

**Files:** Modify: `index.html`（#architecture、#pipeline）

- [x] **Step 1: #architecture 条款化**

- clause-head：`§04 · ARCHITECTURE`；clause-name：zh「架构：一个内核，两种时间观」/ en 原样。
- 2×2 表内 `<span class="tier ok">已支持</span>` 改状态点 `.st + .sdot g`。
- facts 两行原样保留。

- [x] **Step 2: #pipeline 并入为 FIG. 4-1（保留 id="pipeline"）**

```html
  <section class="clause reveal" id="pipeline">
    <p class="clause-head">§04-B · DATA FLOW</p>
    <div class="fig">
      <div class="figtag">FIG. 4-1 · <span class="zh">判定流水线</span><span class="en">Verdict pipeline</span></div>
      <div class="flow">[现有 6 个 step 原样]</div>
      <div class="figcap"><span class="zh">图 4-1 — 从用例 YAML 到证据报告的一条流水线</span><span class="en">Fig. 4-1 — One pipeline from case YAML to evidence report.</span></div>
    </div>
    <!-- 现有 facts 行原样；UDS / rules_cyclone 的 tier 徽章改状态点 -->
  </section>
```

- [x] **Step 3: 构建 + 断言**：`grep -q 'FIG. 4-1' _site/index.html && echo fig41-ok`
- [x] **Step 4: Commit** `git commit -m "feat(site): §04 架构条款化 + 流水线图件化"`

---

### Task 7: §05 证据链规范（深色带）

**Files:** Modify: `index.html`（#evidence）

- [x] **Step 1: 深色带条款化（深色保留，作实物打印件）**

```html
  <section class="evidence" id="evidence">
    <div class="wrap">
      <p class="clause-head">§05 · EVIDENCE CHAIN SPECIFICATION</p>
      <h2 class="clause-name"><span class="zh">证据链规范</span><span class="en">Evidence chain specification</span></h2>
      <div class="ev-inner">
        <div class="ev-left">
          <div class="figtag" style="color:#5c6a75">FIG. 5-1 · <span class="zh">实物验证：不信？你自己验</span><span class="en">Physical proof — don't take our word, verify it</span></div>
          <!-- 现有 ev-steps 三步 + ev-note 四件套原样 -->
        </div>
        <div class="ev-right"><!-- 现有 4 个 pill 原样 --></div>
      </div>
    </div>
  </section>
```

- [x] **Step 2: 构建 + 断言**：`grep -q 'FIG. 5-1' _site/index.html && echo fig51-ok`
- [x] **Step 3: Commit** `git commit -m "feat(site): §05 证据链规范条款化"`

---

### Task 8: §06 判定方法学 + §07 适用范围

**Files:** Modify: `index.html`（#methodology、#audience）

- [x] **Step 1: #methodology 条款化**：clause-head `§06 · VERDICT METHODOLOGY`，clause-name zh「判定方法学：阈值是推导出来的」/ en「Verdict methodology: thresholds are derived, not guessed」（前缀定式）；三卡编号 §6.1–6.3；facts 两行原样。
- [x] **Step 2: #audience 条款化**：clause-head `§07 · APPLICABILITY`，clause-name zh「适用范围：为谁的什么问题」/ en「Applicability: who it's for」（前缀定式）；三卡编号 §7.1–7.3；机器人卡内「规划中」徽章改状态点 `.sdot o`。
- [x] **Step 3: 构建 + 断言**（头部断言）
- [x] **Step 4: Commit** `git commit -m "feat(site): §06/§07 条款化"`

---

### Task 9: §08 修订路线 + §09 集成接口

**Files:** Modify: `index.html`（#roadmap、#integrate）

- [x] **Step 1: #roadmap**：clause-head `§08 · REVISION ROADMAP`，clause-name zh「修订路线」/ en「Revision roadmap」；sub 文案原样；图片包 `.fig`：figtag `FIG. 8-1 · M0→M3 十二周路线`，figcap zh「图 8-1 — 每周可交付，三周一决策（点击下载原 PDF）」/ en「Fig. 8-1 — Weekly deliverables, a decision gate every three weeks (click for the PDF).」（后缀为已核准的实现补充）。roadmap.pdf 链接保留。
- [x] **Step 2: #integrate**：clause-head `§09 · INTEGRATION INTERFACES`，clause-name zh「集成接口」/ en「Integration interfaces」；两张卡标题编号 §9.1（Bazel，状态点 a 原型）/ §9.2（其他 CI）；终端块包 `.fig` 或保持卡内；终端注释文字原样。
- [x] **Step 3: 构建 + 断言**
- [x] **Step 4: Commit** `git commit -m "feat(site): §08/§09 条款化"`

---

### Task 10: 附录 A/B/C + 文档控制

**Files:** Modify: `index.html`（#scenariopack、#downloads、#blog、CTA、footer）

- [x] **Step 1: #scenariopack**：clause-head `APPENDIX A · SCENARIO PACK`，clause-name zh「附录 A · 场景包目录」/ en「Appendix A · Scenario pack」；目录 step 与留资 mailto 原样。
- [x] **Step 2: #downloads**：clause-head `APPENDIX B · REFERENCES`，clause-name zh「附录 B · 参考资料」/ en「Appendix B · References」；两卡原样。
- [x] **Step 3: #blog**：clause-head `APPENDIX C · TECHNICAL NOTES`，clause-name zh「附录 C · 技术笔记」/ en「Appendix C · Technical notes」；列表项加 TN 编号——post-row 改为：

```html
      {% for post in site.posts %}
      <a class="post-row" href="{{ post.url }}">
        <span class="meta">{{ post.tn }} · {{ post.date | date: "%Y-%m-%d" }}</span>
        <span class="post-title">{{ post.title }}</span>
        <span class="post-desc">{{ post.description }}</span>
      </a>
      {% endfor %}
```

（`post.tn` 在 Task 11 加入各篇 front matter；本步先改模板。）

- [x] **Step 4: CTA/footer 文档控制化**：CTA 区块加 clause-head `DOCUMENT CONTROL`，h2 与三按钮（演示/报价/微信）原样；footer 结构原样（已有图例与版本日期），**但页脚图例行里的三个 `.tier` 徽章必须转换为状态点写法**（`.tier` 的 CSS 已删除，不转换会退化成无样式文本）：

```html
<span class="st"><i class="sdot g"></i><span class="zh">已支持</span><span class="en">Supported</span></span>
<span class="st"><i class="sdot a"></i><span class="zh">原型验证中</span><span class="en">Prototype</span></span>
<span class="st"><i class="sdot o"></i><span class="zh">规划中</span><span class="en">Planned</span></span>
```

`.foot-legend` 样式已在新 CSS。
- [x] **Step 5: 构建 + 断言**：`grep -q 'APPENDIX C' _site/index.html && echo appc-ok`
- [x] **Step 6: Commit** `git commit -m "feat(site): 附录 A/B/C + 文档控制区块"`

---

### Task 11: 文章页规格书化 + TN 编号

**Files:**
- Modify: `_layouts/post.html`
- Modify: `_posts/*.md`（7 篇 front matter 各加一行 `tn`）

- [x] **Step 1: 7 篇 front matter 加 tn（按日期正序）**

```bash
cd /Users/caojian/tech/site
# 各篇 front matter 的 series: 行后插入 tn: 行：
# 2026-09-04-30-runs-27-pass.md            → tn: "TN-01"
# 2026-09-05-wilson-lower-bound.md         → tn: "TN-02"
# 2026-09-06-sprt-llr.md                   → tn: "TN-03"
# 2026-09-07-margin-analysis.md            → tn: "TN-04"
# 2026-09-08-dual-clock.md                 → tn: "TN-05"
# 2026-09-09-xil-ladder.md                 → tn: "TN-06"
# 2026-09-10-cross-industry-verification.md → tn: "TN-07"
```

用 Edit 逐篇在 `series: "…"` 行后加 `tn: "TN-0x"`。

- [x] **Step 2: `_layouts/post.html` 规格书化**

- CSS 变量对齐主站（`--paper` 等），body 背景改 `var(--paper)`；卡片/引用/表格改直角发丝线（`border-radius: 0`，边框色 `var(--hair)`，表头 mono 小号）；`.btn` 改方形墨框；footer 顶部 `2px solid var(--ink)`。
- nav 眉线化：左 `CYCLONE · TECHNICAL NOTE`，右 `{{ page.tn }} · <a href="/#blog">全部笔记 ←</a>`，全 mono。
- post-meta 行改为：`{{ page.tn }} · {{ page.date | date: "%Y-%m-%d" }} · {{ page.series }}`（mono 小号）。
- 页脚图例行文字原样，三个 tier 徽章按全站迁移转为 st/sdot 状态点（实现已落地）。

- [x] **Step 3: 构建 + 断言**

```bash
grep -o 'TN-0[1-7]' _site/index.html | sort | uniq | wc -l   # 应输出 7
grep -q 'TN-04' _site/blog/margin-analysis/index.html && echo tn-ok
```

- [x] **Step 4: Commit** `git add _layouts/post.html _posts/ && git commit -m "feat(blog): 文章页规格书化 + TN-01…07 编号"`

---

### Task 12: 滚动淡入 + 整站终验

**Files:** Modify: `index.html`（script 块）

- [x] **Step 1: 滚动淡入 JS —— 已于 Task 1 评审修复中提前落地**（commit a1c3970，含 reduced-motion 与无 IO 回退）。本步无需操作，仅核对 `_site/index.html` 含 `IntersectionObserver`。

- [x] **Step 1.5: noscript 兜底**——`.reveal` 依赖 JS 恢复可见，在 `</style>` 后加一行兜底：

```html
<noscript><style>.reveal{opacity:1;transform:none}</style></noscript>
```

- [x] **Step 1.7: 附录 C 副标对齐**（Task 10 评审建议）：#blog 的两行 `<p class="facts zh/en" style="margin-top:-0.6rem;">` 改为 `clause-sub`（文案不动），与附录 A/B 一致；同时统一残留的旧注释标签（四支柱→§02、路线图→§08、接入→§09、场景包→附录A、资料下载→附录B、博客→附录C、深色证据带→§05 证据链）。

- [x] **Step 2: 整站终验清单（全部必须过）**

```bash
export PATH="$HOME/.local/share/gem/ruby/4.0.0/bin:$PATH"
rm -rf _site && jekyll build 2>&1 | grep -iE 'error|warn' ; echo "build-clean: $?"
# 1) zh/en 数量相等
[ "$(grep -o 'class="zh"' _site/index.html | wc -l)" = "$(grep -o 'class="en"' _site/index.html | wc -l)" ] && echo bilingual-ok
# 2) 标签计数与基线一致
grep -o '已支持\|原型验证中\|规划中' _site/index.html | sort | uniq -c
# 3) 锚点 14 个全在 + 博客 CTA 指向未坏
for id in evidence pillars testmap architecture pipeline pains boundary methodology roadmap integrate scenariopack downloads blog contact; do grep -q "id=\"$id\"" _site/index.html || echo "MISSING: $id"; done; echo anchors-ok
grep -l 'href="/#scenariopack"' _site/blog/*/index.html | wc -l   # ≥ 3
# 4) 条款编号齐全
for c in '§01' '§02' '§03' '§04' '§05' '§06' '§07' '§08' '§09' 'APPENDIX A' 'APPENDIX B' 'APPENDIX C' 'DOCUMENT CONTROL' 'FIG. 0-1' 'FIG. 4-1' 'FIG. 5-1' 'FIG. 8-1'; do grep -q "$c" _site/index.html || echo "MISSING CLAUSE: $c"; done; echo clauses-ok
# 5) 无残留旧类名（tier 徽章应全部清零）
[ "$(grep -o 'class="tier' _site/index.html | wc -l | tr -d ' ')" = "0" ] && echo tier-zero
! grep -qE 'class="(hero|pain-row)' _site/index.html && echo legacy-clean
```

- [x] **Step 3: 本地视觉走查**：`jekyll serve`，开 http://localhost:4000 中英各过一遍；640px 宽度模拟移动端。

- [x] **Step 4: Commit** `git add index.html && git commit -m "feat(site): 条款滚动淡入 + 蓝图风改版收尾"`

---

## Self-Review 记录

- **规格覆盖**：板块映射 13 行 → Task 2–10 全覆盖；视觉系统 → Task 1 + 各任务替换；文章页/TN → Task 11；双语/动效/约束 → Task 1/12；验收 6 条 → Task 12 Step 2（演示数据与页脚保留由守恒断言覆盖）。
- **占位符扫描**：除 Task 3/4 明示「文案逐字取自现有板块」外（文案已存在于 index.html，属精确引用而非占位），无 TBD/TODO。
- **一致性**：`.st`/`.sdot`/`.c-row`/`.fig`/`.figtag`/`.figcap`/`.clause*` 类名在 Task 1 定义、后续任务引用一致；TN 编号 Task 10 模板引用 `post.tn`、Task 11 定义，顺序上 Task 10 Step 5 的断言不含 TN（TN 断言在 Task 11），无前置依赖问题。
