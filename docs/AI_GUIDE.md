# 给 AI 的使用说明：Godot UI Blueprint Designer

> 目标：你不是直接替用户“画死一个最终 UI”，而是帮助用户生成或修改 **Godot UI Blueprint Designer 可重新导入编辑的 JSON 项目**。  
> 用户会把你的 JSON 导回网页设计器中继续拖动、改尺寸、改父子关系，最后再从设计器导出 Markdown 或 `.tscn` 给 Godot。

---

## 1. 你应该优先输出什么

如果用户的目标是：

- “AI 帮我画一版，然后我自己继续调整”
- “帮我重新布局这个 UI”
- “根据截图/需求做一个 Godot UI 草图”
- “修改我现有的 UI Blueprint 项目”

那么**优先输出本工具兼容的项目 JSON**。

### 不同格式的用途

- **项目 JSON**：用于重新导入本网页设计器继续编辑，优先使用。
- **Markdown**：用于给 Codex / Godot MCP 理解设计规范，不是主要回编辑格式。
- **`.tscn`**：用于最终放进 Godot，不是主要回编辑格式。

推荐工作流：

```text
用户需求 / 截图 / 旧 JSON
        ↓
AI 生成或修改项目 JSON
        ↓
用户导入 Godot UI Blueprint Designer
        ↓
用户继续拖动、微调、补备注
        ↓
设计器导出 Markdown 或 .tscn
        ↓
Codex / MCP / Godot
```

---

# 2. 当用户给你“已有项目 JSON”时

这是最重要的模式。

请：

1. **在原 JSON 基础上修改，不要重新发明另一套结构。**
2. 尽量保留已有：
   - `screen.id`
   - `node.id`
   - `parentId`
   - 节点名称
   - 用户备注
   - Codex / MCP 备注
   - 未知的未来扩展字段
3. 只有新建节点时才生成新的唯一 `id`。
4. 删除节点时，也要处理所有引用它的 `parentId`。
5. 不要把用户临时上传的背景参考图写进项目 JSON。
6. 输出前检查 JSON 可以被正常 `JSON.parse()`。

如果用户只要求修改布局，不要顺便重命名节点或改节点类型，除非确实有必要。

---

# 3. 项目 JSON 基本结构

兼容格式如下：

```json
{
  "version": 2,
  "projectName": "Godot UI Project",
  "width": 1152,
  "height": 648,
  "zoom": 0.8,
  "grid": true,
  "snap": true,
  "gridSize": 8,
  "activeScreenId": "screen_main",
  "selectedTool": "select",
  "lastDrawType": "Button",
  "continuousDraw": false,
  "selectedNodeId": null,
  "activeSidebarTab": "inspector",
  "looseLayout": true,
  "screens": [
    {
      "id": "screen_main",
      "name": "MainMenu",
      "nodes": []
    }
  ]
}
```

### 字段解释

- `version`：保持为 `2`。
- `projectName`：项目名称。
- `width` / `height`：设计基准分辨率。
- `zoom`：编辑器视图缩放，通常无需关注。
- `grid`：是否显示网格。
- `snap`：是否启用网格吸附。
- `gridSize`：吸附步长，常用 8 / 16。
- `activeScreenId`：当前打开的场景 ID。
- `selectedTool`：输出时建议 `"select"`。
- `lastDrawType`：上次使用的控件类型，可以保留。
- `continuousDraw`：是否连续绘制，通常可保留。
- `selectedNodeId`：输出时建议 `null`。
- `activeSidebarTab`：一般为 `"inspector"`。
- `looseLayout`：
  - `true` = 快速草图，不要求精确照抄像素。
  - `false` = 尺寸应尽量严格执行。
- `screens`：多个 UI 场景/窗口。

---

# 4. 节点结构

每个 `screen.nodes` 内的节点使用：

```json
{
  "id": "node_start_button",
  "type": "Button",
  "name": "StartGameBtn",
  "x": 440,
  "y": 280,
  "w": 272,
  "h": 52,
  "text": "开始游戏",
  "note": "主菜单的主要 CTA 按钮",
  "mcpNote": "连接 pressed 到 _on_start_game_pressed",
  "parentId": "node_menu_vbox",
  "z": 10,
  "anchor": "custom",
  "color": "#dfe7ef"
}
```

### 各字段含义

- `id`
  - 设计器内部唯一 ID。
  - 不是 Godot 节点名。
  - 在同一个 JSON 中必须唯一。

