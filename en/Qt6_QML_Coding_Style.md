# Qt6 QML Coding Style

English | 简体中文 | Source

> Note: This document is the English translation of the current Qt6 / QML coding guideline. If there is any discrepancy, the package baseline prevails.

This guideline belongs to `Qt6_CPP17_Coding_Style.md` and expands the package rules for QML (`.qml` files and QML JavaScript).

Scope:

- `.qml` files
- JavaScript embedded in QML
- `.js` helpers used by QML
- Qt Quick / Qt Quick Controls components, pages, delegates, loaders, states, animations, accessibility, internationalization, and performance rules

Out of scope:

- C++ source code, QObject headers, and moc macro layout
- C++-side QML registration macros
- QML ↔ C++ boundary types and C++ ownership design

Authority boundaries:

- QML language-level rules (object declaration order, bindings, component APIs, View/delegate, Loader, states, animations, i18n, accessibility, and QML performance) are owned by this document.
- QML registration macros, C++-side class layout, `QML_ELEMENT`, `QML_NAMED_ELEMENT`, `QML_SINGLETON`, `Q_INVOKABLE`, and related macro placement are owned by `Qt_Macro_Layout_Coding_Style.md`.
- QML ↔ C++ boundary types, Borrow / Owning semantics, and QObject ownership are owned by `Qt6_KDE_API_Parameter_Style.md`; this document only references their conclusions.
- Commenting rules are owned by `CPP_Code_Comment_Guidelines.md`; this document only adds QML-specific notes.
- Baseline formatting, naming, lifetime, threading, and error handling are governed by `Qt6_CPP17_Coding_Style.md`.

---

## 0. Overview

### 0.1 Baseline

- Qt: Qt 6.
- Baseline formatting follows `Qt6_CPP17_Coding_Style.md`; this document does not duplicate generic encoding, indentation, or line-length numbers.
- Formatting: use `qmlformat`.
- Static checks: use `qmllint`.
- Tests: QML behavior changes must be covered by QML tests or equivalent project tests.
- Runtime: do not introduce new QML warnings, binding loops, invalid type conversions, or silent Loader/Image failures.

### 0.2 Rule Strength

- **Must**: violations should block the change.
- **Forbidden**: do not do this unless this document explicitly defines a controlled exception.
- **Should**: default for new code; migrate existing code gradually.
- **Recommended**: prefer this, but adapt to project context when needed.
- **Optional**: use when it has clear value; not a default gate.

### 0.3 Core Principles

- **Declarative first**: express UI state through bindings, states, and properties, not repeated imperative synchronization.
- **Explicit boundaries**: use `required property` or explicit APIs for component inputs, delegate roles, and external dependencies.
- **Qualified access**: use `id.property` for cross-object access; avoid bare access to outer scopes.
- **Lightweight QML**: keep QML JavaScript as UI glue code; move complex business logic, I/O, and large loops to C++ or async layers.
- **Compiler-friendly**: keep code friendly to `qmllint`, `qmlcachegen`, and `qmlsc/qmltc`.
- **Performance by design**: account for first screen, bindings, delegates, dynamic creation, and image resources during design.

---

## 1. Files, Modules, and Imports

### 1.1 File Naming

| Kind | Naming | Good | Bad |
|---|---|---|---|
| Reusable QML component | PascalCase | `GameCard.qml` | `game_card.qml` |
| Page / scene | PascalCase | `LibraryPage.qml` | `library-page.qml` |
| Private helper component | PascalCase, under `private/` or an internal directory | `private/FocusRing.qml` | `_focusRing.qml` |
| JavaScript helper file | camelCase or kebab-case | `formatText.js` | generic `Utils.js` |
| JavaScript import alias | Uppercase initial | `import "formatText.js" as FormatText` | `import "formatText.js" as formatText` |

**Must**:

