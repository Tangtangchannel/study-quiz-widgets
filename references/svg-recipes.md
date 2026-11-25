# 图形题的 SVG 配方

图形题只有一条硬规则：**标签在形状外，viewBox 四周留白**。下面坐标都是可直接用的骨架，改数字即可。
颜色一律 `currentColor` + `stroke-opacity`，跟随主题。

## 1. 三角形 + 高/中线（几何大题）

A 在底边中点正上方，保证等腰关系；D 为垂足，直角小方块不能省。

```html
<svg viewBox="0 0 420 300" role="img" aria-label="三角形 ABC，D 为 BC 中点">
  <g fill="none" stroke="currentColor" stroke-width="1.6" stroke-linejoin="round">
    <path d="M210 130 L120 250 L300 250 Z" stroke-opacity=".8"/>
    <path d="M210 130 L210 250" stroke-opacity=".5" stroke-dasharray="5 4"/>
    <path d="M210 238 h12 v12" stroke-opacity=".45" stroke-width="1.2"/>
  </g>
  <g fill="currentColor" font-size="12.5">
    <text x="210" y="116" text-anchor="middle" font-weight="600">A</text>
    <text x="110" y="254" text-anchor="end">B</text>
    <text x="310" y="254">C</text>
    <text x="210" y="268" text-anchor="middle">D</text>
    <text x="210" y="290" text-anchor="middle" opacity=".72">BC = 6</text>
  </g>
</svg>
```

要点：
- 顶点字母离折点 12–16px，`text-anchor` 决定向哪边让位。
- 等长刻度：取边中点，沿法线画 6px 短线；两条边方向不同，刻度方向也要不同。
- 角平分线/中线用 `dasharray="5 4"`，与实线边区分。

## 2. 圆 + 切线（圆幂、切线长）

```html
<svg viewBox="0 0 420 300" role="img" aria-label="圆 O 与过 P 点的切线">
  <g fill="none" stroke="currentColor">
    <circle cx="170" cy="150" r="80" stroke-opacity=".7" stroke-width="1.6"/>
    <path d="M170 150 L295 150" stroke-opacity=".45" stroke-dasharray="5 4"/>
    <path d="M295 150 L170 70" stroke-opacity=".7" stroke-width="1.6"/>
    <path d="M170 150 L170 70" stroke-opacity=".45"/>
    <path d="M170 138 h12 v12" stroke-opacity=".4" stroke-width="1.2"/>
  </g>
  <g fill="currentColor" font-size="12.5">
    <circle cx="170" cy="150" r="2.6"/>
    <text x="170" y="170" text-anchor="middle">O</text>
    <text x="302" y="154">P</text>
    <text x="158" y="64" text-anchor="end">A</text>
    <text x="178" y="104">r</text>
  </g>
</svg>
```

要点：切点处画直角符号；半径虚线；切线与半径的交点必须落在圆上（这里 A 在 (170,70)，正好 r=80）。

## 3. 二次函数 + 直线（坐标题）

先画轴（带箭头 `marker` 或简单线段），再画抛物线，最后标交点和坐标。

```html
<svg viewBox="0 0 420 300" role="img" aria-label="抛物线 y=x²-2x-3 与 x 轴交点">
  <g stroke="currentColor" stroke-opacity=".45" stroke-width="1.2">
    <path d="M40 240 H390"/><path d="M290 30 V270"/>
    <path d="M390 240 l-8 -4 v8 z" fill="currentColor"/>
  </g>
  <g fill="none" stroke="currentColor" stroke-width="1.7" stroke-opacity=".85">
    <path d="M120 60 Q290 360 360 60"/>   <!-- 示意：顶点与 x 轴交点按函数改写 -->
  </g>
  <g fill="currentColor" font-size="12.5">
    <text x="396" y="256">x</text><text x="296" y="24">y</text>
    <text x="112" y="262">-1</text><text x="352" y="262">3</text>
    <text x="270" y="270" text-anchor="end" opacity=".72">O</text>
  </g>
</svg>
```

要点：刻度只标关键点（交点、顶点），不要画满网格；交点坐标写在轴下方或点外侧，别压在曲线上。

## 4. 网络拓扑（排障题）

设备框 + 框外标签；链路标网段/速率；故障点用红虚线 + ✕。

```html
<svg viewBox="0 0 460 280" role="img" aria-label="两台交换机互联，SW1 上联路由器">
  <g fill="none" stroke="currentColor" stroke-opacity=".55" stroke-width="1.4">
    <path d="M110 90 H230"/><path d="M230 90 V180"/><path d="M230 180 H350"/>
  </g>
  <g fill="none" stroke="currentColor" stroke-width="1.4">
    <rect x="40" y="62" width="140" height="56" rx="8" stroke-opacity=".8"/>
    <rect x="160" y="152" width="140" height="56" rx="8" stroke-opacity=".8"/>
    <rect x="320" y="62" width="120" height="56" rx="8" stroke-opacity=".8"/>
  </g>
  <g fill="currentColor" font-size="12.5">
    <text x="110" y="95" text-anchor="middle">SW1</text>
    <text x="230" y="185" text-anchor="middle">SW2</text>
    <text x="380" y="95" text-anchor="middle">R1</text>
    <text x="170" y="48" text-anchor="middle" opacity=".7">Trunk · VLAN 10/20</text>
    <text x="292" y="142" opacity=".7">192.168.1.0/24</text>
  </g>
</svg>
```

要点：
- 设备框 140×56 是舒适尺寸；框内只放设备名，**接口和网段放框外**。
- 故障点：`stroke="#ef4444" stroke-dasharray="4 3"`，再叠一个 ✕（两条短斜线）。
- 端口标签（G0/0/1）放在链路中点上方 10px，别放进框里。

## 5. 流程 / 状态机

```html
<svg viewBox="0 0 460 220" role="img" aria-label="DHCP 四步流程">
  <defs>
    <marker id="ar" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto">
      <path d="M0 0 L10 5 L0 10 z" fill="currentColor"/>
    </marker>
  </defs>
  <g fill="none" stroke="currentColor" stroke-opacity=".6" stroke-width="1.4" marker-end="url(#ar)">
    <path d="M120 60 H190"/><path d="M270 60 H340"/>
  </g>
  <g fill="none" stroke="currentColor" stroke-width="1.4">
    <rect x="30" y="36" width="90" height="46" rx="8"/>
    <rect x="190" y="36" width="80" height="46" rx="8"/>
    <rect x="340" y="36" width="90" height="46" rx="8"/>
  </g>
  <g fill="currentColor" font-size="12">
    <text x="75" y="63" text-anchor="middle">Discover</text>
    <text x="230" y="63" text-anchor="middle">Offer</text>
    <text x="385" y="63" text-anchor="middle">Request</text>
  </g>
</svg>
```

要点：节点间距 ≥30px，箭头用 `marker-end`；`refX` 要略小于箭头宽度，否则箭头会盖住框线。

## 检查清单

- 所有文字在图形外或空白处，没压在线上。
- 关键条件（边长、角度、速率、网段、接口）都在图上。
- 图上**没有答案**（比如不要标 `AD = 4`，标 `AD = ?`）。
- 400×300 左右能放下；字号 ≥12px。
- 在 380px 宽度下渲染一次，最靠边的标签没有出界。
