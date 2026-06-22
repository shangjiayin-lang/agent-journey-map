---
name: agent-journey-map
description: 帮助产品经理创建 Agent Journey Map（AI Agent 旅程图）—— 一种专门可视化 AI Agent 执行路径的交互式 SVG 泳道流程图。当用户提到 "agent journey map"、"AJM"、"AI agent 流程图"、"泳道图"、"帮我梳理 agent 的执行流程"、"画一下这个 agent 怎么跑的"，或者想要记录 AI Agent 的推理步骤、API 调用、风险点、决策分支时，立即触发本 skill。即使用户不知道 "agent journey map" 这个词，只要他们想可视化一个 AI Agent 的工作流程，都应该触发。
---

# Agent Journey Map Skill

## 这个 Skill 做什么

帮助产品经理（尤其是不熟悉 AJM 概念的 PM）通过对话式问答，快速生成一份**交互式 SVG 泳道流程图 HTML 文件**，用来可视化 AI Agent 完成任务的完整执行路径——包括用户交互、Agent 推理、系统调用和风险应对。

生成的 HTML 文件支持在浏览器里直接拖拽、编辑节点、连线，适合团队协作和迭代。

---

## 执行流程

### 第一步：开场 + 判断用户熟悉度

用一句简短的概念介绍打开对话，**不要先写一大段解释**，把概念融入第一个问题里：

> "Agent Journey Map 是一种泳道流程图，专门用来梳理 AI Agent 完成任务时的每一步推理、API 调用和风险点——跟用户旅程图类似，但视角在 Agent 内部。我来帮你做一张，先告诉我：**这个 Agent 是给谁用的？**"

如果 PM 回复表示不太了解（如"AJM 是什么"、"能先介绍一下吗"、"我不太清楚"），则先生成**概念参考卡**（见下方"概念参考卡"章节），再回到信息收集流程。

---

### 第二步：信息收集（5 个核心问题）

分 2-3 轮提问，不要一次全部抛出。建议第一轮问 Q1+Q2，第二轮问 Q3+Q4，第三轮问 Q5。

**Q1 — 目标用户**
> "这个 Agent 的目标用户是谁？（例如：企业内部员工、外部客户、运营团队……）"

**Q2 — 用户目标**
> "用户用这个 Agent 是为了完成什么任务或解决什么问题？"

**Q3 — 交互方式（入口）**
> "用户可以通过什么方式使用这个 Agent？（比如：自然语言对话、上传文件、填表单、语音……可以多选）"

**Q4 — 系统依赖**
> "这个 Agent 运行时需要调用哪些系统、服务或 API？（不确定也可以说大概）"

**Q5 — 风险与失败场景**
> "你最担心这个 Agent 在哪些情况下会出问题？比如：理解歧义、数据不准确、系统超时……"

收集完 5 个问题的答案后，**用 2-3 句话复述你的理解**，确认没有偏差后再开始生成。

---

### 第三步：生成 Agent Journey Map HTML

根据收集到的信息，生成一个**完整的单文件交互式 SVG 泳道流程图**。

#### 文件命名
`agent-journey-map-[场景关键词].html`，保存到用户的输出目录，**绝不覆盖已有文件**。

#### 泳道设计（4 条，固定，中英双语标签）

| 泳道 ID | 中文名 | 英文名 | 内容 |
|---------|--------|--------|------|
| `user` | 用户 | User | 用户输入、补充信息、确认操作 |
| `agent` | 智能体推理 | Agent Reasoning | Agent 的理解、规划、决策（含菱形判断节点）|
| `system` | 系统调用 | System & API | 外部系统、数据库、API 的调用节点 |
| `risk` | 风险与缓解 | Risk & Mitigation | 风险节点（红色）+ 缓解节点（绿色）成对出现 |

#### 节点类型

| type | 形状 | 用途 |
|------|------|------|
| `message` | 气泡 | 用户的自然语言输入 |
| `rect` | 圆角矩形 | 动作节点（Agent 操作、API 调用、缓解措施）|
| `diamond` | 菱形 | 决策/判断节点 |
| `note` | 便签形 | 风险节点（深红色描边）|

#### 流程设计要点

