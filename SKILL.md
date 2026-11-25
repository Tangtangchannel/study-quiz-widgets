---
name: study-quiz-widgets
description: "Use when asked to quiz or drill: self-grading widgets."
version: 1.2.1
author: Hermes Agent
license: MIT
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [study, quiz, exercise, education, widget, self-grading, diagram]
    category: education
---

# 学习提问 widget

把「考考我」做成一张可点、可判分、带图的自测卡：客观题答案内置在 widget 里，点「检查」立刻出对错和解析；
几何/拓扑/流程类大题内联 SVG 图，默认把作答提交给 Hermes 批改，自评 / 参考答案只是可选项；
整卷可以零反馈作答、统一交卷。聊天正文只留一句引导，不重复题目。

## When to Use / 何时用

- 「考考我 / 出几道选择题 / 来点填空 / 连线题 / 多选题 / 阅读理解 / 来道大题 / 模拟考」
- 复习已学内容、错题重练、考前自测、看完一章的随堂测
- 用户贴一段材料说「根据这个出题」

**默认产出 widget**：用户说「考考我 / 出几道题」时直接做可点测验卡，不要退化成聊天里逐题问答；
除非用户明确说「就在聊天里念题」。当前 surface 不能渲染 `::preview` 时也照样落文件并给路径。
- 讲解完一章后说「你考考我吧 / 问题有点模糊」→ 直接出卡，别在聊天里口述连续追问（多问一题必然被读成歧义）

不用它：

- 我需要向用户**确认**信息 → `clarify` 表单，不是练习题
- 用户还没学过这个知识点 → 先讲概念（可配思维导图），再出题
- 纯背诵卡片、无判定需求 → 直接做卡片/思维导图

## 铁律

1. **客观题答案内置**：单选/多选/填空/连线的答案只存在 JS 数据里，题干和可见文本不出现答案；
   点「检查」即时判分。
2. **大题默认交给模型批改，自评只是可选项**：图形/证明/设计/排障题默认给「提交给 Hermes 批改」，把作答、
   图形条件、参考答案和评分点拼成 prompt；只有点「自评 / 参考答案」才展开答案与评分点。
   正文说清这一点，不要让用户以为只能自评。
3. **一份 4–6 题**，练习模式单页高度 ≤1200px（阅读小问算 1 题）；试卷模式允许 frame 内滚动，
   控制在 ~1700px 以内。20 题以上拆成多份 widget 或分批。
   `clarify` 最多 5 题、放不了图、长题干会超时，别用它出整套卷。
   用户明确要 8–10 题时**照样做一张卡**（别静默拆成两张，总分和「错题再来一组」需要同一份状态），
   但必须套紧凑 override 并在正文说明「卡较长、框内滚动」，见「常见坑 · 题多时的紧凑排版」。
4. **图先于题**：题干依赖空间关系的（几何、拓扑、电路、流程、坐标、表格）必须内联 SVG，
   不靠文字描述。
5. **沿用 chat widget 契约**（本 skill 自带最小集）：主题变量带 fallback、无外链资源、flush left、
   `minmax(0,1fr)`、正文 ≥12px、功能文字 ≥11px。文件落在 `$HERMES_HOME/widgets/<topic>-quiz.html`
   （默认 profile 即 `~/.hermes/widgets/`），
   `::preview{file="…"}` 单独一行。
6. **配色只从注入变量取**：文字/次要文字/边框/强调色分别走 `--foreground` `--muted-foreground`
   `--border` `--accent` 的别名；绿/红/琥珀只能做低透明度语义底色（正确/错误/部分分），
   不写死白底、黑底或浅色卡片。原生 checkbox 用 `appearance:none` 自绘，避免深色主题下冒出
   白色小方块。
7. **Talkback 有配额，但答卷必须全量回显**：`window.hermes.send()` 的 prompt 上限 500 字符
   （`window.hermes.maxLength`），返回 Promise；必须 `await` 并检查 `{ok:false, error}`
   （`too_long` / `throttled` / `undelivered`）。超过上限时按 `[作答 i/N]` 分片发送完整作答，
   最后一片带 `[END]`；不要用摘要或剪贴板代替回显。发送失败必须显示**完整**复制框，
   绝不假装送达。
