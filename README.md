# Netron Editor

> Fork of [lutzroeder/netron](https://github.com/lutzroeder/netron) — 在原版 Netron 查看器基础上新增**图布局手动编辑**能力。

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)

## ✨ 新增功能

原版 Netron 是一个强大的只读神经网络模型可视化器。本 fork 在不修改任何模型解析逻辑的前提下，扩展了图的交互编辑能力，方便用户手动调整自动布局的结果。

### 🔀 双模式切换

右下角工具栏新增模式切换按钮，在**画布拖拽**（默认查看模式）和**节点拖拽**（编辑模式）之间自由切换。

| 模式 | 鼠标拖拽行为 | 适用场景 |
|------|------------|---------|
| Canvas Drag | 平移画布 | 浏览模型 |
| Node Drag | 移动节点 / 框选 | 调整布局 |

### 🖱️ 节点拖拽

在 Node Drag 模式下：

- **拖拽节点** → 实时移动位置，关联的边自动跟随
- **Shift + 点击** → 多选 / 取消选择
- **拖拽选中组** → 同时移动所有选中的节点和控制点

### ⬜ 框选

在空白区域拖拽绘制选区矩形，框内的节点和边控制点自动加入选中集合。支持 Shift 追加选中。

### 🔗 边路径编辑

进入 Node Drag 模式后，自动在边的中间控制点上显示蓝色圆点：

- **拖拽控制点** → 改变边的路径走向
- **悬停在边上** → 显示半透明预览点，**点击即插入**新控制点
- **拖拽控制点靠近另一控制点** → 出现红色 ✕ 删除指示器，**松开即删除**

### ↻ 布局重置

点击工具栏重置按钮，一键恢复所有节点和边到 Dagre 自动布局的初始位置。

---

## 🚀 快速开始

```bash
# 克隆仓库
git clone https://github.com/JoanYu/netron.git
cd netron

# 安装依赖
npm install

# 启动开发模式
npx electron .
```

## 📋 修改范围

本 fork 仅修改了 2 个文件，保持与上游的最大兼容性：

| 文件 | 修改内容 |
|------|---------|
| `source/index.html` | +50 行：右下角工具栏 HTML + CSS 样式 |
| `source/view.js` | +710 行：拖拽模式、节点移动、框选、控制点编辑、布局重置 |

## 📄 License

本项目基于 [MIT License](LICENSE) 发布。原始项目 Netron 版权归 Lutz Roeder 所有。
