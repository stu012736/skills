---
name: artifacts-builder
description: 使用现代前端Web技术（React、Tailwind CSS、shadcn/ui）创建复杂多组件claude.ai HTML工件的工具套件。适用于需要状态管理、路由或shadcn/ui组件的复杂工件，不适用于简单的单文件HTML/JSX工件。
license: 完整条款见LICENSE.txt
---

# 工件构建器

要构建强大的前端claude.ai工件，请按照以下步骤操作：
1. 使用 `scripts/init-artifact.sh` 初始化前端仓库
2. 通过编辑生成的代码来开发您的工件
3. 使用 `scripts/bundle-artifact.sh` 将所有代码打包成单个HTML文件
4. 向用户展示工件
5. （可选）测试工件

**技术栈**：React 18 + TypeScript + Vite + Parcel（打包）+ Tailwind CSS + shadcn/ui

## 设计与样式指南

非常重要：为避免通常被称为"AI垃圾"的设计，请避免使用过多的居中布局、紫色渐变、统一圆角和Inter字体。

## 快速开始

### 步骤1：初始化项目

运行初始化脚本创建新的React项目：
```bash
bash scripts/init-artifact.sh <项目名称>
cd <项目名称>
```

这将创建一个完全配置的项目，包含：
- ✅ React + TypeScript（通过Vite）
- ✅ Tailwind CSS 3.4.1 与 shadcn/ui 主题系统
- ✅ 路径别名（`@/`）配置
- ✅ 40+ 个 shadcn/ui 组件预安装
- ✅ 所有 Radix UI 依赖项包含
- ✅ Parcel 配置用于打包（通过 .parcelrc）
- ✅ Node 18+ 兼容性（自动检测并固定Vite版本）

### 步骤2：开发您的工件

要构建工件，请编辑生成的文件。请参阅下面的**常见开发任务**获取指导。

### 步骤3：打包成单个HTML文件

要将React应用打包成单个HTML工件：
```bash
bash scripts/bundle-artifact.sh
```

这将创建 `bundle.html` - 一个自包含的工件，所有JavaScript、CSS和依赖项都已内联。此文件可以直接在Claude对话中作为工件共享。

**要求**：您的项目必须在根目录中有 `index.html` 文件。

**脚本功能**：
- 安装打包依赖项（parcel、@parcel/config-default、parcel-resolver-tspaths、html-inline）
- 创建支持路径别名的 `.parcelrc` 配置
- 使用Parcel构建（无源映射）
- 使用html-inline将所有资源内联到单个HTML中

### 步骤4：与用户共享工件

最后，在对话中与用户共享打包的HTML文件，以便他们可以将其视为工件查看。

### 步骤5：测试/可视化工件（可选）

注意：这是一个完全可选的步骤。仅在必要时或应要求执行。

要测试/可视化工件，请使用可用工具（包括其他技能或内置工具如Playwright或Puppeteer）。通常，避免预先测试工件，因为这会在请求和完成工件展示之间增加延迟。如果出现问题或应要求，在展示工件后进行测试。

## 参考

- **shadcn/ui 组件**：https://ui.shadcn.com/docs/components