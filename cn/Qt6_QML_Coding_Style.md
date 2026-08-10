# Qt6_QML_Coding_Style（中文）

English | 简体中文 | 原文

> 说明：本文档为当前 Qt6 / QML 代码规范的中文整理版（用于阅读与分发）。若与规范基线存在差异，以规范基线为准。

本规范隶属 `Qt6_CPP17_Coding_Style.md`，是 QML（`.qml` 文件与 QML JavaScript）编码规范的专题展开。

适用范围：

- `.qml` 文件
- QML 内嵌 JavaScript
- QML 使用的 `.js` helper
- Qt Quick / Qt Quick Controls 组件、页面、delegate、Loader、状态、动画、可访问性、国际化与性能规则

不适用范围：

- C++ 源码、QObject header、moc 宏布局
- C++ 侧 QML 注册宏
- QML ↔ C++ 边界类型与 C++ ownership 设计

权威边界：

- QML 语言层规则（对象声明顺序、绑定、组件 API、View/delegate、Loader、状态、动画、i18n、可访问性、QML 性能）以本文为权威。
- QML 注册宏、C++ 侧类体布局、`QML_ELEMENT`、`QML_NAMED_ELEMENT`、`QML_SINGLETON`、`Q_INVOKABLE` 等宏位置，以 `Qt_Macro_Layout_Coding_Style.md` 为准。
- QML ↔ C++ 边界类型、Borrow / Owning、QObject 所有权语义，以 `Qt6_KDE_API_Parameter_Style.md` 为准；本文只引用结论。
- 注释规范以 `CPP_Code_Comment_Guidelines.md` 为准；本文只补充 QML 语境中的增量要求。
- 基础格式、命名、生命周期、线程、错误处理，以 `Qt6_CPP17_Coding_Style.md` 为总纲。
- 本项目将无异常错误处理约束扩展到 QML/JavaScript 业务代码；这是本项目自行提高的约束，不是 Qt/QML 语法的普遍禁令。

---

## 0. 总览

### 0.1 基线

- Qt：Qt 6。
- 基础格式沿用 `Qt6_CPP17_Coding_Style.md`；本文不重复维护编码、缩进、行宽等通用数值。
- 格式化：使用 `qmlformat`。
- 静态检查：使用 `qmllint`。
- 测试：QML 行为变更必须覆盖 QML test 或项目等价测试。
- 运行期：不得引入新增 QML warning、绑定循环、无效类型转换或 Loader/Image 静默失败。

### 0.2 规则强度

- **必须**：违反即应拒绝合入。
- **禁止**：不得出现，除非本文明确给出受控例外。
- **应该**：新代码默认遵守；旧代码渐进迁移。
- **推荐**：优先采用，但可根据项目上下文调整。
- **可选**：有明确收益时采用，不作为默认门禁。

### 0.3 核心原则

- **声明式优先**：UI 状态通过绑定、状态机和属性表达，不用命令式脚本反复同步。
- **边界显式**：组件输入、delegate role、外部依赖使用 `required property` 或明确 API。
- **限定访问**：跨对象访问属性时使用 `id.property`，避免裸访问外层作用域。
- **轻量 QML**：QML 内 JavaScript 只做 UI glue code；复杂业务、I/O、大循环下沉到 C++ 或异步层。
- **可编译友好**：代码应尽量满足 `qmllint`、`qmlcachegen`、`qmlsc/qmltc` 的静态分析与编译要求。
- **性能内建**：首屏、绑定、delegate、动态创建和图片资源从设计阶段控制成本。

---

## 1. 文件、模块与 import

### 1.1 文件命名

| 类型 | 命名 | 正例 | 反例 |
|---|---|---|---|
| 可复用 QML 组件 | 大驼峰 | `GameCard.qml` | `game_card.qml` |
| 页面 / 场景 | 大驼峰 | `LibraryPage.qml` | `library-page.qml` |
| 私有辅助组件 | 大驼峰，放 `private/` 或内部目录 | `private/FocusRing.qml` | `_focusRing.qml` |
| JavaScript helper 文件 | 小驼峰或短横线 | `formatText.js` | `Utils.js` 泛化命名 |
| JavaScript import alias | 大写开头 | `import "formatText.js" as FormatText` | `import "formatText.js" as formatText` |

