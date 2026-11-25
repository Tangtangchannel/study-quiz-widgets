# 题库数据片段

`templates/quiz.html` 顶部的 `QUIZ` 对象是唯一要改的地方。下面每种片段可以直接粘进 `questions` 数组。

## 单选 single

```js
{
  type: 'single',
  stem: '下列设备中，工作在<b>网络层</b>的是：',
  options: ['集线器', '二层交换机', '路由器', '中继器'],
  answer: 2,                 // 0-based；也接受 'C'
  cols: 2,                   // 可选：短选项两列排版
  hint: '看它转发时依据什么地址。',   // 可选
  explain: '路由器依据 IP 地址选路……'  // 可选，判定后显示
}
```

- 4 个选项最稳；正确答案位置在整份卷里打散。
- 判断题：两个选项 `['正确','错误']`，`answer: 0`。

## 多选 multi

```js
{
  type: 'multi',
  stem: '关于 VLAN，下列说法<b>正确</b>的有：',
  options: ['…', '…', '…', '…'],
  answers: [0, 1, 3],
  partial: true,            // 默认 true：漏选 0.5；false = 全对才得分
  explain: '…漏选得 0.5，错选不得分。'
}
```

## 填空 fill

```js
{
  type: 'fill',
  stem: 'IPv4 地址长度为 {{}} 位；RIPv2 使用 {{}} 协议，端口号 {{}}。',
  blanks: [
    { a: ['32'], w: 4 },
    { a: ['UDP', 'udp'], w: 4 },
    { a: ['520'], w: 4 }
  ],
  explain: '…'
}
```

- `{{}}` 的个数必须等于 `blanks.length`；对不上时模板会把输入框列在题干下方（带（1）（2）编号）。
- `a` 是同义答案数组；`w` 是建议字符宽度（最终 px = clamp(58, w×13, 220)）。
- 判定前统一做 NFKC、去空白、去中英文标点、转小写；因此 `UDP` / `udp` 写一个即可，但中文别名要列全。
- 带单位的答案：把 `520`、`520秒`、`520 s` 都列进 `a`，别让单位卡住。

## 连线 match

```js
{
  type: 'match',
  stem: '把协议和它的特征连起来：',
  pairs: [
    ['OSPF', '链路状态，用 SPF 计算'],
    ['RIP', '距离矢量，最大 15 跳'],
    ['BGP', '路径矢量，AS 之间'],
    ['ARP', '由 IP 求 MAC']
  ],
  order: [2, 0, 3, 1],       // 可选：右列顺序；省略时自动错位
  explain: '…'
}
```

- 不写 `order` 时右列按 `ceil(n/2)` 旋转，答案不会排成一条直线。
- 交互：点左项 → 点右项成对；再点已选中的左项取消；点右项会抢走别人已连的右项。
- 4–6 组最合适；超过 6 组连线会密。

## 阅读理解 reading

```js
{
  type: 'reading',
  title: '材料一 · Trunk 故障',
  passage: '两台华为交换机用 Trunk 互联……',
  questions: [
    { type: 'single', stem: '…', options: ['…'], answer: 1, explain: '…' },
    { type: 'fill', stem: '… {{}} …', blanks: [{ a: ['10'], w: 3 }], explain: '…' }
  ]
}
```

- 嵌套支持 single / multi / fill / match；不要嵌套 reading。
- 小问会连续编号（第 5、6 题），分组本身不计分。
- 材料 ≤600 字；再加长就拆成两组，或把材料放进可折叠区域。

## 图示大题 diagram

```js
{
  type: 'diagram',
  svg: '<svg viewBox="0 0 420 300" role="img" aria-label="…"> … </svg>',
  stem: '如图，在 △ABC 中，AB = AC = 5，BC = 6，D 是 BC 中点。',
  subs: ['（1）求 AD 的长；', '（2）求 sin∠B；'],
  reference: '解：\n（1）……',
  rubric: [
    { t: '由三线合一得到 AD ⊥ BC、BD = 3', s: 1 },
    { t: '勾股定理算出 AD = 4', s: 2 }
  ],
  rows: 3,
  placeholder: '写下你的解答思路'
}
```

- 大题默认「提交给 Hermes 批改」；「自评 / 参考答案」是可选按钮，展开后才计「大题自评 x/y」。
- `reference` 用 `\n` 换行，模板 `white-space: pre-wrap` 原样保留；`placeholder` 可覆盖输入框提示。
- 单独的图形大题优先用 `templates/diagram-question.html`（版式更大、有图注）。

## 判分速查

| 题型 | 分数 | 说明 |
|---|---|---|
| single | 0 / 1 | 选中即唯一答案 |
| multi | 0 / 0.5 / 1 | 全对 1；只选中子集 0.5；含错项 0 |
| fill | 0–1 | 正确空数 / 总空数 |
| match | 0–1 | 正确对数 / 总对数 |
| diagram | 自评 | 不进 `flat`，不参与正确率 |

## 试卷模式 paper

```js
const QUIZ = {
  mode: 'paper',
  file: '/Users/…/widgets/exam-1.html',   // 建议填绝对路径
  title: '网络基础 · 模拟卷（一）',
  subtitle: '6 题 · 25 分钟 · 交卷后统一批改',
  questions: [ /* 题型与练习模式完全相同 */ ]
};
```

- 页面不出现检查/提示/解析/参考答案/评分点，也没有对错标记；底部只有一个「交卷」。
- 空白题会先提示，再点一次「确认交卷」才提交；交卷后锁定全部输入。
- 发回 Hermes 的是 ≤500 字摘要；完整答卷先写剪贴板，摘要里提示 `pbpaste` 读取。
- 摘要只含题号 + 答案，题干/答案/评分点由 Hermes 读 `file` 指向的 HTML 获取。
- 发送失败会弹完整复制框，可点「清空答卷」重来；不会假装已交。
- 题型数据不用改；`hint` 可以留着，试卷模式不会渲染。

## 篇幅

- 一份 widget 4–6 题（阅读小问算 1 题）；练习模式目标 ≤1200px，超过就拆份。
- 试卷模式允许在 frame 内滚动（控制在 ~1700px 以内）；题数再多就拆成两卷。
- 选项短（≤10 字）时加 `cols: 2`：960 宽下两列，640px 以下自动单列，能省一半高度。
- 超了先删题，不要缩字号；客观题全部判定后会自动出总分和「错题再来一组」。
