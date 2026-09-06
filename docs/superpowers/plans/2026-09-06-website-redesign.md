# Cyclone 官网改版 Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 按规格 `docs/superpowers/specs/2026-09-06-website-redesign-design.md` 把单页官网重排为"证据先行"叙事：新增深色证据带与受众分流三卡、痛点段对照化并后移、导航锚点、CTA 双通道（邮箱+微信）。

**Architecture:** 单文件 `index.html` 内联 CSS/JS，GitHub Pages 静态托管，无构建步骤。双语机制（`body[data-lang]` + localStorage）保持不变。所有改动为对 `index.html` 的增量编辑，每个任务结束页面都完整可用。

**Tech Stack:** 纯 HTML/CSS/JS；Python `http.server` 做本地预览；git 逐任务提交。

**关键约定：**
- 所有新增可见文本必须中英成对：`<span class="zh">…</span><span class="en">…</span>`，不得只写一种语言
- 每个任务的验证都基于本地预览服务（Task 1 启动），命令里的端口统一为 8000
- 微信二维码图片 `assets/wechat-qr.png` 由用户提供；未提供时代码内 `onerror` 自动回退为文字微信号，不阻塞任何任务

---

### Task 1: 保护 docs 不被 Pages 公开发布

docs/ 下有内部策略规格的 markdown，GitHub Pages 默认会公开服务它们。加 Jekyll 排除规则。

**Files:**
- Create: `/Users/caojian/tech/site/_config.yml`

- [ ] **Step 1: 创建 `_config.yml`**

内容：

```yaml
exclude:
  - docs/
```

（`.superpowers/` 是点开头目录，Jekyll 默认排除，无需列出。）

- [ ] **Step 2: 提交**

```bash
cd /Users/caojian/tech/site
git add _config.yml
git commit -m "chore: Jekyll 排除 docs/，内部规格不被 Pages 公开"
```

---

### Task 2: 启动本地预览，记录基线

**Files:** 无（环境准备）

- [ ] **Step 1: 启动预览服务**

```bash
cd /Users/caojian/tech/site && python3 -m http.server 8000
```

后台运行（`run_in_background`），后续任务共用它。

- [ ] **Step 2: 记录双语配对基线**

Run:
```bash
curl -s http://localhost:8000/ | grep -o 'class="zh"' | wc -l
curl -s http://localhost:8000/ | grep -o 'class="en"' | wc -l
```
Expected: 两个数字相等，记下基线值 N。（全站不变式：zh 与 en 出现次数始终相等，每个任务结束都要复核。）

- [ ] **Step 3: 记录现有资源可访问性基线**

Run:
```bash
for p in product-sheet.pdf roadmap.pdf assets/roadmap.webp assets/product-sheet-thumb.webp; do
  curl -s -o /dev/null -w "%{http_code} $p\n" http://localhost:8000/$p
done
```
Expected: 四行都是 `200`。

---

### Task 3: 深色证据带（新核心段）

**Files:**
- Modify: `/Users/caojian/tech/site/index.html` — CSS 区与 hero 之后

- [ ] **Step 1: `<style>` 内追加证据带样式**（加在 `.cta` 规则之前）

```css
  html { scroll-behavior: smooth; }

  .evidence { background: #0c1014; color: #e6edf3; border-top: none; padding: 2.6rem 0; }
  .evidence h2 { color: #fff; }
  .evidence h2 .en { color: #9fb3c8; }
  .ev-inner { display: flex; gap: 2rem; align-items: flex-start; }
  .ev-left { flex: 1.2; }
  .ev-right { flex: 1; display: flex; flex-wrap: wrap; gap: 0.5rem; align-content: flex-start; padding-top: 0.4rem; }
  .ev-steps { margin: 0; padding-left: 1.3rem; color: #9fb3c8; font-size: 0.95rem; }
  .ev-steps li { margin-bottom: 0.5rem; }
  .ev-steps code { color: #3fd08c; font-family: ui-monospace, "SF Mono", Menlo, Consolas, monospace; font-size: 0.85em; }
  .pill { border: 1px solid #22303c; background: #131b22; color: #9fb3c8; border-radius: 999px; padding: 0.4rem 0.9rem; font-size: 0.85rem; }
  .pill b { color: #3fd08c; font-weight: 600; }
  @media (max-width: 640px) { .ev-inner { flex-direction: column; } }
```