**必须**：

- 一个公开 QML 类型对应一个大驼峰 `.qml` 文件。
- 文件名表达组件职责，不使用 `Common.qml`、`Base.qml`、`Utils.qml` 这类泛化名称。
- JavaScript helper 必须通过 alias 导入，alias 必须大写开头。

**应该**：

- 私有组件通过目录、模块导出规则或项目约定隔离，不靠 `_` 前缀伪隐藏。
- 页面、组件、delegate、私有组件分别放在稳定目录中，避免同层级堆所有文件。

### 1.2 文件头顺序

可复用组件、delegate、包含 inline component 的文件，在项目 Qt 基线支持时**应该**使用 `pragma ComponentBehavior: Bound`。

推荐：

```qml
pragma ComponentBehavior: Bound

import QtQuick
import QtQuick.Controls
import QtQuick.Layouts

import App.Core
import App.Controls

import "private" as Private
import "formatText.js" as FormatText
```

顺序：

1. `pragma`。
2. Qt 官方模块。
3. 项目模块。
4. 相对目录。
5. JavaScript helper。

**必须**：

- `pragma` 位于 import 之前。
- import 分组清晰，组间空一行。
- JavaScript import 必须使用 alias。
- 不使用未被引用的 import。

**应该**：

- Qt 6 新代码默认使用无版本 import。
- 只有需要锁定兼容边界时才写 import 版本，并在项目内统一。

---

## 2. 命名

| 元素 | 风格 | 正例 | 反例 |
|---|---|---|---|
| 根对象 `id` | 固定 `root` | `id: root` | `id: item1` |
| 普通 `id` | 小驼峰 | `id: titleLabel` | `id: TitleLabel` |
| property | 小驼峰 | `property bool selected` | `property bool is_selected` |
| signal | 事件事实或过去式 | `signal accepted(string text)` | `signal acceptText()` |
| function | 动词小驼峰 | `function resetFocus()` | `function focus_reset()` |
| 状态名 | 简短小写或 lower-kebab | `"expanded"` / `"keyboard-active"` | `"ExpandedState"` |
| model role | 小驼峰 | `displayName` | `display_name` |

**必须**：

- 根对象默认使用 `id: root`。
- `id` 只服务当前文件内部，不作为外部 API。
- 公开 property / signal / function 是组件契约，命名必须稳定、具体。
- 布尔属性使用正向含义：`enabled`、`selected`、`expanded`，避免 `notDisabled`。

**禁止**：

- 用 `item1`、`rect2`、`dataObj` 这类无语义名称。
- signal 使用命令式命名，例如 `signal doSubmit()`。

---

## 3. QML 对象声明顺序

每个 QML 对象内部按以下顺序组织：

1. `id`
2. `required property`
3. 普通 `property`
4. `readonly property`
5. `property alias`
6. `signal`
7. JavaScript `function`
8. 基础属性：尺寸、布局、可见性、启用状态
9. 视觉属性：颜色、字体、透明度、层级
10. 交互属性：focus、hover、pressed、key、pointer handler
11. model / delegate / source / state
12. `states`
13. `transitions`
14. 生命周期处理：`Component.onCompleted` 等
15. 子对象

说明：

- `property alias` 可以引用文本上靠后声明的子对象；QML 按 id 解析，不依赖文本顺序。
- 即使 alias 能引用内部对象，也不应把内部对象整体暴露出去。
- alias 优先暴露具体子属性，例如 `text`、`source`，不要暴露 `titleLabel` 整个对象。

推荐：

```qml
import QtQuick
import QtQuick.Controls

Item {
    id: root

    required property string title
    property bool selected: false
    readonly property color titleColor: root.selected
        ? root.palette.highlightedText
        : root.palette.text
    property alias text: titleLabel.text

    signal accepted(string text)

    function reset() {
        root.selected = false;
    }

    implicitWidth: Math.max(titleLabel.implicitWidth, contentContainer.implicitWidth)
    implicitHeight: contentColumn.implicitHeight
    visible: root.title.length > 0
    enabled: true

    Label {
        id: titleLabel

        text: root.title
        color: root.titleColor
    }

    Item {
        id: contentContainer

        anchors.top: titleLabel.bottom
        anchors.left: parent.left
        anchors.right: parent.right
    }
}
```