- One public QML type maps to one PascalCase `.qml` file.
- File names describe component responsibility; avoid generic names such as `Common.qml`, `Base.qml`, or `Utils.qml`.
- JavaScript helpers must be imported with an alias, and the alias must start with an uppercase letter.

**Should**:

- Isolate private components via directories, module export rules, or project conventions; do not rely on a leading `_` as fake hiding.
- Keep pages, components, delegates, and private components in stable directories instead of dumping everything into one level.

### 1.2 File Header Order

Reusable components, delegates, and files containing inline components should use `pragma ComponentBehavior: Bound` when the project Qt baseline supports it.

Recommended:

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

Order:

1. `pragma`.
2. Qt modules.
3. Project modules.
4. Relative directories.
5. JavaScript helpers.

**Must**:

- Put `pragma` before imports.
- Keep import groups clear, with a blank line between groups.
- Import JavaScript helpers with aliases.
- Do not keep unused imports.

**Should**:

- New Qt 6 code should use unversioned imports by default.
- Only write import versions when a compatibility boundary must be pinned, and keep that policy consistent across the project.

---

## 2. Naming

| Element | Style | Good | Bad |
|---|---|---|---|
| Root object `id` | fixed `root` | `id: root` | `id: item1` |
| Ordinary `id` | camelCase | `id: titleLabel` | `id: TitleLabel` |
| property | camelCase | `property bool selected` | `property bool is_selected` |
| signal | event fact or past tense | `signal accepted(string text)` | `signal acceptText()` |
| function | verb-based camelCase | `function resetFocus()` | `function focus_reset()` |
| state name | short lowercase or lower-kebab | `"expanded"` / `"keyboard-active"` | `"ExpandedState"` |
| model role | camelCase | `displayName` | `display_name` |

**Must**:

- Use `id: root` for the root object by default.
- Treat `id` as local to the current file, not as an external API.
- Public properties, signals, and functions are component contracts; name them stably and specifically.
- Use positive boolean names such as `enabled`, `selected`, and `expanded`; avoid names like `notDisabled`.

**Forbidden**:

- Meaningless names such as `item1`, `rect2`, or `dataObj`.
- Imperative signal names such as `signal doSubmit()`.

---

## 3. QML Object Declaration Order

Declare attributes inside each QML object in this order:

1. `id`
2. `required property`
3. ordinary `property`
4. `readonly property`
5. `property alias`
6. `signal`
7. JavaScript `function`
8. basic attributes: size, layout, visibility, enabled state
9. visual attributes: color, font, opacity, z-order
10. interaction attributes: focus, hover, pressed, key, pointer handler
11. model / delegate / source / state
12. `states`
13. `transitions`
14. lifecycle handlers such as `Component.onCompleted`
15. child objects

Notes:

- `property alias` may reference child objects declared later in the file; QML resolves by id, not by textual order.
- Even if an alias can reference an internal object, do not expose the entire internal object.
- Prefer aliasing specific child properties such as `text` or `source`; do not expose the whole `titleLabel` object.

Recommended:

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

Forbidden:

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

## 4. Formatting and Line Breaks

**Must**:

- Put only one property binding on each line.
- Do not put semicolons after QML property lines.
- Use semicolons for multi-statement JavaScript blocks.
- Always use braces for control statements.
- Split long expressions into `readonly property` declarations or functions.
- Separate different declaration groups inside an object with a blank line.

Recommended:

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

Forbidden:

```qml
onClicked: if (nameField.text.length > 0) root.accepted(nameField.text)
width: parent.width > 800 ? (sidebar.visible ? parent.width - sidebar.width - 24 : parent.width - 12) : parent.width
```

---

## 5. Properties, Bindings, and Scope

### 5.1 Property Types

**Must**:

- Use a concrete type whenever possible instead of `var`.
- Use `required property` for required external inputs.
- Use `readonly property` for derived values.
- Use `property alias` only for necessary child properties.
- Keep public component API property types stable.

Recommended:

```qml
required property string title
property int count: 0
property color accentColor: "#4c8dff"
readonly property bool empty: root.count === 0
property alias text: titleLabel.text
```

Use with care:

```qml
property var data
property alias label: titleLabel
```

### 5.2 Binding Rules

**Must**:

- Keep binding expressions side-effect free.
- Do not mutate state, write logs, emit signals, or start async work from a binding.
- Do not assign imperatively to an already-bound property unless you intentionally break the binding.
- Use `Qt.binding()` when you must assign a new binding imperatively.
- Do not create binding loops.

Recommended:

```qml
height: width * 0.6
visible: root.modelCount > 0
```

Forbidden:

```qml
visible: {
    console.log("checking visibility");
    return root.modelCount > 0;
}
```

Restoring a binding imperatively:

```qml
height = Qt.binding(function() {
    return width * 0.6;
});
```

### 5.3 Qualified Scope Access

**Must**:

- Use `id.property` for cross-object property access.
- Use `root.property` when accessing root properties from inside a component.
- Use explicit `required property` declarations for delegate roles.
- Do not rely on bare lookup into an outer scope.

Recommended:

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

Forbidden:

```qml
Item {
    property int spacing: 12

    Rectangle {
        radius: spacing / 2
    }
}
```

### 5.4 qmlsc / Static-Analysis Friendly Code

**Should**:

- Add parameter and return type annotations to non-trivial functions.
- Avoid dynamically adding new properties to objects.
- Avoid `eval()`, implicit globals, and type-unstable `var`.
- Avoid magic context-property injection.
- Use `required property` to receive external inputs and model roles explicitly.

In Qt 6.7+ projects, function type annotations are enforced at runtime calls; do not add `pragma FunctionSignatureBehavior: Ignored` unless the project has a clear compatibility reason.

---

## 6. JavaScript Usage

**Must**:

- Keep JavaScript in QML as local UI glue code.
- Do not put core business logic, I/O, large loops, or bulk model transformation in QML.
- Do not use implicit global variables.
- Do not swallow exceptions or error states.
- Use semicolons for multi-statement JavaScript.
- Use `let` / `const`; avoid new `var`.

Recommended:

```qml
function displayTitle(title: string, fallback: string): string {
    const trimmedTitle = title.trim();
    if (trimmedTitle.length > 0) {
        return trimmedTitle;
    }

    return fallback;
}
```

Forbidden:

```qml
function rebuildWholeModel() {
    for (let i = 0; i < 10000; ++i) {
        model.append(expensiveTransform(source[i]));
    }
}
```

### 6.1 `.js` Helpers

**Should**:

- Use `.pragma library` at the top of stateless shared helpers.
- Do not use `.pragma library` for intentional per-instance state; add a comment explaining the state semantics.
- Pass QML objects or values as function arguments; do not directly depend on a QML instance context.

Recommended:

```javascript
// formatText.js
.pragma library

function fallbackTitle(title, fallback) {
    const trimmedTitle = title.trim();
    return trimmedTitle.length > 0 ? trimmedTitle : fallback;
}
```

QML usage:

```qml
import "formatText.js" as FormatText

Label {
    text: FormatText.fallbackTitle(root.title, qsTr("Untitled"))
}
```

---

## 7. Component API Design

**Must**:

- Expose only necessary properties, signals, and functions.
- Use `required property` for required inputs.
- Hide internal structure behind `id`; callers must not depend on internal object hierarchy.
- Provide reasonable `implicitWidth` / `implicitHeight` for reusable components.
- Do not hardcode `width` / `height` in reusable components unless the component is inherently fixed-size.
- Do not expose whole internal objects through aliases.

Recommended:

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

**Should**:

- When customizing Qt Quick Controls, prefer the official slots such as `contentItem`, `background`, `indicator`, and `popup`.
- Prefer `palette` or project design tokens for colors; do not scatter hardcoded colors.
- Once a component API is stable, do not casually rename it or change property types.