证据带是全宽深色块，与 `.wrap` 的 880px 限宽冲突，所以 `.evidence` 这个 section 要**移出 `.wrap`**：见 Step 2 的结构做法（在 `.wrap` 闭合后放 `.evidence`，内部再套一个 `.wrap` 限宽）。

- [ ] **Step 2: 改造页面骨架，让证据带全宽**

现状结构是 `<div class="wrap">` 包裹全部内容直到 footer。改为：

1. hero `</div>`（终端演示块闭合后）闭合外层 `</div>`（即 `.wrap` 第一次闭合）；
2. 插入证据带 section（全宽，内部 `.ev-wrap` 限宽）；
3. 再开一个新的 `<div class="wrap">` 包住后续所有段落直到 footer。

即 hero 之后变为：

```html
    </div><!-- /hero -->
  </div><!-- /wrap（上半） -->

  <!-- ============ 深色证据带 ============ -->
  <section class="evidence" id="evidence">
    <div class="wrap">
      <div class="ev-inner">
        <div class="ev-left">
          <h2><span class="zh">不信？你自己验。</span><span class="en">Don't take our word — verify it.</span></h2>
          <ol class="ev-steps">
            <li><span class="zh">同一个用例跑两次：</span><span class="en">Run the same case twice:</span> <code>cyclone run cases/aeb.yaml</code></li>
            <li><span class="zh">比对两次输出的 SHA-256——逐字节一致，机器可证。</span><span class="en">Compare the SHA-256 of both runs — bitwise identical, machine-checkable.</span></li>
            <li><span class="zh">打开 </span><span class="en">Open </span><code>manifest.json</code><span class="zh">：每条 verdict 可回溯到用例版本与原始数据。</span><span class="en">: every verdict traces to the case version and raw data.</span></li>
          </ol>
        </div>
        <div class="ev-right">
          <span class="pill"><b>261</b> <span class="zh">项自动化测试全绿</span><span class="en">automated tests green</span></span>
          <span class="pill"><span class="zh">逐字节</span><span class="en">bitwise</span> <b><span class="zh">可复现</span><span class="en">reproducible</span></b></span>
          <span class="pill">JUnit · Bazel <span class="zh">原生</span><span class="en">native</span></span>
          <span class="pill">M0→M3 <span class="zh">十二周路线</span><span class="en">twelve-week roadmap</span></span>
        </div>
      </div>
    </div>
  </section>

  <div class="wrap"><!-- 下半部分继续 -->
```

- [ ] **Step 3: 验证**

Run:
```bash
curl -s http://localhost:8000/ | grep -c 'class="evidence"'   # 期望 ≥1
curl -s http://localhost:8000/ | grep -o 'class="zh"' | wc -l  # 与下行相等
curl -s http://localhost:8000/ | grep -o 'class="en"' | wc -l
```
Expected: `class="evidence"` 出现；zh/en 计数相等且比基线 N 各多 10。
浏览器打开 `http://localhost:8000/` 目视：hero 之后紧跟全宽深色带，窄窗（<640px）时左右变上下堆叠。

- [ ] **Step 4: 提交**

```bash
cd /Users/caojian/tech/site
git add index.html
git commit -m "feat(site): 新增深色证据带（不信？你自己验 + 数据点）"
```

---

### Task 4: 受众分流三卡

**Files:**
- Modify: `/Users/caojian/tech/site/index.html`

- [ ] **Step 1: CSS 追加**（接在 Task 3 的样式后）