8. **写完必须渲染并点击验证**，不能只写源码就交付。

## 套用模板

| 场景 | 做法 |
|---|---|
| 混合小测（单选/多选/填空/连线/阅读任意组合） | 复制 `templates/quiz.html`，只替换顶部 `QUIZ` 数据块 |
| 单道图形大题（几何、拓扑、排障、设计） | 复制 `templates/diagram-question.html`，替换 SVG、小问、参考解答、评分点 |
| 整卷 / 模拟考（做完统一交卷） | 复制 `templates/quiz.html`，在 `QUIZ` 顶部加 `mode: "paper"` |
| 只有一种题型 | 仍用 `quiz.html`，数组里只留该题型；最小数据片段见 `references/presets.md` |
| 答题卡、实验装置、地图、时间轴题等特殊版式 | 从头写；沿用上面的契约，渲染+点击验证照做 |

模板里的渲染器全都在，题型按数据自动出现，删掉不用的数据项不会留下空壳。

## 数据格式

```js
const QUIZ = {
  mode: "practice",          // "practice"（默认，逐题判）| "paper"（整卷，统一交卷）
  file: "/Users/…/widgets/topic-quiz.html",   // 建议填：批改时让 Hermes 找到原题与答案
  title: "VLAN 随堂测",
  subtitle: "5 题 · 约 6 分钟",
  questions: [
    { type:"single", stem:"…", options:["…","…","…","…"], answer:2, cols:2,
      hint:"…", explain:"…" },          // cols:2 仅限短选项，640px 以下自动变单列
    { type:"multi", stem:"…", options:["…","…","…"], answers:[0,2], cols:2,
      partial:true, explain:"…" },
    { type:"fill", stem:"IPv4 长度为 {{}} 位，端口号为 {{}}",
      blanks:[{a:["32"],w:4},{a:["520","五二零"],w:5}], explain:"…" },
    { type:"match", stem:"把协议和特征连起来", pairs:[["OSPF","链路状态"],["RIP","距离矢量"]],
      order:[1,0], explain:"…" },   // order 可省，默认自动错位
    { type:"reading", passage:"…材料…", questions:[ {type:"single",…}, {type:"fill",…} ] },
    { type:"diagram", svg:"<svg …>…</svg>", stem:"…", subs:["(1) …","(2) …"],
      reference:"解：\n① …", rubric:[{t:"…", s:2}], rows:3 }
  ]
};
```

字段细节、每种题型的最小片段和判分规则：`references/presets.md`、`references/grading.md`。

## 出题质量

- 干扰项要「像真的」：同层设备、同类协议、相邻数值，别用荒唐项凑数。
- 单选 4 项，正确答案位置打散；不要 A、B、C 连号。
- 填空按空给分，同义写法列全（大小写、全半角、单位、中文数字）；题干用 `{{}}` 占位。
- 多选在解析里写明「漏选得 0.5，错选不得分」；判断题用两个选项的单选实现。
- **多选必须画出方框**：`templates/quiz.html` 里多选和单选外观一样（选中只变蓝），用户看不出能选多个。追加固定 CSS，并在渲染后按题型标签打类（用 MutationObserver，因为「重做 / 再来一组」会重建节点，一次性打类会失效）：
  ```css
  .q.is-multi .opt::before{content:'';display:inline-block;width:11px;height:11px;border:1.5px solid rgba(127,127,140,.55);border-radius:3px;margin-right:8px;vertical-align:-1px;flex:none}
  .q.is-multi .opt.sel::before{background:var(--ac,#2f5ad0);border-color:var(--ac,#2f5ad0);box-shadow:inset 0 0 0 2px rgba(255,255,255,.92)}
  ```
  ```js
  const mark=()=>document.querySelectorAll('.q').forEach(q=>{const t=q.querySelector('.type'); if(t&&t.textContent.includes('多选')) q.classList.add('is-multi');});
  mark(); new MutationObserver(mark).observe(document.body,{childList:true,subtree:true});
  ```
  验证方式：连点两个选项，`.opt.sel` 计数应为 2；再截图确认方框和填充可见。
