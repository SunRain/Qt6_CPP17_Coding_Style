# AI Coding Behavior Guide

English | 简体中文 | Source

> Note: This document is the English translation of the current AI coding-behavior guideline. If there is any discrepancy, the package baseline prevails.

This document explains how the AI assistant handles **"Mandatory"** vs **"Optional Recommended"** rules in this project's coding standards.

---

## 📋 Specification Classification System

This project classifies coding standards into three tiers:

### 1️⃣ **Mandatory**

**Definition**: Rules that must be followed strictly; violations will cause code to be rejected.

**AI behavior**:
- ✅ All generated code must comply with these rules.
- ⚠️ If the user explicitly asks to violate them, the AI must refuse and provide a compliant alternative.
- 🔍 Code reviews should check these items strictly.

**Includes**:
- Naming conventions (`m_`, `s_`, `k` prefixes)
- Formatting (4-space indentation; braces required even for single-statement blocks)
- Qt 6 conventions (new-style signal/slot connections, `QStringLiteral`, `Q_OBJECT`)
- Forbidden items (exceptions, RTTI, `dynamic_cast`, raw `new`/`delete` forbidden by default; raw `new` allowed for `QObject`-derived types but must use parent ownership or `deleteLater()`; manual `delete` for `QObject` is forbidden; C-style casts)
- `QObject` value semantics: copy/move/by-value containers are forbidden; use pointer/reference semantics and manage lifetime with parent ownership / `deleteLater()` (see Chapter 6 of the coding-style guide in this directory)

**Example**:
```cpp
// ❌ Wrong: violates mandatory rule (no braces for a single statement)
if (condition)
    doSomething();

// ✅ Correct: compliant
if (condition) {
    doSomething();
}
```

---

### 2️⃣ **Optional Recommended**

**Definition**: Modern best practices that are encouraged, but not required.

**AI behavior**:
- ✅ **New code / new features**: use the recommended style by default.
- 🔄 **Maintaining old code**: keep the existing style; do not force refactors.
- 🤝 **User preference**: respect explicit choices; avoid repeated reminders.
- 💡 **First use**: you may briefly explain the benefit (once, and only once).

**Includes** (see Chapter 5 of the coding-style guide in this directory):
- `std::optional<T>` vs `bool func(T *out)`
- Structured bindings vs `QPair`
- `constexpr` vs `const`/`#define`
- `[[nodiscard]]`, `[[maybe_unused]]`
- STL containers (`std::span`, `std::variant`, etc.) vs Qt containers

**Example**:
```cpp
// ✅ Recommended: modern C++17 style
std::optional<QColor> tryGetColor() {
    if (isValid) {
        return QColor(255, 0, 0);
    }
    return std::nullopt;
}

// ✅ Also acceptable: traditional style
bool getColor(QColor *outColor) {
    if (isValid) {
        *outColor = QColor(255, 0, 0);
        return true;
    }
    return false;
}
```

---

### 3️⃣ **Project-Specific**

**Definition**: Conventions formed by the project's history and team habits.

**AI behavior**:
- 🔍 **Analyze existing code**: learn the real style used in the codebase.
- 📊 **Prefer the dominant style**: if 90% of the codebase uses a pattern, follow it.
- 🆕 **New modules**: you may introduce modern patterns in a controlled way.
- 📝 **Documentation update**: add new conventions to the relevant documents in the release package (prefer the coding-style and comment-guideline documents in this directory), and update this document's decision notes (if applicable).

---

## 🤖 AI Decision Tree

When generating code, the AI assistant follows this decision flow:

```
Start generating code
    ↓
Does it violate a mandatory rule?
    ├─ Yes → ❌ Refuse and provide a compliant alternative
    └─ No  → Continue
           ↓
Does it involve lifetime management?
           ├─ No → Continue
           └─ Yes → First decide whether it is QObject-derived
                  ├─ Yes → Forbid copy/move/by-value containers; prefer new T(parent);
                  |        use deleteLater() when needed; forbid manual delete
                  └─ No  → Use RAII/smart pointers; forbid raw new/delete
                         ↓
Is it new code or maintenance?
    ├─ New code → Prefer "Optional Recommended" modern style
    └─ Old code → Keep the existing style
          ↓
Did the user explicitly request a specific style?
    ├─ Yes → Respect the user's choice (as long as it does not violate mandatory rules)
    └─ No  → Use the recommended style
         ↓
Done
```

---

## 📊 Practical Examples

### Scenario 1: user asks for a new helper function

**User request**:
> "Write a function that tries to parse a color from a config file; return empty if it fails."

**AI decision**:
1. ✅ New code → prefer modern style
2. ✅ Use `std::optional<QColor>` as the return type
3. ✅ Use `camelCase` for function names; `k` prefix for constants
4. ✅ Use `QStringLiteral` for string literals

**Generated code**:
```cpp
#include <QColor>
#include <QString>
#include <optional>

std::optional<QColor> parseColorFromConfig(const QString &key) {
    constexpr int kDefaultAlpha = 255;

    // Try to read from config
    QString colorStr = readConfigValue(key);
    if (colorStr.isEmpty()) {
        return std::nullopt;
    }

    QColor color(colorStr);
    if (!color.isValid()) {
        return std::nullopt;
    }

    return color;
}
```

**AI note**:
> I used `std::optional<QColor>` as the return type, which is a recommended C++17 style. If you prefer the traditional `bool parseColor(const QString&, QColor*)` style, I can adjust it.

