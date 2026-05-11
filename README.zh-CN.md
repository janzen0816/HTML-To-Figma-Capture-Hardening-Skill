# HTML To Figma Capture Hardening Skill

[English](README.md) | 简体中文

一个用于提高 HTML/CSS 导入 Figma 成功率和视觉还原度的 Codex Skill。

当你把网页、展板、长图、作品集页面或静态 HTML 导入 Figma 时，如果遇到图标丢失、CSS 网格背景变形、渐变/伪元素装饰不见、SVG sprite 无法解析、图标字体变成空白方块等问题，这个 skill 会引导 Codex 先创建一个“加固版”临时 HTML，再用这个加固版执行 Figma HTML-to-design 捕获。

核心目标很简单：不要直接捕获脆弱的浏览器效果，而是把重要视觉元素变成 Figma 更容易理解的显式 DOM、SVG 或图片节点。

## 适合解决的问题

- Figma 导入 HTML 后，Phosphor、Font Awesome、Material Icons、Lucide 等图标不显示。
- 页面使用 `<svg><use></use></svg>` 或外部 SVG sprite，导入后图标丢失。
- 图标来自 CDN CSS、icon font、伪元素 `content`，Figma 捕获后变成空白。
- CSS `::before` / `::after` 做的装饰块、角标、箭头、标记在 Figma 中消失。
- CSS `linear-gradient`、`radial-gradient`、`repeating-linear-gradient` 做的网格、点阵、条纹背景被拉伸或丢失。
- 展板、海报、报告页、长页面导入时，背景纹理和几何图案不稳定。
- 你需要保护原始 HTML，不希望为了 Figma 导入把源文件改乱。

英文搜索关键词：HTML to Figma, Figma HTML import, Figma capture, missing icons, icon font, SVG sprite, CSS pseudo-elements, CSS gradient background, design import hardening.

## 它怎么工作

这个 skill 的工作流不是“直接导入原始页面”，而是：

1. 创建一个临时副本，例如 `example-figma-hardened.html`。
2. 扫描 HTML 和 CSS，识别高风险视觉来源。
3. 把重要图标、CSS 装饰和背景图案转换成更稳定的显式节点。
4. 只在加固副本中注入 Figma capture 脚本。
5. 用本地 HTTP 服务打开加固副本并验证视觉效果。
6. 捕获加固副本到 Figma，并轮询直到新文件生成完成。

原始 HTML 默认保持不变。所有为了 Figma 捕获做的修改，都应该只发生在 `*-figma-hardened.html` 这类临时文件里。

## 使用方式

把这个 skill 放到 Codex skills 目录后，在需要导入 HTML 到 Figma 时，可以这样对 Codex 说：

```text
使用 html-to-figma-capture-hardening skill，把这个 HTML 导入 Figma，新建一个 Figma 文件。
```

也可以更具体一点：

```text
根据这个 skill 的规则，先创建 hardened HTML，处理图标和 CSS 背景，再捕获到 Figma。
```

如果你的文件夹里有多个 HTML，建议说明入口文件：

```text
用 html-to-figma-capture-hardening，把 index.html 加固后导入 Figma。
```

## 推荐目录结构

作为 Codex Skill 发布时，建议使用标准结构：

```text
html-to-figma-capture-hardening/
  SKILL.md
  README.md
  README.zh-CN.md
```

当前 skill 的正文可以放在 `SKILL.md` 中，README 面向人类读者，负责介绍用途、场景和使用方法。

## 加固策略概览

### 1. 图标来源

这个 skill 会要求先分类图标来源，而不是假设只有一种图标库。

常见高风险来源包括：

- icon font，例如 `<i class="ph ph-plant"></i>`；
- CDN 图标 CSS；
- SVG sprite 和 `<use>`；
- 伪元素图标，例如 `.icon::before { content: ... }`；
- CSS 画出来的图标。

加固方式通常是使用官方 SVG、内联 SVG、或把外部 SVG 展开为真实节点。

### 2. CSS 生成的装饰图形

很多网页会用 `::before`、`::after`、渐变、边框、box-shadow、mask 或 clip-path 画装饰元素。浏览器看起来没问题，但 Figma 捕获时可能无法稳定还原。

这个 skill 会把重要装饰转成明确的 SVG 或 DOM 节点，并禁用原来的伪元素，避免重复显示。

### 3. CSS 生成的背景

网格、点阵、条纹和重复渐变是 Figma 捕获中很容易出问题的一类。这个 skill 推荐把它们转换为 SVG 背景层，尤其是几何网格。

这样可以避免固定 `viewBox` 被拉伸，导致方格变成长方格。

## 捕获前检查

在真正导入 Figma 前，skill 会要求检查加固副本中还剩下哪些风险项，例如：

```bash
grep -n '<use\|::before\|::after\|content:\|linear-gradient\|radial-gradient\|repeating-\|background-size\|font-family\|mask\|clip-path' hardened.html
grep -n '<svg\|<img\|<picture' hardened.html
```

残留匹配不一定是错误。关键是要解释它们为什么安全，或者为什么不影响主要视觉还原。

如果重要图标仍然依赖 icon font、重要背景仍然只依赖 CSS 渐变、或者本地预览已经发现网格变形，就应该先修复，再捕获。

## 注意事项

- 不要直接捕获原始 HTML。
- 不要把 Figma capture 脚本注入原始文件。
- 不要为了导入 Figma 直接改坏源页面。
- 本地预览应该通过 HTTP 服务打开，不要使用 `file://`。
- 对运行时生成的 SVG 或背景层，捕获时应设置足够的 `figmadelay`。
- Figma capture 会把渲染后的页面数据发送到 Figma 服务，处理敏感页面前请先确认是否可以上传。
- 捕获完成后，可以保留加固副本用于复现，也可以在不需要时删除。

## 一个典型提示词

```text
请使用 html-to-figma-capture-hardening skill：
1. 不要修改原始 HTML。
2. 创建 *-figma-hardened.html 副本。
3. 检查图标、SVG sprite、伪元素、CSS 渐变背景和网格。
4. 把关键视觉元素改成 capture-safe 的 SVG/DOM 节点。
5. 本地 HTTP 预览验证后，新建 Figma 文件并导入。
6. 告诉我 hardened 文件路径和 Figma 链接。
```

## 为什么这个 skill 有用

Figma 的 HTML-to-design 捕获很适合快速把网页变成可编辑设计稿，但它面对浏览器专属渲染技巧时并不总是可靠。这个 skill 把“导入失败后再修补”的流程前置，先做视觉来源盘点和加固，再捕获。

它特别适合这些场景：

- 从 AI 生成的 HTML 导入 Figma；
- 把毕业设计展板、作品集、报告页面导入 Figma；
- 把网页视觉稿交给设计师二次编辑；
- 修复“浏览器里正常，Figma 里缺东西”的导出问题；
- 建立可重复、可解释、不会污染源文件的 HTML-to-Figma 工作流。

## License

如果你开源这个 skill，可以按你的项目偏好添加许可证。常见选择包括 MIT、Apache-2.0 或 CC BY 4.0。