- **决策菱形**：至少 1-2 个，体现 Agent 的关键判断逻辑（如"信息是否完整？"）
- **反问循环**：决策菱形的 "No" 分支 → 向用户反问补充 → 重新进入推理
- **风险-缓解配对**：每个风险节点（note）都要有对应的缓解节点，用虚线连接
- **节点数量**：15-30 个节点为宜
- **边标签**：关键分支的箭头要加标签（如 "Yes / No"、"补充信息"、"确认"）

#### HTML 完整功能清单

生成的 HTML 必须包含以下所有功能：

```
核心画布
- SVG 画布，逻辑宽度 1820（通过 viewBox 实现，非写死 px）
- 高度根据泳道数量动态计算
- 左侧泳道标签区（LBL_W = 56px），标签旋转 -90°，中英双语两行显示
- 顶部工具栏（标题、工具按钮、导出按钮、主题切换按钮）
- 右下角 Legend 图例（固定定位）

主题（必须，见下方"双主题系统"章节）
- 内置 light / dark 两套完整主题，所有颜色均来自 THEMES 对象，禁止散落硬编码
- 工具栏最右侧（导出按钮右边）放主题切换按钮：dark 下显示"☀️ Light"，light 下显示"🌙 Dark"
- 按钮带 aria-label="切换深色/浅色主题" 和可见的 :focus-visible 焦点轮廓
- 默认主题跟随系统 prefers-color-scheme；用户切换后用 localStorage 记住（try/catch 包裹，失败则仅本次会话生效）
- 切换时整页跟随：页面背景、工具栏、泳道、节点、边、箭头 marker、图例、弹窗
- 导出 SVG 使用当前主题的颜色

节点交互
- 拖拽移动（鼠标按住拖动）
- 双击编辑文字（弹出输入框）
- 右键菜单（编辑、删除）
- 选中高亮（蓝色描边）

工具模式（工具栏按钮切换）
- 选择/移动模式（默认）
- 添加矩形节点
- 添加菱形节点
- 添加风险便签节点
- 连线模式（拖拽连接两个节点）
- 删除模式

边（箭头）
- 贝塞尔曲线路径，自动计算最优锚点
- 支持虚线（dashed: true）用于风险关联
- 边中点显示可编辑标签
- SVG arrowhead marker（兼容写法，见下方）

泳道管理
- 双击泳道标签可重命名（两步提示：先中文后英文）

导出
- 导出 SVG 按钮（下载当前画布为 .svg 文件）
```

---

## ⚠️ 跨平台渲染兼容性规范（必须严格遵守，违反会导致箭头/连线错位）

不同 AI 平台（Coze、Dify、扣子、智谱、文心、自定义 Agent 平台等）的 artifact iframe 有不同的容器宽度、缩放比例、scroll offset 和 CSS transform。以下规范是保证跨平台正确渲染的**强制要求**。

---

### 规范 1：节点坐标必须是 SVG 绝对坐标

节点的 `x`、`y` 必须是 **SVG 画布上的绝对坐标**（包含泳道偏移），不能是泳道内的相对偏移。

```javascript
// ✅ 正确：x/y 是 SVG 绝对坐标
// 泳道 user 的 startY = TOOLBAR_H（假设 56），节点在泳道内偏移 37
{ id:'u1', lane:'user', x:280, y:93, w:200, h:65, type:'message', text:'用户输入...' }
//                                  ↑ = TOOLBAR_H(56) + laneOffset(37)

// ❌ 错误：x/y 是泳道内相对坐标，再靠 DOM 换算绝对坐标
{ id:'u1', lane:'user', x:280, y:37, ... }  // 然后用 getBoundingClientRect 补算
```

泳道的绝对 Y 起始位置在初始化时一次性计算并存入 lane 对象：

```javascript
const TOOLBAR_H = 56;  // 顶部工具栏高度
const LBL_W     = 56;  // 左侧泳道标签宽度
const CANVAS_W  = 1820; // SVG 逻辑宽度（viewBox 用）

function computeLanePositions() {
  let y = TOOLBAR_H;
  lanes.forEach(lane => {
    lane.startY = y;
    y += lane.h;
  });
  return y; // totalHeight，用于 viewBox
}
```

初始化节点时，y 值 = `lane.startY + 节点在泳道内的垂直偏移`：

```javascript
function initNodes(rawNodes) {
  // rawNodes 里的 y 是泳道内偏移，初始化时转为绝对坐标
  return rawNodes.map(n => {
    const lane = lanes.find(l => l.id === n.lane);
    return { ...n, y: lane.startY + n.y };
  });
}
```