---

### Scenario 2: user asks to modify an existing function

**Existing code**:
```cpp
bool MainWindow::loadConfig(QString *errorMsg) {
    if (!m_configFile.exists()) {
        *errorMsg = QString("Config file not found");
        return false;
    }
    // ...
}
```

**User request**:
> "Add a new config validation step."

**AI decision**:
1. ✅ Maintenance → keep the existing style
2. ✅ Keep the `bool + out-parameter` pattern (do not force `std::optional`)
3. ✅ Use `QStringLiteral` for string literals (mandatory)
4. ✅ Keep `m_` prefix for member variables (mandatory)

**Generated code**:
```cpp
bool MainWindow::loadConfig(QString *errorMsg) {
    if (!m_configFile.exists()) {
        *errorMsg = QStringLiteral("Config file not found");
        return false;
    }

    // New: config validation step
    if (!validateConfigSchema()) {
        *errorMsg = QStringLiteral("Config schema validation failed");
        return false;
    }

    // ...
}
```

**AI behavior**:
- ✅ Preserves the original `bool + out-parameter` style
- ✅ Fixes the string literal style (from `QString("...")` to `QStringLiteral`)
- ✅ Does not proactively suggest refactoring to `std::optional`

---

### Scenario 3: user explicitly asks for a traditional style

**User request**:
> "Use the traditional QPair style to return an iterator and an insertion status."

**AI decision**:
1. ✅ User explicitly asked for `QPair` → respect the choice
2. ✅ Do not push structured bindings
3. ✅ Still follow mandatory rules (naming, formatting, etc.)

**Generated code**:
```cpp
QPair<QMap<QString, int>::iterator, bool> insertValue(
    QMap<QString, int> &map,
    const QString &key,
    int value)
{
    auto it = map.find(key);
    if (it != map.end()) {
        return qMakePair(it, false);
    }

    it = map.insert(key, value);
    return qMakePair(it, true);
}
```

---

### Scenario 4: user asks to violate a mandatory rule

**User request**:
> "Create a QWidget with raw new, and I'll delete it when appropriate."

**AI decision**:
1. ❌ Violates a mandatory rule (manual `delete` for `QObject`-derived types is forbidden)
2. ✅ Raw `new` is allowed for `QObject`-derived types, but destruction must be via parent ownership or `deleteLater()`

**AI response**:
> ⚠️ **I cannot generate code exactly as requested**
>
> According to the project coding rules: `QObject`-derived objects may be created with raw `new`, but **manual `delete` is forbidden**. Use the Qt parent-child tree or `deleteLater()` for event-loop-safe destruction.
>
> **Recommended alternatives**:
>
> **Option 1: use parent ownership (preferred)**
> ```cpp
> void createWidget(QWidget *parent) {
>     auto *widget = new QWidget(parent);
>     widget->setObjectName(QStringLiteral("myWidget"));
> }
> ```
>
> **Option 2: no parent and involves async/event loop → use `deleteLater()`**
> ```cpp
> void createWidget() {
>     auto *widget = new QWidget();
>     widget->setObjectName(QStringLiteral("myWidget"));
> 
>     // Destroyed by event loop to avoid manual delete
>     widget->deleteLater();
> }
> ```
>
> If you truly need immediate destruction, explain why event-loop destruction is not acceptable. I can help redesign the lifetime safely (e.g., clarify ownership, use RAII for non-QObject resources, etc.).

---

## 🎯 Key Principles Summary

### What the AI should do ✅

1. **Strictly enforce mandatory rules**: zero tolerance for violations
2. **Use modern patterns for new code**: default to "Optional Recommended"
3. **Preserve style when maintaining old code**: do not force refactors
4. **Respect explicit user choices**: the user may have valid trade-offs
5. **Educate briefly**: when using a new feature the first time (no verbosity)

### What the AI should not do ❌

1. **Do not enforce optional recommendations**: do not repeatedly push `std::optional`
2. **Do not mix styles**: avoid mixing traditional and modern styles in the same file
3. **Do not assume knowledge gaps**: explain succinctly, no lecturing
4. **Do not refactor proactively**: unless it is required to satisfy mandatory rules
5. **Do not ignore project history**: learn and follow the codebase's real conventions

---

## 📚 Related Documents

### Release Package Entry (Single Entry Point)

This directory provides a localized reading set that can be reviewed and distributed on its own.

- [`Coding Style Guide`](./Qt6_CPP17_Coding_Style.md) (optional recommendations in Chapter 5)
- bundled clang-format baseline notes for formatting consistency
- [`Comment Guidelines`](./CPP_Code_Comment_Guidelines.md)
- AI behavior and decision tree (this document)

#### Suggested Reading Order
1. Formatting conventions and bundled baseline notes
2. [`Coding Style Guide`](./Qt6_CPP17_Coding_Style.md)
3. [`Comment Guidelines`](./CPP_Code_Comment_Guidelines.md)
4. AI behavior notes (this document)

#### Link Rules
- Use relative links only, and only to files/anchors within the release package. Do not link to non-release paths or tool-specific entry names.
- If heading changes alter anchors, update all references accordingly to avoid broken links.

#### Version Coupling Rules
- The release package is maintained as a whole with a unified **Document Package Version**. All artifacts are versioned together.

---

**Document Package Version**: v1.0.6  
**Last Updated**: 2026-01-17