- 题干里用 `<code>` 写 `seq = 301` 这类值时，模板已带 `.stem code` 行内代码样式；不要另加全局 `code{}` 覆盖，否则会改到按钮和其他控件。
- 连线题右列默认错位，别让 1 对 1 排成一条直线。
- 阅读理解材料 ≤600 字，配 3–4 题，每题都能回到原文定位。
- 图形题条件写全（边长、角度、速率、网段），图上不写答案；难题给 `hint`。
- **一题只考一个变量**：题干把已知状态完整写出来（`seq = 301`、`ack = 651`、长度 200 字节），只问一个格子——报文里写多少 / 发完账本变多少 / 回包的 ack 是多少。把“两个数各是多少”塞进一道题，用户会直接回“问题有点模糊”；拆成同场景连续两题，比补一句解释有用。
- **易混概念配成对照题**：同一场景连问两题，只换提问对象（报文值 vs 账本值、seq vs ack）。错题一题就能定位混在哪一列，比换个场景重出一遍有效。

## 图形题

- 几何：`viewBox="0 0 420 300"`，辅助线 `stroke-dasharray`，等长用刻度、直角用小方块；顶点字母放形状外。
- 拓扑：设备框 + 框外标签，链路上标速率/网段；故障点用红色虚线和 ✕。
- 坐标/函数：先画轴和刻度，再画曲线；交点坐标标在交点外侧。
- 流程/状态机：`marker-end` 箭头，节点间距 ≥24px，viewBox 四周留 16px。
- 文字不压图形：CJK 标签 ≥12px，长标签拆两行或移到图外。
- 现成坐标和写法：`references/svg-recipes.md`。

## 试卷模式（mode: "paper"）

用于「来一套卷子 / 模拟考 / 做完统一交」：

- 不渲染提示、检查、解析、参考答案、评分点，也不做任何对错标注；底部只有一个「交卷」。
- 作答过程中只更新「已答 x/y」；有空白题时先提示，再点一次「确认交卷」才真正提交。
- 交卷时把**每一题的作答**（选项字母、填空值、连线对、图示题全文）按 `[作答 i/N]` 分片回显，
  最后一片带 `[END]`；每片都带 `QUIZ.file` 路径，Hermes 据此回读题干与答案。
- 示例：`试卷作答回显《…》（/abs/path.html）：\n1 单选题 C\n…\n6 图示大题 <完整解答>`。
- `window.hermes.send()` 必须 `await`；`{ok:false}` 时显示完整复制框（同样只含作答与文件路径，
  不含答案键）与失败状态，并允许「清空答卷」重来。
- 交卷后锁定全部输入和「重做」；答案与评分点只存在 JS 数据与发送内容里，页面上不展示。
- 没有 `window.hermes` 的环境（`file://` 直接打开）降级成只读文本框 + 复制按钮，不静默丢答案。

复制 `templates/quiz.html`，只加 `mode: "paper"`，题型数据完全不变。

## 收到「交卷 / 提交批改」时

1. 等 `[END]` 收齐：按 `[作答 i/N]` 的顺序收集，去掉每段开头的 `[作答 i/N]` 标记，
   拼成完整答卷；中间片段不要回复、不要对半截判分。
2. 答卷头部的 `QUIZ.file` 路径读原 HTML 的 `QUIZ` 数据，取题干、标准答案、解析、评分点。
3. 客观题按内置答案判，图示题按评分点判；输出总分、逐题对错、每题一句解析，错题指出错在哪一步。
4. 只收到部分片段、或用户从复制框粘贴时：以收到的完整文本为准；缺题就明说缺哪几题，
   让用户重发，不要猜答案或假装已收到。缺片时想读剪贴板救急，先核对头部：只有以
   「试卷作答回显 / 练习作答回显」开头才算答卷；`pbpaste` 里常躺着更早复制过的碎片
   （单个词、上一道填空），形态不对就直接让用户补发，别把旧文本当答卷。让他补发时给
   **一问一行的填法**（`1. seq ack 账本 / 2. seq ack / 3. … / 4. 理由`），比“把第 N 题再做一遍”省一个来回。
5. **连线题按渲染后的列顺序判分**：模板会在运行时打乱右列，`pairs` 里的原始顺序不是屏幕顺序。
   判分前用浏览器打开原卡，读 `.mcol.left button` / `.mcol.right button` 的文本，再把学习者写的
   `1C,2D,…` 映射过去；凭 `pairs` 顺序判会把全对判成错。