---

### 规范 2：SVG 必须使用 viewBox，禁止写死像素宽高

```html
<!-- ✅ 正确：viewBox + 百分比宽高，自适应任何容器 -->
<svg id="svg"
     viewBox="0 0 1820 740"
     preserveAspectRatio="xMinYMin meet"
     xmlns="http://www.w3.org/2000/svg"
     style="width:100%;height:100%;display:block;overflow:visible;">

<!-- ❌ 错误：写死像素，在窄 iframe 里坐标全错 -->
<svg id="svg" width="1820" height="740">
```

`viewBox` 的高度 = `computeLanePositions()` 的返回值。

---

### 规范 3：所有鼠标/触控事件必须转换为 SVG 坐标系

**绝对禁止**用 `e.clientX/clientY`、`e.offsetX/offsetY` 或 `getBoundingClientRect()` 直接作为 SVG 坐标。必须用以下函数转换：

```javascript
// 将任意鼠标/触控事件转换为 SVG 内部坐标
function toSVGPoint(e) {
  const svg = document.getElementById('svg');
  const pt  = svg.createSVGPoint();
  // 支持 touch 和 mouse
  const src = e.touches ? e.touches[0] : e;
  pt.x = src.clientX;
  pt.y = src.clientY;
  // getScreenCTM().inverse() 正确处理 iframe 缩放、CSS transform、scroll
  return pt.matrixTransform(svg.getScreenCTM().inverse());
}
```

拖拽节点的完整实现：

```javascript
let dragState = null;

svg.addEventListener('mousedown', onDragStart);
svg.addEventListener('touchstart', onDragStart, { passive: false });
document.addEventListener('mousemove', onDragMove);
document.addEventListener('touchmove',  onDragMove, { passive: false });
document.addEventListener('mouseup',   onDragEnd);
document.addEventListener('touchend',  onDragEnd);

function onDragStart(e) {
  const pt   = toSVGPoint(e);
  const node = hitTest(pt.x, pt.y);
  if (!node) return;
  e.preventDefault();
  dragState = { node, dx: pt.x - node.x, dy: pt.y - node.y };
}

function onDragMove(e) {
  if (!dragState) return;
  e.preventDefault();
  const pt = toSVGPoint(e);
  dragState.node.x = pt.x - dragState.dx;
  dragState.node.y = pt.y - dragState.dy;
  render(); // 每次移动都重新渲染节点和所有边
}

function onDragEnd() { dragState = null; }

// 节点命中测试（纯坐标计算，不查 DOM）
function hitTest(px, py) {
  // 倒序遍历，优先命中最上层节点
  for (let i = nodes.length - 1; i >= 0; i--) {
    const n = nodes[i];
    if (px >= n.x && px <= n.x + n.w && py >= n.y && py <= n.y + n.h) {
      return n;
    }
  }
  return null;
}
```

---

### 规范 4：边（箭头）路径必须纯数据驱动，每次 render() 重算

**绝对禁止**在边路径计算中查询任何 DOM 元素的位置。所有端点坐标从节点数据直接计算：

```javascript
// 根据两节点的相对位置自动选择最优锚点
function getEdgePath(edge) {
  const src = nodes.find(n => n.id === edge.from);
  const dst = nodes.find(n => n.id === edge.to);
  if (!src || !dst) return '';

  const sx = src.x + src.w / 2,  sy = src.y + src.h / 2;
  const dx = dst.x + dst.w / 2,  dy = dst.y + dst.h / 2;
  const diffX = dx - sx, diffY = dy - sy;

  let x1, y1, x2, y2;

  if (Math.abs(diffX) >= Math.abs(diffY) * 0.6) {
    // 主方向：水平
    if (diffX >= 0) {
      x1 = src.x + src.w; y1 = sy;   // src 右侧出
      x2 = dst.x;          y2 = dy;   // dst 左侧入
    } else {
      x1 = src.x;          y1 = sy;
      x2 = dst.x + dst.w;  y2 = dy;
    }
    const cp = Math.abs(x2 - x1) * 0.45;
    const cx1 = x1 + (diffX >= 0 ?  cp : -cp), cy1 = y1;
    const cx2 = x2 + (diffX >= 0 ? -cp :  cp), cy2 = y2;
    return `M${x1},${y1} C${cx1},${cy1} ${cx2},${cy2} ${x2},${y2}`;
  } else {
    // 主方向：垂直（跨泳道）
    if (diffY >= 0) {
      x1 = sx; y1 = src.y + src.h;  // src 底部出
      x2 = dx; y2 = dst.y;           // dst 顶部入
    } else {
      x1 = sx; y1 = src.y;
      x2 = dx; y2 = dst.y + dst.h;
    }
    const cp = Math.abs(y2 - y1) * 0.45;
    const cx1 = x1, cy1 = y1 + (diffY >= 0 ?  cp : -cp);
    const cx2 = x2, cy2 = y2 + (diffY >= 0 ? -cp :  cp);
    return `M${x1},${y1} C${cx1},${cy1} ${cx2},${cy2} ${x2},${y2}`;
  }
}
```

