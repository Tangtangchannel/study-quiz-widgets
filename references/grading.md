# 客观题判定细节（模板实现）

## 判定时机

- 每题一个「检查」按钮（仅练习模式），点后锁定该题并显示对错、正确答案和解析。
- 未作答时点检查只弹提示（`warn`），不判分。
- 全部客观题锁定后，底部自动出现总分条。
- 顶部「重做」重建全部题目，清空选择、输入、连线和自评。

## 分数

- 单选：`pick === answer ? 1 : 0`。
- 多选：集合完全相等 1；是答案子集且非空 0.5（`partial:false` 时 0）；含错选 0。
- 填空：`正确空数 / 总空数`，保留一位小数。
- 连线：`正确对数 / 总对数`，保留一位小数。
- 图示大题：不进客观分；只有展开「自评 / 参考答案」后才显示「大题自评 x/y」。

## 答案归一化

```js
String(v).normalize('NFKC').toLowerCase()
  .replace(/\s+/g, '')
  .replace(/[。．.，,；;：:、"'“”‘’()（）]/g, '')
```

影响：
- 大小写、全半角、空格差异自动通过。
- 标点差异自动通过；但**单位不自动通过**，需要写进 `a` 别名。
- 同义表达（如「距离矢量 / 距离向量」）必须列全，不要指望模型猜。

## 连线判定

右列渲染顺序 `order`，左项 i 的正确右列位置是 `order.indexOf(i)`。
未写 `order` 时按 `ceil(n/2)` 旋转，保证不是 1↔1 对齐。

## 试卷模式

- 不调用 `check*`，不写 `.fb`，不给卡片加 `ok / no / half`；交卷前零反馈。
- 交卷时 `buildPaperPrompt()` 生成完整 payload（题干、作答、答案、解析、评分点），写入剪贴板；
  `buildPaperDigest()` 生成 ≤ `window.hermes.maxLength`（默认 500）的摘要，`await window.hermes.send()`。
- 有空白题时两段式确认（先「确认交卷」，5 秒内再点提交）；交卷后禁用全部输入。
- 失败（`too_long` / `undelivered` / 无 bridge）显示只读完整答卷 + 复制按钮，保留「清空答卷」。

## Talkback 约束

```js
const res = await window.hermes.send(text);
if (res && res.ok === false) { /* too_long | throttled | undelivered | invalid */ }
```

- `maxLength` 默认 500；节流 1 次/秒，`throttled` 带 `retryAfterMs`。
- 长内容不要直接发：写剪贴板 + 发摘要，提示 Hermes `pbpaste`。
- 只有 `ok !== false` 才能显示“已交卷”。

## 常见错误

| 现象 | 原因 | 修法 |
|---|---|---|
| 点检查没反应 | 未作答，被 `warn` 拦下 | 先选/先填；提示 2 秒后消失 |
| 多选全对却显示 0.5 | `answers` 与选项索引错位 | 数一遍选项，0-based |
| 填空永远判错 | `blanks` 数量与 `{{}}` 不一致，走了 fallback 行 | 对齐占位符数量 |
| 连线对不上 | 手写 `order` 不是排列 | 省略 `order`，用默认错位 |
| 总分算错 | 把 diagram 塞进了 `flat` | diagram 由模板自动排除，别手改 |