禁止：

```qml
Item {
    color: "red"
    id: root
    Rectangle { id: background }
    property string title
    width: 100; height: 50
}
```

---

## 4. 格式与换行

**必须**：

- 一行只写一个属性绑定。
- QML 属性行不写分号。
- JavaScript block 内多语句使用分号。
- 控制语句必须加 braces。
- 长表达式拆为 `readonly property` 或函数。
- 对象内部不同声明区之间空一行。

推荐：

```qml
readonly property bool canSubmit: nameField.text.length > 0
    && !submitButton.busy

function canAcceptText(text: string): bool {
    return text.trim().length > 0;
}

function submit() {
    if (!root.canSubmit) {
        return;
    }

    root.accepted(nameField.text);
}
```

禁止：

```qml
onClicked: if (nameField.text.length > 0) root.accepted(nameField.text)
width: parent.width > 800 ? (sidebar.visible ? parent.width - sidebar.width - 24 : parent.width - 12) : parent.width
```

---

## 5. 属性、绑定与作用域

### 5.1 属性类型

**必须**：

- 能写具体类型就不写 `var`。
- 外部必需输入使用 `required property`。
- 派生值使用 `readonly property`。
- `property alias` 只暴露必要子属性。
- 公开组件 API 的 property 类型保持稳定。

推荐：

```qml
required property string title
property int count: 0
property color accentColor: "#4c8dff"
readonly property bool empty: root.count === 0
property alias text: titleLabel.text
```

慎用：

```qml
property var data
property alias label: titleLabel
```

### 5.2 绑定规则

**必须**：

- 绑定表达式无副作用。
- 不在绑定中修改状态、写日志、发信号或触发异步操作。
- 不对已有绑定的属性做命令式赋值，除非明确要打断绑定。
- 若必须重新设置绑定，使用 `Qt.binding()`。
- 不制造绑定循环。

推荐：

```qml
height: width * 0.6
visible: root.modelCount > 0
```

禁止：

```qml
visible: {
    console.log("checking visibility");
    return root.modelCount > 0;
}
```

命令式恢复绑定：

```qml
height = Qt.binding(function() {
    return width * 0.6;
});
```

### 5.3 限定作用域访问

**必须**：

- 跨对象访问属性时使用 `id.property`。
- 组件内部访问根属性时使用 `root.property`。
- delegate 访问自身 role 时使用显式 `required property`。
- 禁止依赖外层作用域的裸属性查找。

推荐：

```qml
Item {
    id: root

    property int spacing: 12

    Rectangle {
        id: background

        radius: root.spacing / 2
    }
}
```

禁止：

```qml
Item {
    property int spacing: 12

    Rectangle {
        radius: spacing / 2
    }
}
```

### 5.4 qmlsc / 静态分析友好

**应该**：

- 给非平凡函数添加参数与返回值类型注解。
- 避免动态给对象添加新属性。
- 避免 `eval()`、隐式全局变量和类型不稳定的 `var`。
- 避免依赖 context property 魔法注入。
- 使用 `required property` 显式接收外部输入和 model role。

Qt 6.7+ 项目中，函数类型注解会被运行期调用强制执行；不要添加 `pragma FunctionSignatureBehavior: Ignored` 降级，除非项目有明确兼容理由。

---

## 6. JavaScript 使用

**必须**：

- QML 内 JavaScript 只处理局部 UI glue code。
- 不把业务核心、I/O、大循环、模型批量转换写在 QML。
- 不使用隐式全局变量。
- 不吞掉错误状态；QML/JavaScript 业务代码不得引入异常控制流。
- 多语句 JavaScript 使用分号。
- 使用 `let` / `const`，避免新增 `var`。

推荐：

```qml
function displayTitle(title: string, fallback: string): string {
    const trimmedTitle = title.trim();
    if (trimmedTitle.length > 0) {
        return trimmedTitle;
    }

    return fallback;
}
```

禁止：

```qml
function rebuildWholeModel() {
    for (let i = 0; i < 10000; ++i) {
        model.append(expensiveTransform(source[i]));
    }
}
```

