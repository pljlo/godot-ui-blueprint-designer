# Godot UI Blueprint Designer

一个轻量的浏览器工具，用来快速规划 Godot 4 UI 布局，并把草图带回 AI 工作流或 Godot 项目。

A lightweight browser-based tool for sketching Godot 4 UI layouts and exporting them to an AI workflow or a Godot project.

## 在线使用 / Live demo

启用 GitHub Pages 后，可直接通过仓库对应的 Pages 地址在线使用。

## 功能 / Features

- 通过拖放方式绘制、选择和移动 Godot Control 节点。
- 使用 Godot 风格的控件面板和节点层级信息规划场景。
- 调整画布尺寸、缩放、网格和吸附。
- 创建多个界面，编辑节点属性、父子关系和备注。
- 撤销和重做编辑操作。
- 导入、导出可继续编辑的项目 JSON。
- 导出 Markdown 设计稿，以及 Godot `.tscn` 场景文本。
- 配套 AI 指南，帮助 AI 在保留节点 ID、层级和用户备注的前提下修改项目 JSON。

## 快速开始 / Quick start

1. 下载或克隆本仓库。
2. 在现代浏览器中打开根目录的 `index.html`。
3. 在画布中编辑布局，并通过“保存项目 JSON”或导出按钮保存结果。

无需安装依赖或启动服务器。GitHub Pages 启用后，也可以直接在线打开 `index.html`。

## 数据与隐私 / Data and privacy

设计器在浏览器本地运行，并将工作状态保存在浏览器的 `localStorage` 中。工具不会自动把设计数据上传到服务器。导出的 JSON、Markdown 和 `.tscn` 文件由用户主动保存。

## 项目结构 / Project layout

```text
index.html       独立网页设计器
README.md        项目介绍与使用说明
LICENSE          MIT License
docs/AI_GUIDE.md    供 AI 使用的项目 JSON 编辑指南
```

## 许可证 / License

本项目按 [MIT License](LICENSE) 发布。你可以使用、复制、修改、合并、发布、分发、再许可及销售本软件的副本；分发时需保留版权声明和许可证文本。软件按“现状”提供，不附带明示或默示担保，作者或版权持有人不对使用造成的索赔或损失承担责任。

这份许可证覆盖本仓库中的项目代码和文档。Godot 名称及商标仍归其各自权利人所有；如未来加入第三方素材，应为该素材附上适用的来源和许可证说明。