- `type`
  - Godot Control 节点类型。
  - 例如 `Button`、`Label`、`VBoxContainer`。

- `name`
  - **Godot Scene Tree 中的节点名称。**
  - 例如：`StartGameBtn`。
  - 不是玩家看到的文字。

- `x`, `y`, `w`, `h`
  - 当前设计器中的绝对画布坐标。
  - 即使有父节点，仍然保存绝对坐标。
  - 设计器会自己计算相对父节点位置。

- `text`
  - 玩家实际看到的文字。
  - 例如 Button 上的“开始游戏”。

- `note`
  - 视觉、交互、用途说明。
  - 例如：“右上角关闭按钮”“弱化显示”。

- `mcpNote`
  - 给 Codex / Godot MCP 的实现提示。
  - 例如信号连接、动态数据绑定、运行时行为。

- `parentId`
  - 真正的父子层级。
  - `null` 表示直接挂在场景根 `Control` 下。

- `z`
  - 编辑器中的大致绘制层级。
  - 数字越大通常越靠前。

- `anchor`
  - 布局意图。
  - 支持：
    - `custom`
    - `full_rect`
    - `center`
    - `top_wide`
    - `bottom_wide`
    - `top_left`
    - `center_left`
    - `center_right`

- `color`
  - 原型预览颜色。
  - 一般使用 `#RRGGBB`。

---

# 5. 父子关系是核心，不要只靠坐标表示“在里面”

如果一个节点视觉上属于另一个节点，应该用 `parentId` 表示。

例如：

```text
MenuPanel
└── MarginContainer
    └── VBoxContainer
        ├── StartGameBtn
        ├── SettingsBtn
        └── ExitBtn
```

JSON 应类似：

```json
[
  {
    "id": "panel",
    "type": "PanelContainer",
    "name": "MenuPanel",
    "parentId": null
  },
  {
    "id": "margin",
    "type": "MarginContainer",
    "name": "MenuMargin",
    "parentId": "panel"
  },
  {
    "id": "vbox",
    "type": "VBoxContainer",
    "name": "MenuButtonsVBox",
    "parentId": "margin"
  },
  {
    "id": "start",
    "type": "Button",
    "name": "StartGameBtn",
    "parentId": "vbox"
  }
]
```

不要把所有节点全部挂在根节点，然后只用坐标做出“看起来像嵌套”。

---

# 6. Godot 结构规则

生成 JSON 时，请遵守这些规则。

## 6.1 抽象基类不要作为最终可实例化节点

下面这些在本工具中被视为抽象/基类：

- `BaseButton`
- `Range`
- `Separator`

优先使用具体子类，例如：

- `Button`
- `CheckBox`
- `CheckButton`
- `HSlider`
- `VSlider`
- `ProgressBar`
- `HSeparator`
- `VSeparator`

---

## 6.2 Graph 节点

- `GraphElement`
- `GraphFrame`
- `GraphNode`

应该作为 `GraphEdit` 的直接子节点。

不要把普通 `Button`、`Label` 等直接作为 `GraphEdit` 的图元素。

---

## 6.3 SplitContainer

- `HSplitContainer`
- `VSplitContainer`
- `SplitContainer`

最多安排 **2 个直接子 Control**。

---

## 6.4 SubViewportContainer

不要把普通 Control 节点直接塞进 `SubViewportContainer` 当普通布局容器。

它的用途是承载 `SubViewport` 内容。

---

## 6.5 Container 会自动布局

下面这类节点会自动控制子节点布局：

- `VBoxContainer`
- `HBoxContainer`
- `GridContainer`
- `MarginContainer`
- `CenterContainer`
- `FlowContainer`
- `HFlowContainer`
- `VFlowContainer`
- `PanelContainer`
- `ScrollContainer`
- `TabContainer`
- `SplitContainer`
- 等

因此：

- 父子结构比精确像素坐标更重要。
- 不要为了匹配草图而制造明显违反 Container 工作方式的布局。
- 最终 Godot 中可以通过：
  - `size_flags`
  - `custom_minimum_size`
  - Theme constants
  - separation
  - margins
  - anchors
  来实现正确视觉结果。

---

# 7. 支持的常用控件类型

当前工具主要支持以下 Control 类型：