### 6.1 `.js` helper

**应该**：

- 无状态共享 helper 顶部使用 `.pragma library`。
- 有状态 per-instance helper 不使用 `.pragma library`，并用注释说明状态语义。
- helper 函数通过参数接收 QML 对象或值，不直接依赖 QML 实例上下文。

推荐：

```javascript
// formatText.js
.pragma library

function fallbackTitle(title, fallback) {
    const trimmedTitle = title.trim();
    return trimmedTitle.length > 0 ? trimmedTitle : fallback;
}
```

QML 使用：

```qml
import "formatText.js" as FormatText

Label {
    text: FormatText.fallbackTitle(root.title, qsTr("Untitled"))
}
```

---

## 7. 组件 API 设计

**必须**：

- 组件对外只暴露必要 property、signal、function。
- 必需输入使用 `required property`。
- 内部结构通过 `id` 隐藏，不让调用方依赖内部层级。
- 可复用组件必须提供合理 `implicitWidth` / `implicitHeight`。
- 可复用组件不写死 `width` / `height`，除非它本身就是固定尺寸资产。
- 不通过 alias 暴露内部对象整体。

推荐：

```qml
Control {
    id: root

    required property string text
    property bool busy: false

    signal activated()

    implicitWidth: Math.max(contentItem.implicitWidth + leftPadding + rightPadding, 96)
    implicitHeight: contentItem.implicitHeight + topPadding + bottomPadding

    contentItem: Label {
        text: root.text
        color: root.palette.buttonText
        font: root.font
        elide: Text.ElideRight
        horizontalAlignment: Text.AlignHCenter
        verticalAlignment: Text.AlignVCenter
    }

    background: Rectangle {
        color: root.enabled ? root.palette.button : root.palette.mid
        border.color: root.visualFocus ? root.palette.highlight : root.palette.mid
        radius: 6
    }
}
```

**应该**：

- 自定义 Qt Quick Controls 外观时，优先定制 `contentItem`、`background`、`indicator`、`popup` 等官方槽位。
- 颜色优先使用 `palette` 或项目设计 token，不硬编码散落颜色。
- 组件 API 稳定后，不随意改名或改变 property 类型。

禁止：

```qml
Item {
    id: root

    width: 300
    height: 80

    property alias label: titleLabel
}
```

---

## 8. 布局、尺寸与锚点

### 8.1 Anchors 与 Layouts

**必须**：

- 简单父子定位使用 anchors。
- 表单、工具栏、动态排列使用 Layouts。
- 不在同一个 item 上混用 anchors 和 `Layout.*`。
- 不手动设置 Layout 管理项的 `x` / `y`。
- 优先设置 `implicitWidth` / `implicitHeight`，让父级决定实际尺寸。

推荐：

```qml
ColumnLayout {
    anchors.fill: parent
    spacing: 12

    Label {
        text: root.title
        Layout.fillWidth: true
    }

    Button {
        text: qsTr("Apply")
        Layout.alignment: Qt.AlignRight
    }
}
```

禁止：

```qml
Button {
    anchors.left: parent.left
    Layout.fillWidth: true
    x: 20
}
```

### 8.2 尺寸 token

**应该**：

- 重复出现的尺寸、间距、圆角、时长抽为 token。
- 视觉 token 优先来自设计系统、主题对象或 palette。
- 一次性局部数值可以保留，但语义必须清楚。

推荐：

```qml
readonly property int spacingMedium: 12
readonly property int cornerRadius: 8

Column {
    spacing: root.spacingMedium
}
```

---

## 9. 状态、动画与过渡

**必须**：

- 状态表达 UI 模式，不表达临时命令。
- 不用动画掩盖主线程阻塞或加载慢。
- 动画不改变业务事实，只表达视觉过渡。
- 列表 delegate 内避免大量常驻动画。
- 状态切换不得破坏焦点、可访问性和输入路径。

**应该**：

- 多属性联动使用 `State` + `Transition`。
- 单属性局部变化使用 `Behavior`。
- 动画时长短、明确、可调。
- 状态名使用稳定语义，例如 `"expanded"`、`"empty"`、`"error"`。

推荐：