### 练习模式的「作答回显」

练习卡在**全部客观题都点过「检查」后自动回显**，试卷卡在**交卷时回显**；两种模式都会把每一题的
作答（含图示题全文）按 `[作答 i/N] … [END]` 分片发出，不再依赖剪贴板或摘要。
收到时：

- 以 `[END]` 为结束符，收齐再回；分多条到达时先攒着，去掉每段 `[作答 i/N]` 标记后按顺序拼回，
  别对半截判分。
- 只讲**错的那几题**：错在哪一步、正确角度是什么、下次怎么一眼看出来；对的题一句带过。
- 用户问“你能看到我的作答吗”时直说：看不到点击过程，但答完/交卷后卡片会把**全部作答**分片回显；
  如果只收到部分片段，说明发送中断，让他重试或从复制框粘贴。
- 「错题再来一组」发来的是**错题题干列表**，回一份新文件（`…-retry.html`）的同类题卡，别覆盖原卡；
  同一知识点、同难度，3–5 道即可，并在正文点明“考的是哪一条规则”。
- **先诊断根因再出题**：几道错题往往是同一处混淆（把报文里的值当成账本、把 ack 的值抄进 seq）。正文先用一张「你选 / 正确 / 你想成了」三列表点破，再出 3–4 道同场景对照题专打这一点；不要复述整套题、也不要逐题重讲。分数不是重点，错因是否同源才是。

## 判分语义（模板已实现，别另造）

- 题分 0–1：单选 0/1；多选全对 1、漏选 0.5、错选 0；填空按空比例；连线按对数比例。
- 判定后该题锁定并显示解析；「重做」清空全部状态。
- 客观题全部判定后出现总分、正确率和「错题再来一组」按钮（`data-hermes-send` 回传给 Hermes）。
- 图示大题不计入客观分；只有展开「自评 / 参考答案」后才显示「大题自评 x/y」。
- 试卷模式不在本地判分、不显示得分；由 Hermes 按发送的作答与评分点批改。

## 验证（交付前做完）

1. 用你惯用的静态检查器（如 impeccable）先扫一遍；没有就跳过，直接进入渲染检查。
2. `file://` 打开，在 960 / 756 / 380 三个宽度量 `scrollWidth === clientWidth`、初始 `scrollHeight ≤ 1200`
   （判定后每题反馈会多 60–90px，超出后允许 frame 内滚动）。
3. 点击链路：选项 → 检查 → 反馈出现；填空输入 → 检查；连线两次点击成对；大题展开参考解答并勾评分点。
4. 截图 + 一次 `vision_analyze`：标签是否重叠、图形是否裁切、对比度是否够。
5. **深色联动检查**：注入一套深色变量（`--foreground #e8e8ee`、`--border #33333f`、
   `--accent #7d9bff`）并把宿主背景设为深色，扫描是否还有白色背景块（`backgroundColor` 亮度和
   透明度），再截一张图确认填空线、选项框、复选框都看得见。
6. **试卷流程**：确认没有检查/提示/参考解答/评分点，交卷两次确认，发送内容包含**全部作答与
   `QUIZ.file` 路径**（不含答案键），交卷后没有任何 `.right` / `.wrong` / `.ok` / `.no` 标记。
7. **Talkback 三态 + 分片**：mock `window.hermes.send` 返回 `{ok:true}`、`{ok:false,error:'undelivered'}`、
   `{ok:false,error:'throttled',retryAfterMs}`；用一段 1000+ 字的大题答案断言切出的每片
   ≤ `maxLength`、按 `[作答 i/N]` 顺序可无损拼回、末片含 `[END]`；失败时复制框含完整作答。
8. 汇报时写清测过的宽度和点击结果，不要只说「渲染正常」。

验证流程：三段宽度（960 / 756 / 380）、rect 测量、点击回读、截图 + vision；
如果环境里有 `chat-rich-media` / `hermes-visualization`，可以参考它们的完整 widget 规范。

## 常见坑

- 答案写进题干、提示语或 `<title>` —— 用户一眼就看到；答案只放 JS 数据。
- **Talkback 不看返回值**：`send()` 是异步且限长 500 字，超长返回 `too_long`；不 `await` 就会
  “显示已交卷但什么都没送到”。长答卷必须分片回显（`[作答 i/N]` + `[END]`），失败弹完整复制框；
  不要退回摘要/剪贴板。