---

### 规范 5：arrowhead marker 使用兼容写法

部分平台的 CSP 会过滤 SVG `<defs>` 内容或不支持某些 marker 属性。使用以下经过验证的兼容写法：

```html
<defs>
  <!-- 实线箭头 -->
  <marker id="arrowhead" markerWidth="8" markerHeight="6"
          refX="7" refY="3" orient="auto" markerUnits="strokeWidth">
    <polygon points="0 0, 8 3, 0 6" fill="#64748b"/>
  </marker>
  <!-- 虚线箭头（用于风险关联） -->
  <marker id="arrowhead-dashed" markerWidth="8" markerHeight="6"
          refX="7" refY="3" orient="auto" markerUnits="strokeWidth">
    <polygon points="0 0, 8 3, 0 6" fill="#94a3b8"/>
  </marker>
</defs>
```

边 `<path>` 的引用方式：

```html
<!-- ✅ 正确：marker-end 指向 defs 内的 id -->
<path d="..." stroke="#64748b" stroke-width="1.5"
      fill="none" marker-end="url(#arrowhead)"/>

<!-- 虚线边 -->
<path d="..." stroke="#94a3b8" stroke-width="1.5" stroke-dasharray="6,4"
      fill="none" marker-end="url(#arrowhead-dashed)"/>
```

**注意**：`markerUnits="strokeWidth"` 比 `markerUnits="userSpaceOnUse"` 兼容性更好；用 `<polygon>` 比 `<path>` 渲染更稳定。

---

### 规范 6：render() 函数是唯一的 DOM 写入入口

所有 SVG 元素通过一个统一的 `render()` 函数生成，每次状态变化（拖拽、新增节点、连线）后调用完整 render，不做局部 DOM patch：

```javascript
function render() {
  const svgEl = document.getElementById('svg');
  // 保留 defs（marker）和固定元素，清空并重绘动态内容
  const dynamicLayer = document.getElementById('dynamic-layer');
  dynamicLayer.innerHTML = '';

  // 1. 绘制泳道背景
  lanes.forEach(lane => drawLane(dynamicLayer, lane));

  // 2. 绘制边（在节点之下）
  edges.forEach(edge => drawEdge(dynamicLayer, edge));

  // 3. 绘制节点（在边之上）
  nodes.forEach(node => drawNode(dynamicLayer, node));
}
```

HTML 骨架中，`<defs>` 和 `<g id="dynamic-layer">` 分开：

```html
<svg id="svg" viewBox="0 0 1820 740" ...>
  <defs>
    <!-- marker 定义放这里，render() 不会清除 defs -->
    <marker id="arrowhead" ...>...</marker>
    <marker id="arrowhead-dashed" ...>...</marker>
  </defs>
  <g id="dynamic-layer"></g>
</svg>
```

---

### 规范 7：节点文字换行使用 `<tspan>` 而非依赖 CSS

不同平台字体宽度不一致，不要用 JavaScript 测量文字宽度来决定换行。改用按 `\n` 分割 + `<tspan>` 多行：

