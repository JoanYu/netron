# Netron 修改功能分析

> 比较 `netron-ori/source` 与 `netron-main/source` 的差异，反推新增功能。

## 修改概览

| 文件 | 变化类型 | 变化规模 |
|------|----------|----------|
| `index.html` | 新增 HTML/CSS | +50 行（新工具栏 + 样式） |
| `view.js` | 新增 JS 逻辑 | +710 行（交互功能） |

其余 168 个文件完全没有变化。

---

## 新增功能一：拖拽模式切换（Canvas Drag / Node Drag）

### 功能描述

在原版 Netron 中，鼠标拖拽画布只会进行**平移/滚动**操作。修改后新增了**双模式切换**机制：

- **Canvas Drag 模式**（默认）：拖拽画布 → 平移视图（保持原有行为）
- **Node Drag 模式**：拖拽节点 → 移动节点位置，拖拽空白区域 → 框选

### UI 变化

在 `index.html` 右下角新增了一组工具栏按钮 (`toolbar-right`)：

| 按钮 | ID | 功能 |
|------|----|------|
| ✥ 方向箭头图标 | `canvas-drag-button` | 切换到画布拖拽模式 |
| ☐ 矩形+箭头图标 | `node-drag-button` | 切换到节点拖拽模式 |
| ↻ 重置图标 | `reset-layout-button` | 重置所有布局到初始状态 |

### CSS 新增

- `.toolbar-right` — 右下角工具栏定位（`position: absolute; bottom: 10px; right: 10px`）
- `.toolbar-button.active` — 当前激活按钮的高亮样式（亮色/暗色模式均有适配）
- 在 `welcome`、`alert`、`about` 页面状态下隐藏此工具栏

### 代码实现

```diff
// view.js - view.View 类新增

+ _setDragMode(mode) {
+     // 切换 canvas/node 模式
+     // 更新按钮 active 状态
+     // 显示/隐藏边控制点
+ }
```

---

## 新增功能二：节点拖拽移动

### 功能描述

在 **Node Drag 模式** 下，用户可以：

1. **单击节点** → 选中节点（以视觉效果标注）
2. **拖拽节点** → 实时移动该节点的位置
3. **Shift + 点击** → 多选/取消选择（追加/移除选中集合）
4. **拖拽选中组** → 同时移动所有选中的节点和控制点

### 视觉反馈

- 选中的节点：`opacity: 0.7`，`filter: brightness(0.85) saturate(1.3)`
- 拖拽中：鼠标变为 `grabbing`

### 关键方法

| 方法 | 功能 |
|------|------|
| `_nodePointerDownHandler(e, nodeElement)` | 节点拖拽的核心逻辑：查找目标节点 → 处理选中 → 注册 move/up 事件 |
| `_clearDragSelection()` | 清空所有已选节点和控制点的视觉状态 |
| `_updateDragSelectionVisuals()` | 根据选中集合更新所有节点/控制点的视觉样式 |
| `_ensureOriginalsSaved()` | 首次操作时快照所有节点和边的原始坐标 |

---

## 新增功能三：框选（Box Selection）

### 功能描述

在 **Node Drag 模式** 下点击空白区域并拖拽，绘制一个**半透明蓝色矩形框选区域**，松开后框内的所有节点和控制点被自动选中。

### 视觉效果

```css
fill: rgba(74, 144, 217, 0.15);
stroke: rgba(74, 144, 217, 0.6);
stroke-dasharray: 4 2;  /* 虚线边框 */
```

### 支持功能

- **Shift + 框选**：追加选中（不清除已选项）
- **普通框选**：先清除再选中
- 同时选中框内的**节点**和**边控制点**

### 关键方法

| 方法 | 功能 |
|------|------|
| `_boxSelectPointerDown(e)` | 创建 SVG rect → 监听 pointermove 更新尺寸 → pointerup 时检测碰撞 |

---

## 新增功能四：边控制点编辑

### 功能描述

在 **Node Drag 模式** 下，所有边的中间控制点（不含起止端点）会以**蓝色圆点**形式显示在边路径上。用户可以：

1. **拖拽控制点** → 改变边的形状/路径
2. **Shift + 点击控制点** → 加入多选集
3. **悬停在边上** → 显示半透明"幽灵点" (ghost point)
4. **点击幽灵点** → 在该位置插入新的控制点
5. **拖拽控制点靠近另一个控制点** → 显示红色删除指示器 (×)，松开即删除

### 控制点样式

```css
/* 普通状态 */
fill: #4a90d9; stroke: #ffffff; r: 5; opacity: 0.8;
/* 选中状态 */
fill: #2c6fad; r: 7;
/* 删除提示 */
fill: #e74c3c;  /* 红色 */
```

### 幽灵点 (Ghost Point)