```css
  .cards.three { grid-template-columns: 1fr 1fr 1fr; }
  @media (max-width: 760px) { .cards.three { grid-template-columns: 1fr; } }
  a.card { text-decoration: none; color: var(--ink); display: block; }
  a.card:hover { border-color: var(--accent); }
  .aud-who { font-size: 0.78rem; color: var(--gray); letter-spacing: 0.5px; text-transform: uppercase; }
```

- [ ] **Step 2: 在证据带之后（新 `.wrap` 内最前）插入受众段**

```html
  <!-- ============ 受众分流 ============ -->
  <section id="audience">
    <h2><span class="zh">为谁的什么问题</span><span class="en">Who it's for</span></h2>
    <div class="cards three">
      <a class="card" href="#boundary">
        <span class="aud-who">OEM · <span class="zh">主机厂智驾测试团队</span><span class="en">ADAS test teams</span></span>
        <h3><span class="zh">给功能安全审核一份机器证据</span><span class="en">Machine evidence for safety audits</span></h3>
        <p class="zh">每条 PASS / FAIL 可回溯到用例版本与原始数据；证据链格式可按审核要求定制。</p>
        <p class="en">Every PASS / FAIL traces to the case version and raw data; evidence formats tailored to audit needs.</p>
      </a>
      <a class="card" href="#integrate">
        <span class="aud-who">Tier1 · <span class="zh">零部件供应商</span><span class="en">Suppliers</span></span>
        <h3><span class="zh">不替换，只增强</span><span class="en">Augment, never replace</span></h3>
        <p class="zh">叠加在 CANoe / dSPACE / 自研脚本之上；JUnit 输出直接进现有 CI。</p>
        <p class="en">Layers on top of CANoe / dSPACE / in-house scripts; JUnit output drops into your CI.</p>
      </a>
      <a class="card" href="#integrate">
        <span class="aud-who">Robotics · <span class="zh">机器人公司</span><span class="en">Robotics teams</span></span>
        <h3><span class="zh">五分钟跑通第一个场景</span><span class="en">First scenario in five minutes</span></h3>
        <p class="zh">单文件二进制下载即跑；一个 YAML 就是一个回归用例。</p>
        <p class="en">Single-file binary, download and run; one YAML is one regression case.</p>
      </a>
    </div>
  </section>
```

注：`.card p` 现有规则 `color: var(--gray)` 会同时作用于 zh/en 两段，无需改。

- [ ] **Step 3: 验证**

Run:
```bash
curl -s http://localhost:8000/ | grep -c 'id="audience"'     # 期望 1
curl -s http://localhost:8000/ | grep -o 'class="zh"' | wc -l  # 与 en 相等，各比基线 N 多 20
curl -s http://localhost:8000/ | grep -o 'class="en"' | wc -l
```
浏览器目视：三卡横排，窄屏竖排；点击卡片平滑跳到对应段（`#boundary` / `#integrate` 的 id 在 Task 6 才加，此刻点击暂无跳转，属预期）。

- [ ] **Step 4: 提交**

```bash
git add index.html
git commit -m "feat(site): 受众分流三卡（OEM / Tier1 / 机器人）"
```

---

### Task 5: 痛点段改造为"老问题 → 答案"对照并后移

**Files:**
- Modify: `/Users/caojian/tech/site/index.html`

- [ ] **Step 1: CSS 追加**

```css
  .pain-row { display: grid; grid-template-columns: 1fr 1fr; gap: 1rem; border: 1px solid var(--line); border-radius: 8px; padding: 1rem 1.2rem; margin-bottom: 0.8rem; }
  .pain-old { font-weight: 600; position: relative; padding-left: 1.4rem; }
  .pain-old::before { content: "✗"; color: #c01818; position: absolute; left: 0; }
  .pain-new { color: var(--gray); position: relative; padding-left: 1.4rem; }
  .pain-new::before { content: "→"; color: var(--accent); position: absolute; left: 0; }
  @media (max-width: 640px) { .pain-row { grid-template-columns: 1fr; } }
```

- [ ] **Step 2: 删除原痛点段**（现位于受众段之后、四支柱之前的整个 `<section>`，内含 `<ul class="pains">` 三对 li），替换为以下新段，并放在"一条流水线"段**之后**（`</section>` 流水线闭合后插入）：