```qml
Item {
    id: root

    property bool expanded: false

    DetailsPanel {
        id: details

        opacity: 0
        visible: opacity > 0
        enabled: root.expanded
    }

    states: [
        State {
            name: "expanded"
            when: root.expanded

            PropertyChanges {
                target: details
                opacity: 1
            }
        }
    ]

    transitions: [
        Transition {
            NumberAnimation {
                properties: "opacity"
                duration: 140
                easing.type: Easing.OutCubic
            }
        }
    ]
}
```

禁止：

```qml
State {
    name: "expanded"

    PropertyChanges {
        target: details
        opacity: 1
        visible: true
    }
}
```

说明：退出 state 时 `visible` 会立即恢复为 `false`，opacity 淡出动画不可见。

---

## 10. Model、View 与 delegate

**必须**：

- 大列表使用 `ListView` / `GridView` / `TableView`，不要用 `Repeater` 承载大量可滚动项。
- delegate 必须轻量，不做 I/O、网络请求、大循环或重计算。
- delegate 依赖的 model role 使用 `required property` 显式声明。
- delegate 不依赖实例永久存在。
- delegate 不直接修改全局状态；通过 signal 或明确 API 向外报告用户动作。
- 开启 delegate 复用前，必须确认 delegate 状态已外部化或可重置。

**应该**：

- 复用型 delegate 文件顶部使用 `pragma ComponentBehavior: Bound`。
- 使用 `required property` 接收 model role，不依赖 `model.xxx` 私有上下文。
- delegate 内跨对象访问使用 id 限定。

推荐：

```qml
pragma ComponentBehavior: Bound

import QtQuick
import QtQuick.Controls

ListView {
    id: gamesList

    model: gamesModel
    clip: true

    delegate: ItemDelegate {
        id: delegateRoot

        required property int index
        required property string title
        required property url coverUrl

        width: ListView.view.width
        text: delegateRoot.title

        onClicked: {
            gamesList.currentIndex = delegateRoot.index;
        }
    }
}
```

禁止：

```qml
Repeater {
    model: 5000

    delegate: HeavyGameCard {
        Component.onCompleted: loadNetworkData()
    }
}
```

---

## 11. Loader 与动态对象

**必须**：

- 优先使用 `Loader`、`Component`、`Instantiator`、`Repeater`、View delegate 等声明式机制。
- 只有声明式机制无法表达时，才使用 `Qt.createComponent()`。
- 禁止在业务 UI 中使用 `Qt.createQmlObject()` 拼接运行期 QML 字符串。
- 动态创建对象时，parent 必须比对象活得更久。
- 不再需要的动态对象必须显式 `destroy()` 或由明确 owner 统一释放。
- Loader、动态创建和异步加载必须处理失败状态。

受控例外：

- 工具、测试、隔离沙箱场景可以使用 `Qt.createQmlObject()`，但必须明确输入来源可信、生命周期可控、错误可诊断。

推荐：

```qml
Loader {
    id: detailsLoader

    active: root.expanded
    asynchronous: true
    sourceComponent: DetailsPanel {
        itemId: root.itemId
    }

    onStatusChanged: {
        if (status === Loader.Error) {
            console.warn("Failed to load details panel:", detailsLoader.source);
        }
    }
}
```

动态创建推荐：

```qml
function createPopup(component: Component): Item {
    const object = component.createObject(root, {
        "x": 0,
        "y": 0
    });

    if (object === null) {
        console.warn("Failed to create popup component");
        return null;
    }

    return object;
}
```

---

## 12. 信号处理与 Connections

**必须**：

- 普通 QML 信号处理器可以使用 `onClicked: { ... }` 等直接 handler。
- `Connections` 内必须使用函数式语法：`function onXxxChanged() { ... }`。
- 禁止在 `Connections` 中使用旧式 `onXxx: ...` 写法。
- `Connections.target` 可能为 `null` 时，逻辑必须可安全处理。
- `ignoreUnknownSignals` 只用于兼容多种 target 类型，并需要注释说明原因。

推荐：

```qml
Connections {
    target: root.backend

    function onStatusChanged() {
        root.statusText = root.backend.statusText;
    }
}
```

禁止：

```qml
Connections {
    target: root.backend
    onStatusChanged: root.statusText = backend.statusText
}
```