```javascript
function renderNodeText(parent, node) {
  const cx = node.x + node.w / 2;
  const cy = node.y + node.h / 2;
  const lines = node.text.split('\n');
  const lineH = 16; // 行高（px，固定值）
  const startY = cy - ((lines.length - 1) * lineH) / 2;

  const t = document.createElementNS('http://www.w3.org/2000/svg', 'text');
  t.setAttribute('text-anchor', 'middle');
  t.setAttribute('dominant-baseline', 'middle');
  t.setAttribute('font-size', '13');
  t.setAttribute('font-family', 'system-ui, -apple-system, sans-serif');
  t.setAttribute('fill', nodeColors(node).text); // 从 THEMES[currentTheme].nodes 取色，禁止硬编码
  t.setAttribute('pointer-events', 'none'); // 防止文字拦截鼠标事件

  lines.forEach((line, i) => {
    const ts = document.createElementNS('http://www.w3.org/2000/svg', 'tspan');
    ts.setAttribute('x', cx);
    ts.setAttribute('y', startY + i * lineH);
    ts.textContent = line;
    t.appendChild(ts);
  });

  parent.appendChild(t);
}
```

---

## 数据结构（JavaScript 初始化格式）

```javascript
// ── 常量 ──────────────────────────────────────
const TOOLBAR_H = 56;
const LBL_W     = 56;
const CANVAS_W  = 1820;

// ── 泳道（h 是泳道高度，startY 由 computeLanePositions() 填入）──
// 注意：泳道不存颜色，渲染时从 THEMES[currentTheme].lanes[lane.id] 取色
let lanes = [
  { id:'user',   name:'用户\nUser',                   h:140, startY:0 },
  { id:'agent',  name:'智能体推理\nAgent Reasoning',   h:240, startY:0 },
  { id:'system', name:'系统调用\nSystem & API',        h:140, startY:0 },
  { id:'risk',   name:'风险与缓解\nRisk & Mitigation', h:170, startY:0 }
];

// ── 节点原始数据（y 是泳道内偏移，initNodes() 会转为绝对坐标）──
const RAW_NODES = [
  { id:'u1', lane:'user',   x:220, y:37,  w:200, h:65,  type:'message', text:'用户输入示例' },
  { id:'a1', lane:'agent',  x:220, y:89,  w:185, h:62,  type:'rect',    text:'解析用户意图' },
  { id:'a2', lane:'agent',  x:460, y:80,  w:160, h:80,  type:'diamond', text:'信息\n是否完整?' },
  { id:'s1', lane:'system', x:460, y:39,  w:185, h:62,  type:'rect',    text:'调用 XX API' },
  { id:'r1', lane:'risk',   x:460, y:50,  w:185, h:62,  type:'note',    text:'风险：数据不准确' },
  { id:'r2', lane:'risk',   x:700, y:50,  w:185, h:62,  type:'rect',    text:'缓解：人工复核' },
];

// ── 边 ────────────────────────────────────────
let edges = [
  { id:'e1', from:'u1', to:'a1', label:'',    dashed:false },
  { id:'e2', from:'a1', to:'a2', label:'',    dashed:false },
  { id:'e3', from:'a2', to:'s1', label:'Yes', dashed:false },
  { id:'e4', from:'r1', to:'r2', label:'',    dashed:true  },
];

// ── 初始化 ────────────────────────────────────
let nodes = [];

window.addEventListener('load', () => {
  const totalH = computeLanePositions();
  // 更新 SVG viewBox 高度
  document.getElementById('svg').setAttribute('viewBox', `0 0 ${CANVAS_W} ${totalH}`);
  // 将节点 y 从泳道相对坐标转为 SVG 绝对坐标
  nodes = initNodes(RAW_NODES);
  render();
});
```

---

## 双主题系统（Light / Dark，必须实现）

生成的 HTML 必须同时内置 light 和 dark 两套主题，并在右上角提供切换按钮。**以下色板的每一对文字/背景组合都已通过 WCAG 2.1 AA 对比度验证（正文 ≥4.5:1，图形元素/描边 ≥3:1），生成时必须原样使用，不得自行替换颜色**。如果场景确实需要调整颜色，必须先计算对比度确认达标。

### THEMES 对象（唯一颜色来源）