```html
  <!-- ============ 痛点 → 答案 ============ -->
  <section id="pains">
    <h2><span class="zh">老问题，新答案</span><span class="en">Old problems, new answers</span></h2>
    <div class="pain-row">
      <div class="pain-old"><span class="zh">FAIL 了，分不清是产品缺陷还是测试抖动。</span><span class="en">A failure: product bug or test noise? You can't tell.</span></div>
      <div class="pain-new"><span class="zh">确定性内核让"同输入、不同结果"在结构上不可能——红就是红。</span><span class="en">The deterministic kernel makes "same input, different result" structurally impossible. Red means red.</span></div>
    </div>
    <div class="pain-row">
      <div class="pain-old"><span class="zh">报告写着 PASS，却没有机器可验证的证据。</span><span class="en">The report says PASS, with no machine-verifiable evidence.</span></div>
      <div class="pain-new"><span class="zh">每个用例产出 case hash + SHA-256 digest，两次运行逐字节一致是可验证的机器证明。</span><span class="en">Every case emits a case hash + SHA-256 digest; bitwise-identical reruns are machine-checkable proof.</span></div>
    </div>
    <div class="pain-row">
      <div class="pain-old"><span class="zh">接进 CI 要写一周胶水代码。</span><span class="en">CI integration costs a week of glue code.</span></div>
      <div class="pain-new"><span class="zh">JUnit / Bazel / BES 原生输出——不是"支持集成"，是一等公民。</span><span class="en">Native JUnit / Bazel / BES output — not "integratable", a first-class citizen.</span></div>
    </div>
  </section>
```

同时删除不再使用的 `.pains` 系列 CSS（`.pains`、`.pains li`、`.pains li::before`、`.pains li b`、`.pains .fix` 五条规则）。

- [ ] **Step 3: 验证**

Run:
```bash
curl -s http://localhost:8000/ | grep -c 'pain-row'          # 期望 ≥3（CSS 定义 + 3 行）
curl -s http://localhost:8000/ | grep -c 'class="pains"'     # 期望 0，旧列表已删
curl -s http://localhost:8000/ | grep -o 'class="zh"' | wc -l  # 与 en 相等，各比基线 N 多 23
```
浏览器目视：痛点段位于"一条流水线"之后，每行左问题右答案；窄屏变上下两行。

- [ ] **Step 4: 提交**

```bash
git add index.html
git commit -m "feat(site): 痛点段改造为老问题→新答案对照并后移"
```

---

### Task 6: 导航锚点 + 各段 id + 平滑滚动

**Files:**
- Modify: `/Users/caojian/tech/site/index.html`

- [ ] **Step 1: CSS 追加**

```css
  .navlinks { display: flex; gap: 1.2rem; align-items: center; }
  .navlinks a { color: var(--gray); text-decoration: none; font-size: 0.9rem; }
  .navlinks a:hover { color: var(--accent); }
  @media (max-width: 640px) { .navlinks a { display: none; } .navlinks a:last-of-type { display: inline; } }
```

（`html { scroll-behavior: smooth; }` 已在 Task 3 加过。）

- [ ] **Step 2: 替换 `<nav>` 块为**

```html
  <nav>
    <div class="brand">Cyclone<small class="zh">确定性场景测试内核</small><small class="en">Deterministic Scenario Testing</small></div>
    <div class="navlinks">
      <a href="#evidence"><span class="zh">证据</span><span class="en">Evidence</span></a>
      <a href="#pillars"><span class="zh">方案</span><span class="en">Product</span></a>
      <a href="#roadmap"><span class="zh">路线</span><span class="en">Roadmap</span></a>
      <a href="#downloads"><span class="zh">资料</span><span class="en">Downloads</span></a>
      <button id="langbtn" onclick="toggleLang()">EN</button>
    </div>
  </nav>
```