- **分片头没换行**：`[作答 i/N]` 后必须换行，否则 Hermes 去掉头部时会把正文第一行一起删掉；
  用 1000+ 字答案做一次拼回校验。
- **分片测试被后台节流**：headless/后台标签页的 `setTimeout` 会被压慢，4 片可能要 10s+，看起来像卡住；
  先 `Page.bringToFront` + `Emulation.setFocusEmulationEnabled`，并等到 `[END]` 再断言，别误判死循环。
- **深色主题闪光弹**：写死 `#fff`/`#f8f8f8` 卡片、原生 checkbox、`background:white` 的输入框，
  在深色 app 里就是一块白板。底色一律透明或 `rgba(127,127,140,.05–.1)`，语义色只做低透明度。
- 只用 1px `--border` 下划线，深色下几乎看不见；给输入框加淡底色 + `--muted-foreground` 下划线。
- `[hidden]` 打不过 `.foot{display:flex}`：空的脚注/交卷条会白白占高度，必须写
  `.foot[hidden],.paperbar[hidden]{display:none}`。
- `::preview` 指向 scratch 文件 —— 会被清理，预览变空白；必须落在 `$HERMES_HOME/widgets/`
  （默认 profile 即 `~/.hermes/widgets/`）。
- **preview 指向还没写出来的文件**：回复里出现 `::preview{file=…}` 之前，HTML 必须已经落盘并跑完宽度与点击验证。顺序固定：写文件 → 渲染验证 → 再写正文；先写“点这里练”再补文件，用户端只会看到「预览不可用」，还要多解释一轮。
- 连线 SVG 盖住按钮 —— `.links` 必须 `pointer-events:none`，按钮 `position:relative;z-index:1`。
- 填空输入框按字符数给宽，太窄看不见自己写的答案；`w` 按最长答案估。
- 阅读理解整段塞满 → 超过 1200px；拆题或缩材料，不要缩字号。
- **题多时的紧凑排版**：模板原样放 10 道单选，960px 宽实测 **2251px** 高，380px 宽 **2908px** —— 高度是题量决定的，压不下来就别硬压字号（正文仍 ≥12px）。加一段紧凑 override 可降到 2049px / 2664px：`.q{padding:9px 11px 8px;margin-bottom:6px}` `.stem{font-size:13px;line-height:1.6;margin:0 0 6px}` `.opts{gap:4px}` `.opt{padding:5px 8px;font-size:12px}` `.qh{margin-bottom:3px}` `.num{width:19px;height:19px}` `.fb{padding:7px 9px;font-size:11.5px}`，并在正文说明会滚动。紧凑 CSS 要作为**固定的一整段**追加到 `</style>` 前；**别拿自己上次生成的卡当模板**——按注释标记字符串切 CSS 会切出半截注释，规则静默失效（页面看起来只是“没生效”，查半天）。一律从 `templates/quiz.html` 重新复制、整块替换 `QUIZ`。六道长题干（每题带 `<code>` 与状态行）实测约 1300px，套紧凑 override 也压不进 1200px，正文说明“卡较长、框内滚动”即可。
- 用 `clarify` 出整套卷：最多 5 题、不能带图、长题干会超时。
- 只写不测：SVG 标签压住设备框、连线错位，都是渲染后才看得见。
- **用 `re.sub` 把 QUIZ JSON 灌进模板会静默损坏**：替换串里的 `\n` 被当成转义，JSON 字符串里冒出裸换行 → `const QUIZ` 整段语法错误、`typeof QUIZ === 'undefined'`、页面 0 题。一律 `re.sub(pat, lambda m: block, tpl, flags=re.S)`，别直接传 block。
- **`fill` 题干支持内联 HTML**：`renderFillStem` 按段 `innerHTML`，`<code>` / `<b>` 会正常显示；仍不要放脚本、外链或未闭合标签。
- **一条消息只显示一个 `::preview`**：发两三张只出第一张，还容易被读成“预览不可用”。整套卷要合并成单个 widget（用户明确说过“别分开”），长就让它滚动。