```javascript
const THEMES = {
  light: {
    pageBg:'#f8fafc',   canvasBg:'#f8fafc',
    toolbarBg:'#ffffff', toolbarText:'#1e293b', toolbarBorder:'#e2e8f0',
    legendBg:'#ffffff',  legendText:'#1e293b',  legendBorder:'#cbd5e1',
    laneLabelText:'#ffffff',                  // 画在 lane hdr 色条上
    edge:'#475569', edgeDashed:'#64748b', edgeLabel:'#334155',
    selection:'#2563eb',
    lanes: {  // bg = 泳道主体背景，hdr = 左侧标签条背景
      user:   { bg:'#eff6ff', hdr:'#1d4ed8' },
      agent:  { bg:'#f0fdf4', hdr:'#15803d' },
      system: { bg:'#faf5ff', hdr:'#7e22ce' },
      risk:   { bg:'#fff7ed', hdr:'#c2410c' }
    },
    nodes: {  // 按节点角色取色；text 与 fill 的对比度均 ≥8:1
      user:       { fill:'#dbeafe', stroke:'#2563eb', text:'#1e3a8a' },
      agent:      { fill:'#dcfce7', stroke:'#16a34a', text:'#14532d' },
      system:     { fill:'#f3e8ff', stroke:'#9333ea', text:'#581c87' },
      risk:       { fill:'#fee2e2', stroke:'#dc2626', text:'#7f1d1d' },
      mitigation: { fill:'#ffedd5', stroke:'#ea580c', text:'#7c2d12' }
    }
  },
  dark: {
    pageBg:'#0f172a',   canvasBg:'#0f172a',
    toolbarBg:'#1e293b', toolbarText:'#e2e8f0', toolbarBorder:'#334155',
    legendBg:'#1e293b',  legendText:'#e2e8f0',  legendBorder:'#475569',
    laneLabelText:'',                         // dark 下按泳道取 labelText，见 lanes
    edge:'#94a3b8', edgeDashed:'#94a3b8', edgeLabel:'#cbd5e1',
    selection:'#60a5fa',
    lanes: {
      user:   { bg:'#16213e', hdr:'#1e3a8a', labelText:'#dbeafe' },
      agent:  { bg:'#122b1d', hdr:'#14532d', labelText:'#dcfce7' },
      system: { bg:'#231435', hdr:'#581c87', labelText:'#f3e8ff' },
      risk:   { bg:'#2b1216', hdr:'#7c2d12', labelText:'#ffedd5' }
    },
    nodes: {  // dark 下深底浅字，text 与 fill 对比度均 ≥8:1
      user:       { fill:'#1e3a8a', stroke:'#60a5fa', text:'#dbeafe' },
      agent:      { fill:'#14532d', stroke:'#4ade80', text:'#dcfce7' },
      system:     { fill:'#581c87', stroke:'#c084fc', text:'#f3e8ff' },
      risk:       { fill:'#7f1d1d', stroke:'#f87171', text:'#fee2e2' },
      mitigation: { fill:'#7c2d12', stroke:'#fb923c', text:'#ffedd5' }
    }
  }
};
```

取色规则：节点颜色按"角色"而非泳道——`user`/`agent`/`system` 泳道里的普通节点用同名色组；`risk` 泳道里 `note` 型节点用 `nodes.risk`，配对的缓解节点用 `nodes.mitigation`。light 下泳道标签文字统一用 `laneLabelText`（白字画在 hdr 色条上）；dark 下用各泳道自己的 `labelText`。

### 主题初始化与切换

```javascript
let currentTheme = (() => {
  try {
    const s = localStorage.getItem('ajm-theme');
    if (s === 'light' || s === 'dark') return s;
  } catch (e) { /* 沙箱/iframe 禁用 localStorage 时静默降级 */ }
  return (window.matchMedia && matchMedia('(prefers-color-scheme: dark)').matches)
    ? 'dark' : 'light';
})();

function theme() { return THEMES[currentTheme]; }

function setTheme(t) {
  currentTheme = t;
  try { localStorage.setItem('ajm-theme', t); } catch (e) {}
  document.documentElement.setAttribute('data-theme', t);
  const btn = document.getElementById('theme-toggle');
  btn.textContent = (t === 'dark' ? '☀️ Light' : '🌙 Dark');
  btn.setAttribute('aria-label', t === 'dark' ? '切换到浅色主题' : '切换到深色主题');
  syncMarkerColors(); // defs 不在 render() 里重绘，箭头颜色要单独同步
  render();           // 泳道/节点/边全部从 theme() 重新取色
}

// 箭头 marker 颜色跟随主题
function syncMarkerColors() {
  document.querySelector('#arrowhead polygon').setAttribute('fill', theme().edge);
  document.querySelector('#arrowhead-dashed polygon').setAttribute('fill', theme().edgeDashed);
}
```