- [ ] **Step 3: 给既有段落加 id**（逐处把 `<section>` 改为带 id；`#evidence`、`#audience`、`#pains` 已有）：

- 四支柱段 → `<section id="pillars">`
- 一条流水线段 → `<section id="pipeline">`
- 划清边界段 → `<section id="boundary">`
- 演进路线段 → `<section id="roadmap">`
- 接入路径段 → `<section id="integrate">`
- 资料下载段 → `<section id="downloads">`

- [ ] **Step 4: 验证**

Run:
```bash
for id in evidence audience pillars pipeline boundary roadmap integrate downloads pains; do
  curl -s http://localhost:8000/ | grep -c "id=\"$id\"" | xargs echo "$id:"
done
```
Expected: 每个 id 都是 1。
浏览器目视：点击导航四个链接分别平滑滚动到对应段；Task 4 的受众卡点击 now 可跳转。

- [ ] **Step 5: 提交**

```bash
git add index.html
git commit -m "feat(site): 导航锚点 + 段落 id + 平滑滚动"
```

---

### Task 7: CTA 双通道（邮箱 + 微信浮层）与页脚联系方式

**Files:**
- Modify: `/Users/caojian/tech/site/index.html`
- （可选，用户提供）Create: `/Users/caojian/tech/site/assets/wechat-qr.png`

- [ ] **Step 1: CSS 追加**

```css
  .wx-pop { margin-top: 1.2rem; color: var(--gray); font-size: 0.9rem; }
  .wx-pop img { width: 140px; height: 140px; object-fit: cover; border: 1px solid var(--line); border-radius: 8px; display: block; margin: 0.6rem auto; }
  .repo-link { margin-top: 1.2rem; font-size: 0.88rem; }
  .repo-link a { color: var(--gray); }
  .repo-link a:hover { color: var(--accent); }
```

- [ ] **Step 2: 替换整个 `.cta` div 的内容为**

```html
  <div class="cta" id="contact">
    <h2><span class="zh">拿你自己的场景跑一次</span><span class="en">Run it on your own scenario</span></h2>
    <p class="zh">30 分钟演示：AEB 回归 → 注入故障 → 打开证据报告。</p>
    <p class="en">A 30-minute demo: AEB regression → inject a fault → open the evidence report.</p>
    <a class="btn primary" href="mailto:caojian258@icloud.com?subject=Cyclone%20%E6%BC%94%E7%A4%BA%E9%A2%84%E7%BA%A6"><span class="zh">约 30 分钟演示</span><span class="en">Book a 30-min demo</span></a>
    <button class="btn" id="wxbtn" type="button"><span class="zh">加微信</span><span class="en">Add WeChat</span></button>
    <p class="wx-pop" id="wxpop" hidden>
      <img src="assets/wechat-qr.png" alt="微信二维码 / WeChat QR" onerror="this.style.display='none'">
      <span class="zh">微信号：</span><span class="en">WeChat ID: </span><b>jiancao258</b>
    </p>
    <p class="repo-link"><a href="https://github.com/cyclone-core/rules-cyclone">GitHub: cyclone-core/rules-cyclone ↗</a></p>
  </div>
```

（删除原 CTA 里的 `TODO(deploy)` 注释与旧的 GitHub 按钮——仓库链接已降级为文字链。）

- [ ] **Step 3: 页脚追加联系方式**，footer 内现有 span 之后加：

```html
    <span class="zh"> · 联系：caojian258@icloud.com / 微信 jiancao258</span><span class="en"> · Contact: caojian258@icloud.com / WeChat jiancao258</span>
```

- [ ] **Step 4: `<script>` 内追加浮层开关**（放在 `toggleLang` 定义之后即可）

```js
  document.getElementById("wxbtn").addEventListener("click", function () {
    var p = document.getElementById("wxpop");
    p.hidden = !p.hidden;
  });
```

- [ ] **Step 5: 验证**