- 在边的线段附近悬停时出现半透明蓝点
- 通过计算鼠标到各线段的投影距离，找到最近的线段
- 点击后在该位置 `splice` 插入一个新控制点

### 删除指示器

- 当拖拽的控制点距离另一个控制点 < 12px 时显示
- 红色圆形背景 + 白色 × 号
- 松开后执行 `points.splice(pointIndex, 1)` 删除

### 关键方法

| 方法 | 功能 |
|------|------|
| `_showEdgeControlPoints()` | 遍历所有边创建控制点圆形 SVG 元素 |
| `_hideEdgeControlPoints()` | 移除所有控制点 DOM 元素 |
| `_createControlPointsForEdge(edge)` | 为单条边创建控制点 |
| `_refreshEdgeControlPoints()` | 销毁并重建所有控制点 |
| `_updateEdgeControlPointPositions(edge)` | 拖拽中实时更新控制点圆的坐标 |
| `_controlPointPointerDown(e, circle, edge, pointIndex)` | 控制点拖拽逻辑（含合并删除） |
| `_setupEdgeHoverGhosts()` | 设置幽灵点的 hover 事件 |
| `_onEdgeHover(e)` | 计算最近线段投影 → 显示幽灵点 |
| `_onGhostClick(e)` | 在幽灵点位置插入新控制点 |
| `_getOrCreateDeleteIndicator()` | 创建红色 × 删除指示器 |

---

## 新增功能五：布局重置

### 功能描述

点击工具栏的 **重置按钮** (↻)，将所有被拖拽移动过的节点和控制点恢复到 Dagre 布局引擎计算的初始位置。

### 实现原理

1. 在**首次拖拽操作**时，`_ensureOriginalsSaved()` 会快照所有节点坐标到 `_originalPositions` Map，所有边的 points 到 `_originalEdgePoints` Map
2. 重置时从这两个 Map 恢复所有坐标
3. 重新渲染所有边
4. 如果当前在 Node Drag 模式，重新显示控制点

### 关键方法

| 方法 | 功能 |
|------|------|
| `_resetLayout()` | 恢复节点坐标 → 恢复边控制点 → 重渲染边 → 清空快照 |
| `_ensureOriginalsSaved()` | 保存原始坐标的懒初始化 |

---

## 新增状态属性汇总

以下是在 `grapher.Graph` 构造函数中新增的状态属性：

```javascript
this._dragMode = 'canvas';          // 当前拖拽模式
this._originalPositions = null;      // 节点原始坐标快照 Map<key, {x, y}>
this._dragSelectedNodes = new Set(); // 当前选中的节点 key 集合
this._dragSelectedCPs = [];          // 当前选中的控制点数组
```

---

## 功能总结

你之前想在 Netron 中添加的核心功能是 **图编辑/图手动调整能力**：

> **目标：让用户可以手动调整神经网络计算图的布局**，而不仅仅依赖 Dagre 自动布局。

具体包括：

1. ✅ **双模式切换** — 区分"查看"和"编辑"场景
2. ✅ **节点拖拽** — 手动移动节点位置
3. ✅ **框选** — 批量选中多个元素
4. ✅ **边路径编辑** — 通过控制点精确调整边的走向
5. ✅ **控制点增/删** — 动态添加和删除边的拐点
6. ✅ **布局重置** — 安全回退到自动布局结果
7. ✅ **画布边界扩展** — 解决小图缩放时的平移溢出痛点
8. ✅ **文本批注功能** — 支持给边文本（Shape信息）进行移动，以及添加全图独立漂浮的批注文字，以方便导出SVG

---

## 新增功能六：画布边界控制与自由文本批注

### 功能描述

- **编辑画布边界 (Edit Canvas Bounds)**：在右侧工具栏新增按钮，允许用户手动指定额外的外边距（Padding），方便对极小图进行安全缩放和平移操作。
- **自定义文本批注 (Add Text Annotation)**：在右侧工具栏新增按钮，可在视图中心生成一段自定义文本，用户可随意拖拽，并且该文本在SVG导出时可用，不随源ONNX文件保存。
- **Shape 文本移动**：针对线上展示节点间的 `edge-label`，现已支持在 Node Drag 模式下选中并自由拖动，拖动信息同样隔离于源文件。

### 关键更改
- 在 `view.js` 的 `restore` 阶段注入了 `this._canvasMargin` 控制 `SVG` 画布边界。
- 扩充了 `_nodePointerDownHandler` 及 `_boxSelectPointerDown` 识别与选中目标，使其支持包含 `.edge-label` 与新增的 `.custom-text` 类节点元素。
- 在 `grapher.js` 的 `Edge.update` 中读取可能经过修改的 `label.x` 和 `label.y` 进而应用 Transform 位移效果。