Forbidden:

```qml
Item {
    id: root

    width: 300
    height: 80

    property alias label: titleLabel
}
```

---

## 8. Layout, Size, and Anchors

### 8.1 Anchors and Layouts

**Must**:

- Use anchors for simple parent-child positioning.
- Use Layouts for forms, toolbars, and dynamic arrangement.
- Do not mix anchors and `Layout.*` on the same item.
- Do not manually set `x` / `y` on items managed by a Layout.
- Prefer `implicitWidth` / `implicitHeight` and let the parent decide the actual size.

Recommended:

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

Forbidden:

```qml
Button {
    anchors.left: parent.left
    Layout.fillWidth: true
    x: 20
}
```

### 8.2 Size Tokens

**Should**:

- Extract repeated sizes, spacing, radii, and durations into tokens.
- Prefer design-system tokens, theme objects, or palettes for visual tokens.
- Local one-off values are acceptable when their meaning is clear.

Recommended:

```qml
readonly property int spacingMedium: 12
readonly property int cornerRadius: 8

Column {
    spacing: root.spacingMedium
}
```

---

## 9. States, Animations, and Transitions

**Must**:

- Use states to represent UI modes, not temporary commands.
- Do not use animation to hide main-thread blocking or slow loading.
- Animations must not change business facts; they only express visual transition.
- Avoid large numbers of always-on animations inside list delegates.
- State transitions must not break focus, accessibility, or input paths.

**Should**:

- Use `State` + `Transition` for multi-property changes.
- Use `Behavior` for local single-property changes.
- Keep animation durations short, explicit, and adjustable.
- Use stable semantic state names such as `"expanded"`, `"empty"`, or `"error"`.

Recommended:

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

Forbidden:

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

Reason: when leaving the state, `visible` immediately returns to `false`, so the opacity fade-out cannot be seen.

---

## 10. Model, View, and Delegate

**Must**:

- Use `ListView` / `GridView` / `TableView` for large lists; do not use `Repeater` for large scrollable content.
- Keep delegates lightweight; do not perform I/O, network requests, large loops, or heavy computation inside delegates.
- Declare model roles used by delegates with explicit `required property`.
- Delegates must not rely on instances living forever.
- Delegates must not mutate global state directly; report user actions through signals or explicit APIs.
- Before enabling delegate reuse, ensure delegate state is externalized or resettable.

**Should**:

- Use `pragma ComponentBehavior: Bound` at the top of reusable delegate files.
- Receive model roles via `required property`; do not rely on private `model.xxx` context values.
- Use qualified id access inside delegates.

Recommended:

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

Forbidden:

```qml
Repeater {
    model: 5000

    delegate: HeavyGameCard {
        Component.onCompleted: loadNetworkData()
    }
}
```

---

## 11. Loader and Dynamic Objects

**Must**:

- Prefer declarative mechanisms such as `Loader`, `Component`, `Instantiator`, `Repeater`, and View delegates.
- Use `Qt.createComponent()` only when declarative mechanisms cannot express the need.
- Do not use `Qt.createQmlObject()` to build runtime QML strings for product UI.
- When creating objects dynamically, the parent must outlive the created object.
- Destroy objects explicitly with `destroy()` when they are no longer needed, or release them through a clear owner.
- Handle failures from Loader, dynamic creation, and async loading.

Controlled exception:

- Tools, tests, and isolated sandbox scenarios may use `Qt.createQmlObject()`, but the input source must be trusted, the lifetime must be controlled, and errors must be diagnosable.

Recommended:

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

Recommended dynamic creation:

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

## 12. Signal Handling and Connections

**Must**:

- Ordinary QML signal handlers may use direct handlers such as `onClicked: { ... }`.
- Inside `Connections`, use function syntax: `function onXxxChanged() { ... }`.
- Do not use old-style `onXxx: ...` syntax inside `Connections`.
- If `Connections.target` may be `null`, the logic must handle that safely.
- Use `ignoreUnknownSignals` only for compatible target variants, and add a comment explaining why.