Run:
```bash
curl -s http://localhost:8000/ | grep -c 'mailto:caojian258@icloud.com'   # 期望 1
curl -s http://localhost:8000/ | grep -c 'wxbtn'                          # 期望 ≥2（按钮 + JS）
curl -s http://localhost:8000/ | grep -o 'class="zh"' | wc -l  # 与 en 相等，各比基线 N 多 29
```
浏览器目视：点「加微信」展开浮层——有 `assets/wechat-qr.png` 时显示二维码，没有时只显示文字微信号（回退生效）。

- [ ] **Step 6: 提交**

```bash
git add index.html
git commit -m "feat(site): CTA 双通道（mailto + 微信浮层），页脚补联系方式"
```

---

### Task 8: OG/SEO meta 补全

**Files:**
- Modify: `/Users/caojian/tech/site/index.html`

- [ ] **Step 1: 在 `<meta name="description" ...>` 之后插入**

```html
<meta property="og:title" content="Cyclone — 确定性场景测试内核">
<meta property="og:description" content="同一个 case hash，永远同一个测试结果。回放 → 故障注入 → 统计判定 → 证据链。">
<meta property="og:type" content="website">
<meta property="og:url" content="https://www.cyclone-xil.com/">
<meta name="twitter:card" content="summary">
```

- [ ] **Step 2: 文案核对（规格要求，现状已符合，不改代码）**

浏览器目视确认两项规格要求已被现状满足：Hero 的 sub 为两句话（"收进一条流水线…"+"PASS 不再是…"）；四支柱每张卡片正文不超过两行。若目视发现超长，压缩到规格要求再提交。

- [ ] **Step 3: 验证**

Run:
```bash
curl -s http://localhost:8000/ | grep -c 'property="og:'   # 期望 4
```

- [ ] **Step 4: 提交**

```bash
git add index.html
git commit -m "feat(site): 补全 OG/SEO meta"
```

---

### Task 9: 全量验收与发布

**Files:** 无（验收）

- [ ] **Step 1: 双语完整性总复核**

Run:
```bash
curl -s http://localhost:8000/ | grep -o 'class="zh"' | wc -l
curl -s http://localhost:8000/ | grep -o 'class="en"' | wc -l
```
Expected: 两数相等（应各为基线 N + 29）。

- [ ] **Step 2: 全链接检查**

Run:
```bash
for p in product-sheet.pdf roadmap.pdf assets/roadmap.webp assets/product-sheet-thumb.webp; do
  curl -s -o /dev/null -w "%{http_code} $p\n" http://localhost:8000/$p
done
curl -s -o /dev/null -w "%{http_code} wechat-qr\n" http://localhost:8000/assets/wechat-qr.png
```
Expected: 前四个 `200`；二维码 `200`（已提供）或 `404`（未提供，回退已覆盖，不阻塞）。

- [ ] **Step 3: 响应式目视**

浏览器分别用 ~1280px / ~768px / ~375px 宽度打开 `http://localhost:8000/`，确认：导航、证据带堆叠、三卡竖排、痛点行单列、终端框横向滚动均无破版；中英文各切换一遍。

- [ ] **Step 4: Lighthouse**

Chrome DevTools → Lighthouse → 跑 Performance + SEO。
Expected: 两项 ≥ 95。（无外部依赖，预计满分附近；若 webp 图片偏大导致扣分，记录后再议，不阻塞发布。）

- [ ] **Step 5: 停掉本地预览服务，提交并推送**

```bash
cd /Users/caojian/tech/site
git push
```

- [ ] **Step 6: 线上验证**（推送后等 1-2 分钟 Pages 部署）

Run:
```bash
curl -sI https://www.cyclone-xil.com/ | head -3
curl -s https://www.cyclone-xil.com/ | grep -o '<title>[^<]*</title>'
curl -s -o /dev/null -w "%{http_code}\n" https://www.cyclone-xil.com/docs/superpowers/specs/2026-09-06-website-redesign-design.md
```
Expected: 前两行 200 且标题正确；第三行 `404`（docs 已被 Jekyll 排除，未公开）。
浏览器开 https://www.cyclone-xil.com/ 走一遍完整页面。