页面级样式（body、工具栏、图例、弹窗）用 CSS 变量 + `:root[data-theme="..."]` 实现，SVG 内部元素在 `render()` 里从 `theme()` 取色。两条路径必须同步切换，不允许只切一半。

切换按钮 CSS 要求：light 下文字色 `#1e293b`、dark 下 `#e2e8f0`（同 toolbarText）；`:focus-visible` 时显示 2px 实线轮廓（light 用 `#2563eb`，dark 用 `#60a5fa`）。

### 标题渐变色（根据场景选择，仅用于工具栏标题文字）

- 通用：`#3b82f6 → #6c47ff`
- 流程审批类：`#0ea5e9 → #6c47ff`
- 数据/分析类：`#059669 → #0ea5e9`
- 客服/对话类：`#f97316 → #ef4444`
- 创作/生成类：`#8b5cf6 → #ec4899`

**渐变两端颜色必须在两种主题的工具栏背景上都达到 ≥4.5:1 对比度**：light 工具栏为白色，偏亮的一端要压暗（如 `#0ea5e9`→`#0369a1`、`#f97316`→`#c2410c`、`#059669`→`#047857`）；dark 工具栏为 `#1e293b`，偏暗的一端要提亮（如 `#059669`→`#34d399`、`#0ea5e9`→`#38bdf8`、`#6c47ff`→`#a78bfa`）。即两套主题各维护一对渐变色（CSS 变量 `--title-g1/--title-g2`），生成前逐一验算对比度。

---

## 技术实现参考

如果当前 session 中存在以下文件，参考其完整 SVG 引擎实现：
- `agent-journey-map-flow.html`：SVG 引擎核心（拖拽、连线、编辑、导出）
- `agent-journey-map-travel.html`：差旅审批场景参考
- `agent-journey-map-lowcode.html`：双入口起点参考

如果这些文件不在上下文中，按照上述规格从头生成，确保所有交互功能完整，**并严格遵守上方 7 条跨平台兼容性规范**。

---

### 第四步：交付说明

生成完成后，提供文件链接，并告知用户：
1. 点击链接在浏览器打开，支持**拖拽移动节点、双击修改文字**
2. 如需较大结构调整（增减泳道、重设流程），再次使用这个 skill

---

## 概念参考卡

当用户需要先了解 AJM 概念时，生成 `agent-journey-map-concept.html`。

**内容结构**（精美单页 HTML，中文为主，适合分享给团队）：

1. **核心定义**：AJM 是什么，一句话 + 一段解释
2. **与传统流程图的区别**：对比表格（用户旅程图 vs. AJM vs. 普通流程图）
3. **泳道结构说明**：四条泳道各自的职责，配合色块图示
4. **节点类型说明**：各类型节点的含义和使用场景
5. **AJM 的价值**：三点，面向 PM 的语言（设计阶段发现风险 / 对齐工程实现 / 明确能力边界）
6. **开始创建**：引导用户说"帮我创建一个 AJM"

生成完概念参考卡后，询问用户是否准备好开始创建自己的 AJM。

---

## 修改模式

当用户说"帮我修改这个 AJM"或"调整一下之前的 agent journey map"时：

**用户提供了文件名或文件内容：**
1. 读取文件中的 `lanes`、`nodes`、`edges` 变量
2. 简述现有结构，询问想改哪个部分

**用户口头描述修改意图（没有提供文件）：**
1. 询问是对哪个场景的文件
2. 确认修改意图后，生成新文件（文件名加 `-v2` 后缀，不覆盖原文件）

---

## 注意事项

- **不覆盖原文件**：始终生成新文件，通过文件名或版本号区分
- **中文优先**：节点文字、标题、图例使用中文；泳道名称中英双语（用 `\n` 分隔）
- **忠实于用户信息**：不要编造用户没有提到的节点或 API，不确定的地方节点里注明"待补充"
- **完整可运行**：生成的 HTML 必须是单文件，可在浏览器直接打开，所有交互正常
- **严格遵守跨平台兼容性规范**：7 条规范是硬性要求，任何一条违反都可能导致在某些平台上连线错位或箭头消失
- **双主题是硬性要求**：必须内置 light/dark 两套主题 + 右上角切换按钮；颜色只能来自 THEMES 对象（该色板已通过 WCAG 2.1 AA 验证），渲染代码里不得出现绕过 THEMES 的硬编码颜色