Recommended:

```qml
Connections {
    target: root.backend

    function onStatusChanged() {
        root.statusText = root.backend.statusText;
    }
}
```

Forbidden:

```qml
Connections {
    target: root.backend
    onStatusChanged: root.statusText = backend.statusText
}
```

---

## 13. Input, Focus, and Accessibility

**Must**:

- Interactive components must be keyboard reachable.
- Composite controls should use `FocusScope` to manage internal focus.
- Visible focus must be clear.
- Key handling must make `event.accepted` semantics explicit.
- Do not intercept global keys casually in leaf components.
- Do not use `MouseArea` to override semantics already provided by controls.
- Custom interactive components must provide suitable Accessible information.

**Should**:

- Prefer Qt Quick Controls or Pointer Handlers.
- Do not override built-in Accessible semantics of Qt Quick Controls unless necessary.
- Add `Accessible.role`, `Accessible.name`, and `Accessible.description` for custom controls.

Recommended:

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

Forbidden:

```qml
Keys.onPressed: {
    root.forceNavigateSomewhere();
    event.accepted = true;
}
```

---

## 14. Text, Fonts, and Internationalization

**Must**:

- Use `qsTr()`, `qsTrId()`, or the project translation mechanism for user-visible text.
- Do not concatenate translatable sentences.
- Use placeholders in translated strings.
- Text must handle long strings, empty strings, elision, wrapping, and language length differences.
- Do not hardcode business states as English text.
- Prefer theme or design tokens for font family, size, and weight.

Recommended:

```qml
Label {
    text: qsTr("Installed games: %1").arg(root.installedCount)
    elide: Text.ElideRight
    maximumLineCount: 1
}
```

Recommended plural form:

```qml
Label {
    text: qsTr("%n file(s) selected", "", root.selectedCount)
}
```

Forbidden:

```qml
Label {
    text: "Installed games: " + root.installedCount
}
```

---

## 15. Images, Resources, and Visual Performance

**Must**:

- Do not reference resources through machine-local absolute paths.
- Use `qrc:/`, module resource paths, or the project resource path convention.
- Large images must set a suitable `sourceSize` or be generated at the right size by the asset pipeline.
- Prefer async loading for remote, non-first-screen, or large images.
- Handle error states for Image / AnimatedImage.
- Do not keep full-size large images displayed as small thumbnails.

**Should**:

- Use image caching carefully for frequently changing images.
- Use `clip: true`, `layer.enabled: true`, and large-area opacity carefully.
- Measure SVG, Shader, blur, and shadow effects.
- Prefer a shared icon component or themed resources for icons.

Recommended:

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

## 16. Error Handling and Diagnostics

**Must**:

- Do not commit meaningless `console.log()`.
- `console.warn()` / `console.error()` must include enough context to locate the issue.
- Handle failures for Loader, Image, dynamic creation, and async components.
- Do not write empty `catch` blocks.
- Do not silently swallow errors and continue as if everything succeeded.
- Centralize stable error codes, diagnostic fields, and trace payloads; do not scatter strings.

Recommended:

```qml
try {
    root.applyConfig(config);
} catch (error) {
    console.error("Failed to apply config:", error);
    root.errorMessage = qsTr("Failed to apply configuration.");
}
```

Forbidden:

```qml
try {
    doSomething();
} catch (error) {
}
```

---

## 17. Performance Rules

### 17.1 Must Follow

- Do not perform heavy computation in bindings.
- Do not synchronously create large numbers of objects in `Component.onCompleted`.
- Do not perform network requests, file reads, or large loops inside delegates.
- Do not animate properties that trigger broad layout cascades.
- Do not repeatedly allocate temporary objects, arrays, or regular expressions on hot paths.
- Do not create binding loops between `width` / `implicitWidth`, parent/child sizes, or Layouts and anchors.
- Load what the user sees first; defer non-critical content.