```text
Control
ColorRect
Panel
NinePatchRect
TextureRect
ReferenceRect

Container
AspectRatioContainer
BoxContainer
VBoxContainer
HBoxContainer
CenterContainer
FlowContainer
HFlowContainer
VFlowContainer
FoldableContainer
GridContainer
SplitContainer
HSplitContainer
VSplitContainer
MarginContainer
PanelContainer
ScrollContainer
SubViewportContainer
TabContainer

Label
RichTextLabel
LineEdit
TextEdit
CodeEdit

Button
CheckButton
CheckBox
LinkButton
MenuButton
OptionButton
TextureButton
BaseButton

ItemList
Tree
TabBar
MenuBar
GraphEdit
GraphElement
GraphFrame
GraphNode

Range
HSlider
VSlider
HScrollBar
VScrollBar
ProgressBar
SpinBox
Separator
HSeparator
VSeparator

VideoStreamPlayer
VirtualJoystick
```

尽量从这份列表里选择节点类型。

---

# 8. `looseLayout = true` 时怎么理解

这是用户的 **“我只快速画个大概，AI 帮我把实际尺寸做好”** 模式。

当：

```json
"looseLayout": true
```

时：

## 你应该保留

- 场景结构
- 父子关系
- 节点用途
- 控件类型
- 大致相对位置
- 主要视觉层级
- 用户写的备注
- 功能关系

## 你可以优化

- `x`
- `y`
- `w`
- `h`
- 节点间距
- Margin
- Container separation
- Anchor
- Size Flags
- 字号适配
- 按钮实际高度
- 面板实际大小

### 优先级

1. 如果用户现有项目、项目 MD、Theme 或设计规范已经规定尺寸，**优先遵循已有项目**。
2. 没有明确规范时，结合 Godot 的 Container / Anchor / Size Flags 机制自行给出更合理尺寸。
3. 不要机械照抄草图中明显不合理的像素值。

---

# 9. `looseLayout = false` 时

当：

```json
"looseLayout": false
```

时，应尽量把设计器中的：

- 坐标
- 宽高
- Anchor
- 父子层级

视为明确规格。

可以修正明显错误，但不要在没有说明的情况下大幅改变布局。

---

# 10. 图片背景不是项目数据

用户可以在网页设计器里：

- 上传截图
- 拖入截图
- 粘贴截图
- 拖动背景图
- 缩放背景图

这些图片只是帮助用户临摹和摆 UI。

**不要在项目 JSON 中添加任何背景图片数据。**

不要添加：

```text
background
backgroundImage
imageData
base64
assetKey
referenceImage
```

之类字段来保存用户截图。

如果用户把截图同时提供给你，你可以利用截图设计 UI，但最终只输出 UI 节点 JSON。

---

# 11. 从截图帮用户“画一版”时

如果用户只给截图，没有旧 JSON：

1. 判断画布比例。
2. 识别主要区域。
3. 用合理的 Control / Container 拆层级。
4. 给每个重要元素清晰的英文/程序友好节点名。
5. 文本放在 `text` 中。
6. 视觉说明放在 `note`。
7. 实现逻辑放在 `mcpNote`。
8. 根据视觉包含关系设置 `parentId`。
9. 如果只是大概还原，建议：
   ```json
   "looseLayout": true
   ```
10. 不要为了还原截图而创建几十个毫无语义的 `Control_1`、`Control_2`。

优先生成**可维护的 Godot UI 结构**，而不仅仅是像素拼图。

---

# 12. 推荐的节点命名方式

建议使用明确的 Godot Scene Tree 名称：

```text
MainPanel
HeaderMargin
TitleLabel
CloseButton
ActionButtonsVBox
StartGameButton
SettingsButton
InventoryGrid
HpProgressBar
SearchLineEdit
ItemList
```

避免：

```text
Button1
Button2
Node123
AAA
框1
矩形2
```

中文节点名虽然工具可以显示，但如果后续要写 GDScript，通常优先使用清晰英文名。

---

# 13. 完整示例

下面是一个可以直接导入设计器的简化示例：