---

## 13. 输入、焦点与可访问性

**必须**：

- 可交互组件必须可键盘到达。
- 复合控件使用 `FocusScope` 管理内部焦点。
- 可见焦点必须明确。
- 按键处理必须明确 `event.accepted` 语义。
- 不在叶子组件随意截获全局按键。
- 不用 `MouseArea` 覆盖已有控件的语义行为。
- 自定义交互组件必须设置合适的 Accessible 信息。

**应该**：

- 优先使用 Qt Quick Controls 或 Pointer Handlers。
- Qt Quick Controls 已内置 Accessible 默认语义时，不重复覆盖导致语义冲突。
- 自定义控件才补充 `Accessible.role`、`Accessible.name`、`Accessible.description`。

推荐：

```qml
FocusScope {
    id: root

    required property string text

    signal activated()

    focus: true

    Accessible.role: Accessible.Button
    Accessible.name: root.text

    Keys.onPressed: function(event) {
        if (event.key === Qt.Key_Return || event.key === Qt.Key_Space) {
            root.activated();
            event.accepted = true;
            return;
        }

        event.accepted = false;
    }
}
```

禁止：

```qml
Keys.onPressed: {
    root.forceNavigateSomewhere();
    event.accepted = true;
}
```

---

## 14. 文本、字体与国际化

**必须**：

- 用户可见文本使用 `qsTr()`、`qsTrId()` 或项目统一翻译机制。
- 不拼接可翻译句子。
- 翻译文本使用占位符。
- 文本必须考虑长字符串、空字符串、elide、wrap 和多语言长度差异。
- 不把业务状态写成硬编码英文。
- 字体、字号、字重优先来自主题或设计 token。

推荐：

```qml
Label {
    text: qsTr("Installed games: %1").arg(root.installedCount)
    elide: Text.ElideRight
    maximumLineCount: 1
}
```

复数推荐：

```qml
Label {
    text: qsTr("%n file(s) selected", "", root.selectedCount)
}
```

禁止：

```qml
Label {
    text: "Installed games: " + root.installedCount
}
```

---

## 15. 图片、资源与视觉性能

**必须**：

- 禁止使用机器本地绝对路径引用资源。
- 资源路径使用 `qrc:/`、模块资源路径或项目统一资源路径。
- 大图必须设置合适的 `sourceSize` 或由资源管线生成合适尺寸。
- 远程、非首屏或大图优先异步加载。
- Image / AnimatedImage 必须处理错误状态。
- 不用全尺寸大图缩成小图长期显示。

**应该**：

- 高频变化图片谨慎使用缓存。
- 慎用 `clip: true`、`layer.enabled: true`、大面积 opacity。
- SVG、Shader、模糊、阴影等效果必须实测。
- 图标优先使用统一图标组件或主题资源。

推荐：

```qml
Image {
    id: cover

    source: root.coverUrl
    sourceSize.width: 320
    sourceSize.height: 180
    asynchronous: true
    fillMode: Image.PreserveAspectCrop

    onStatusChanged: {
        if (status === Image.Error) {
            console.warn("Cover image failed:", root.coverUrl);
        }
    }
}
```

---

## 16. 错误处理与诊断

**必须**：

- 不提交无意义 `console.log()`。
- `console.warn()` / `console.error()` 必须包含可定位上下文。
- Loader、Image、动态创建、异步组件必须处理失败状态。
- **项目自定义加严约束（强制）**：QML/JavaScript 业务代码禁止 `throw`、`try`、`catch`；不得把 JavaScript 异常作为 API 错误返回或控制流机制。
- API 失败必须通过 C++ 暴露的错误码、错误状态、`errorString()` 或等价的显式结果/属性表达。
- 不允许静默吞错后继续假装成功。
- 稳定错误码、诊断字段、trace payload 必须集中定义，不散落字符串。

---

## 17. 性能规则

### 17.1 必须遵守

- 不在绑定中做重计算。
- 不在 `Component.onCompleted` 同步创建大量对象。
- 不在 delegate 中进行网络请求、文件读取或大循环。
- 不在动画中修改会触发布局级联的大量属性。
- 不在热路径频繁创建临时对象、数组或正则。
- 不制造 `width` / `implicitWidth`、父子尺寸、Layout 与 anchors 的绑定循环。
- 首屏优先加载用户立即可见内容，非关键内容延迟加载。