### 17.2 Should Use

- Use the QML Profiler to locate startup, binding, creation, and rendering bottlenecks.
- Use `Qt.callLater()` to coalesce high-frequency updates into the next event-loop turn.
- Use Loader to defer complex non-first-screen areas.
- If data can be provided by the model, do not rescan the model in delegates.
- Move complex sorting, filtering, and aggregation into the model layer.

Recommended:

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

Binding-loop anti-example:

```qml
Item {
    implicitWidth: width + 12
    width: implicitWidth
}
```

---

## 18. QML Commenting Notes

The complete commenting rules are owned by `CPP_Code_Comment_Guidelines.md`. This section only adds QML-specific boundaries that are easy to remove or write incorrectly.

**Must**:

- Explain external dependencies for stable QML contract fields such as model roles, restore keys, trace payloads, and diagnostic fields.
- When states, bindings, delegates, or Loader lifetimes have non-obvious timing or side effects, comment on why the code is written that way; do not explain syntax.
- When accessibility, input focus, or dynamically created object lifetimes are not obvious, document the stable boundary and failure behavior.

**Forbidden**:

- Do not add comments for simple properties, simple bindings, obvious signals, or ordinary handlers one by one.
- Do not use comments to restate QML syntax or literal code behavior.

Recommended:

```qml
// Stable role consumed by tests and saved view state.
required property string restoreKey

// Keep visible derived from opacity so the exit transition can fade out.
visible: opacity > 0
```

---

## 19. Tools and Verification

Before committing, run the project-provided equivalent commands.

Example:

```bash
qmlformat -i path/to/file.qml
qmllint path/to/file.qml
qmltestrunner
```

For CMake projects:

```bash
cmake --build build
ctest --test-dir build --output-on-failure
```

**Must**:

- No unexpected formatting differences after `qmlformat`.
- No new `qmllint` warnings.
- QML tests pass.
- UI component changes cover key states: default, hover/focus, pressed, disabled, empty, error, and loading.
- Performance-sensitive changes are verified with the QML Profiler or an equivalent project tool.

---

## 20. Pre-Commit Checklist

```text
- [ ] File is UTF-8 without BOM
- [ ] Component file names use PascalCase
- [ ] Import order is correct, with no unnecessary version pinning
- [ ] Reusable components / delegates use pragma ComponentBehavior: Bound according to the project baseline
- [ ] Root object uses id: root
- [ ] Object declaration order is consistent
- [ ] Indentation is 4 spaces and each property is on its own line
- [ ] Concrete property types are used instead of var when possible
- [ ] Required external inputs use required property
- [ ] Delegate roles are declared explicitly with required property
- [ ] Cross-object access uses id.property
- [ ] Derived values use readonly property
- [ ] property alias does not expose internal objects
- [ ] Bindings have no side effects
- [ ] No accidental imperative binding breakage
- [ ] No binding loops
- [ ] anchors and Layout are not mixed on the same item
- [ ] Components provide reasonable implicitWidth / implicitHeight
- [ ] Custom Controls prefer contentItem / background / palette
- [ ] Large lists use View, not Repeater
- [ ] Delegates are lightweight and do not perform I/O or heavy computation
- [ ] Connections uses function onXxx() syntax
- [ ] Loader / Image / dynamic creation handles error states
- [ ] User-visible text uses qsTr / qsTrId or the project translation mechanism
- [ ] Interactive components are keyboard reachable and have visible focus
- [ ] Custom interactive controls provide suitable Accessible information
- [ ] No meaningless console.log
- [ ] No new qmllint warnings
- [ ] No unexpected qmlformat differences
- [ ] QML tests pass
```

---

**Package version**: v1.0.7
**Last updated**: 2026-07-07

---
