---
name: study-quiz-widgets
description: "Use when asked to quiz or drill: self-grading widgets."
version: 1.1.0
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
4. **图先于题**：题干依赖空间关系的（几何、拓扑、电路、流程、坐标、表格）必须内联 SVG，
   不靠文字描述。
5. **沿用 chat widget 契约**（本 skill 自带最小集）：主题变量带 fallback、无外链资源、flush left、
   `minmax(0,1fr)`、正文 ≥12px、功能文字 ≥11px。文件落在 `~/.hermes/widgets/<topic>-quiz.html`，
   `::preview{file="…"}` 单独一行。
6. **配色只从注入变量取**：文字/次要文字/边框/强调色分别走 `--foreground` `--muted-foreground`
   `--border` `--accent` 的别名；绿/红/琥珀只能做低透明度语义底色（正确/错误/部分分），
   不写死白底、黑底或浅色卡片。原生 checkbox 用 `appearance:none` 自绘，避免深色主题下冒出
   白色小方块。
7. **Talkback 有配额**：`window.hermes.send()` 的 prompt 上限 500 字符（`window.hermes.maxLength`），
   返回 Promise；必须 `await` 并检查 `{ok:false, error}`（`too_long` / `throttled` / `undelivered`）。
   长答卷先写剪贴板，只发 ≤500 字的摘要并让 Hermes `pbpaste`；发送失败必须显示完整复制框，绝不假装送达。
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
- 连线题右列默认错位，别让 1 对 1 排成一条直线。
- 阅读理解材料 ≤600 字，配 3–4 题，每题都能回到原文定位。
- 图形题条件写全（边长、角度、速率、网段），图上不写答案；难题给 `hint`。

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
- 交卷时把每题题干、我的作答、标准答案/解析/评分点拼成一段 prompt；**完整 payload 写剪贴板**，
  实际发送的是 ≤500 字摘要（题号 + 答案 + 大题前若干字），并让 Hermes 用 `pbpaste` 读取完整答卷。
- 摘要示例：`批改试卷《…》（/abs/path.html）。我的答案：1C 2ABD 3=32/UDP/520 4=1C,2D 5B 6:<大题>。完整答卷在剪贴板（pbpaste）…`
- `window.hermes.send()` 必须 `await`；`{ok:false}` 时显示复制框与失败状态，并允许「清空答卷」重来。
- 交卷后锁定全部输入和「重做」；答案与评分点只存在 JS 数据与发送内容里，页面上不展示。
- 没有 `window.hermes` 的环境（`file://` 直接打开）降级成只读文本框 + 复制按钮，不静默丢答案。

复制 `templates/quiz.html`，只加 `mode: "paper"`，题型数据完全不变。

## 收到「交卷 / 提交批改」时

1. 先读剪贴板：macOS `pbpaste`、Linux `xclip -o`、Windows `Get-Clipboard`。内容应以「请批改…」开头；
   不是答卷就不要当答案。
2. 按摘要里的 `file` 路径读原 HTML 的 `QUIZ` 数据，取题干、标准答案、解析、评分点。
3. 客观题按内置答案判，图示题按评分点判；输出总分、逐题对错、每题一句解析，错题指出错在哪一步。
4. 剪贴板为空或内容不对时，让用户从卡片复制框粘贴，不要猜答案或假装已收到。

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
6. **试卷流程**：确认没有检查/提示/参考解答/评分点，交卷两次确认，发送内容包含作答与标准答案，
   交卷后没有任何 `.right` / `.wrong` / `.ok` / `.no` 标记。
7. **Talkback 三态**：mock `window.hermes.send` 返回 `{ok:true}`、`{ok:false,error:'too_long'}`、
   `{ok:false,error:'throttled',retryAfterMs}`；断言摘要 ≤ `maxLength`、失败出现复制框、节流后重试成功。
8. 汇报时写清测过的宽度和点击结果，不要只说「渲染正常」。

验证流程：三段宽度（960 / 756 / 380）、rect 测量、点击回读、截图 + vision；
如果环境里有 `chat-rich-media` / `hermes-visualization`，可以参考它们的完整 widget 规范。

## 常见坑

- 答案写进题干、提示语或 `<title>` —— 用户一眼就看到；答案只放 JS 数据。
- **Talkback 不看返回值**：`send()` 是异步且限长 500 字，超长返回 `too_long`；不 `await` 就会
  “显示已交卷但什么都没送到”。长答卷走剪贴板 + 摘要，失败弹复制框。
- **深色主题闪光弹**：写死 `#fff`/`#f8f8f8` 卡片、原生 checkbox、`background:white` 的输入框，
  在深色 app 里就是一块白板。底色一律透明或 `rgba(127,127,140,.05–.1)`，语义色只做低透明度。
- 只用 1px `--border` 下划线，深色下几乎看不见；给输入框加淡底色 + `--muted-foreground` 下划线。
- `[hidden]` 打不过 `.foot{display:flex}`：空的脚注/交卷条会白白占高度，必须写
  `.foot[hidden],.paperbar[hidden]{display:none}`。
- `::preview` 指向 scratch 文件 —— 会被清理，预览变空白；必须落在 `~/.hermes/widgets/`。
- 连线 SVG 盖住按钮 —— `.links` 必须 `pointer-events:none`，按钮 `position:relative;z-index:1`。
- 填空输入框按字符数给宽，太窄看不见自己写的答案；`w` 按最长答案估。
- 阅读理解整段塞满 → 超过 1200px；拆题或缩材料，不要缩字号。
- 用 `clarify` 出整套卷：最多 5 题、不能带图、长题干会超时。
- 只写不测：SVG 标签压住设备框、连线错位，都是渲染后才看得见。