### 17.2 应该采用

- 使用 QML Profiler 定位启动、绑定、创建和渲染瓶颈。
- 高频更新使用 `Qt.callLater()` 合并到下一轮事件循环。
- 非首屏复杂区域使用 Loader 延迟加载。
- 能由模型提供的数据，不在 delegate 中二次扫描模型。
- 复杂排序、过滤、聚合下沉到模型层。

推荐：

```qml
property bool refreshPending: false

function scheduleRefresh() {
    if (root.refreshPending) {
        return;
    }

    root.refreshPending = true;
    Qt.callLater(function() {
        root.refreshPending = false;
        root.refreshVisibleState();
    });
}
```

绑定循环反例：

```qml
Item {
    implicitWidth: width + 12
    width: implicitWidth
}
```

---

## 18. QML 注释增量

完整注释规则以 `CPP_Code_Comment_Guidelines.md` 为准。本节只补充 `.qml` 语境中容易被误删或误写的注释边界。

**必须**：

- QML 稳定契约字段需要说明外部依赖，例如 model role、restore key、trace payload、诊断字段。
- 状态、绑定、delegate、Loader 生命周期存在非直观时序或副作用时，注释“为什么这样做”，不解释语法。
- 可访问性、输入焦点、动态创建对象的生命周期规则不直观时，注释稳定边界和失败行为。

**禁止**：

- 为简单属性、简单绑定、显然 signal 或普通 handler 逐项堆注释。
- 用注释复述 QML 语法或代码字面行为。

推荐：

```qml
// Stable role consumed by tests and saved view state.
required property string restoreKey

// Keep visible derived from opacity so the exit transition can fade out.
visible: opacity > 0
```

---

## 19. 工具与验证

提交前至少执行项目封装的等价命令。

示例：

```bash
qmlformat -i path/to/file.qml
qmllint path/to/file.qml
qmltestrunner
```

如项目使用 CMake：

```bash
cmake --build build
ctest --test-dir build --output-on-failure
```

**必须**：

- `qmlformat` 后无非预期格式差异。
- `qmllint` 无新增 warning。
- QML 测试通过。
- UI 组件变更覆盖关键状态：默认、hover/focus、pressed、disabled、empty、error、loading。
- 性能敏感变更使用 QML Profiler 或项目等价工具验证。

---

## 20. 提交前自检清单

```text
- [ ] 文件 UTF-8 无 BOM
- [ ] 组件文件大驼峰命名
- [ ] import 顺序正确，无无意义版本锁定
- [ ] 复用型组件 / delegate 已按项目基线使用 pragma ComponentBehavior: Bound
- [ ] 根对象使用 id: root
- [ ] 对象内部声明顺序一致
- [ ] 缩进 4 空格，一行一个属性
- [ ] 能用具体 property 类型时未使用 var
- [ ] 外部必需输入使用 required property
- [ ] delegate role 使用 required property 显式声明
- [ ] 跨对象访问使用 id.property 限定
- [ ] 派生值使用 readonly property
- [ ] property alias 未泄漏内部对象
- [ ] 无绑定副作用
- [ ] 无命令式打断绑定的误用
- [ ] 无绑定循环
- [ ] anchors 与 Layout 未混用
- [ ] 组件提供合理 implicitWidth / implicitHeight
- [ ] 自定义 Controls 优先使用 contentItem / background / palette
- [ ] 大列表使用 View，不使用 Repeater
- [ ] delegate 轻量，未做 I/O 或重计算
- [ ] Connections 使用 function onXxx() 语法
- [ ] Loader / Image / 动态创建处理错误状态
- [ ] 用户可见文本使用 qsTr / qsTrId 或项目翻译机制
- [ ] 可交互组件可键盘到达，焦点可见
- [ ] 自定义交互控件设置合适 Accessible 信息
- [ ] 无无意义 console.log
- [ ] qmllint 无新增 warning
- [ ] qmlformat 无非预期格式差异
- [ ] QML 测试通过
```

---

**文档包版本**：v1.1.0
**最后更新**：2026-07-25

---