```json
{
  "version": 2,
  "projectName": "Main Menu Demo",
  "width": 1152,
  "height": 648,
  "zoom": 0.8,
  "grid": true,
  "snap": true,
  "gridSize": 8,
  "activeScreenId": "screen_main",
  "selectedTool": "select",
  "lastDrawType": "Button",
  "continuousDraw": false,
  "selectedNodeId": null,
  "activeSidebarTab": "inspector",
  "looseLayout": true,
  "screens": [
    {
      "id": "screen_main",
      "name": "MainMenu",
      "nodes": [
        {
          "id": "node_background",
          "type": "ColorRect",
          "name": "Background",
          "x": 0,
          "y": 0,
          "w": 1152,
          "h": 648,
          "text": "",
          "note": "主界面背景",
          "mcpNote": "",
          "parentId": null,
          "z": 0,
          "anchor": "full_rect",
          "color": "#28313d"
        },
        {
          "id": "node_menu_panel",
          "type": "PanelContainer",
          "name": "MainMenuPanel",
          "x": 396,
          "y": 150,
          "w": 360,
          "h": 360,
          "text": "",
          "note": "主菜单主体区域",
          "mcpNote": "结合项目 Theme 设置 StyleBox，不要求照抄临时颜色",
          "parentId": null,
          "z": 1,
          "anchor": "center",
          "color": "#dfe5eb"
        },
        {
          "id": "node_margin",
          "type": "MarginContainer",
          "name": "MenuMargin",
          "x": 420,
          "y": 174,
          "w": 312,
          "h": 312,
          "text": "",
          "note": "提供面板内边距",
          "mcpNote": "使用 Theme constant 设置合理 margins",
          "parentId": "node_menu_panel",
          "z": 2,
          "anchor": "full_rect",
          "color": "#dfe5eb"
        },
        {
          "id": "node_vbox",
          "type": "VBoxContainer",
          "name": "MenuVBox",
          "x": 440,
          "y": 210,
          "w": 272,
          "h": 240,
          "text": "",
          "note": "主菜单按钮纵向排列",
          "mcpNote": "按钮间 separation 由项目 Theme 决定",
          "parentId": "node_margin",
          "z": 3,
          "anchor": "full_rect",
          "color": "#dfe5eb"
        },
        {
          "id": "node_title",
          "type": "Label",
          "name": "TitleLabel",
          "x": 440,
          "y": 210,
          "w": 272,
          "h": 52,
          "text": "游戏标题",
          "note": "主标题，视觉权重最高",
          "mcpNote": "使用项目标题字体和字号规范",
          "parentId": "node_vbox",
          "z": 4,
          "anchor": "custom",
          "color": "#27313c"
        },
        {
          "id": "node_start",
          "type": "Button",
          "name": "StartGameButton",
          "x": 440,
          "y": 286,
          "w": 272,
          "h": 48,
          "text": "开始游戏",
          "note": "主操作按钮",
          "mcpNote": "连接 pressed 到开始游戏流程",
          "parentId": "node_vbox",
          "z": 5,
          "anchor": "custom",
          "color": "#dfe7ef"
        },
        {
          "id": "node_settings",
          "type": "Button",
          "name": "SettingsButton",
          "x": 440,
          "y": 350,
          "w": 272,
          "h": 48,
          "text": "设置",
          "note": "次要操作",
          "mcpNote": "pressed 打开 Settings 场景或面板",
          "parentId": "node_vbox",
          "z": 6,
          "anchor": "custom",
          "color": "#dfe7ef"
        }
      ]
    }
  ]
}
```

---

# 14. 给用户返回 JSON 时的输出规则

如果用户明确说：

> “给我能导入这个 UI Blueprint Designer 的文件/JSON”

请遵守：

1. 最好直接生成 `.json` 文件。
2. 如果只能回复文本，**只给一个完整 JSON 代码块**。
3. 不要在 JSON 内写注释。
4. 不要在 JSON 代码块前后夹杂“这里省略了一些节点”。
5. 所有引用的 `parentId` 都必须真实存在。
6. 所有 `id` 必须唯一。
7. 至少有一个 `screen`。
8. `activeScreenId` 必须指向真实 screen。
9. 不保存背景参考图。
10. 修改现有 JSON 时，不要无故丢字段。

---

# 15. 最后检查清单

在输出前检查：

- [ ] JSON 可以正常解析
- [ ] `version = 2`
- [ ] `screens` 非空
- [ ] `activeScreenId` 存在
- [ ] 节点 ID 没重复
- [ ] 节点名称清楚
- [ ] `parentId` 没有悬空
- [ ] 没有循环父子关系
- [ ] Graph 节点放在 GraphEdit 下
- [ ] SplitContainer 没有超过 2 个直接子节点
- [ ] 没把抽象基类当最终节点
- [ ] 没保存截图/背景图
- [ ] 用户备注没有被擅自删掉
- [ ] `looseLayout=true` 时没有把草图坐标误当成绝对不可变规格

---

## 一句话原则

> **把 JSON 当成“可回编辑的 UI 工程”，而不是一次性的最终答案。结构、语义、父子关系和用户意图优先；精确像素是否严格执行，由 `looseLayout` 决定。**
