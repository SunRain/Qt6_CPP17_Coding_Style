# AI Coding Behavior and Execution Boundaries

[English](./AI_CODING_BEHAVIOR.md) | [Simplified Chinese](../cn/AI_CODING_BEHAVIOR.md) |
[Source](../AI_CODING_BEHAVIOR.md)

> This document constrains how an AI/agent applies project authorities. It does not redefine or
> enlarge C++/Qt/QML rules, build configuration, documentation tooling, version metadata, or task scope.

> **Project-policy prerequisite:** The rule that every `QObject` subclass must contain `Q_OBJECT` is
> treated here as a project gate only when the project authority or an explicit decision has published
> it. It is not a universal Qt rule.

## 1. Responsibility and authority

The AI/agent must determine scope, facts, authority, and verification before editing code or comments.
The authority relationship is:

1. `Qt6_CPP17_Coding_Style.md` is the C++/Qt lifetime, threading, and formatting baseline.
2. `Qt6_QML_Coding_Style.md` owns QML-specific additions and must explicitly reference the comment
   guideline instead of duplicating common comment rules.
3. `Qt_Macro_Layout_Coding_Style.md` owns the placement and order of Qt, QML, and moc macros; it does
   not decide whether a project requires `Q_OBJECT`.
4. `Qt6_KDE_API_Parameter_Style.md` owns public API parameter, ownership, view-lifetime, and
   QML/meta-object boundary rules.
5. `CPP_Code_Comment_Guidelines.md` owns comment content, coverage, documentation profiles, and
   documentation verification boundaries.
6. This document only explains how the AI applies those authorities. Report conflicts and modify an
   authority only in an explicitly authorized documentation-revision task.

## 2. Classification and decision priority

### 2.1 Mandatory rules

Mandatory rules are:

1. Qt/C++ technical requirements, the target language standard, build constraints, and public API/ABI
   contracts.
2. Project policies explicitly published as `must`, `must not`, or an executable gate.

When a requested implementation violates a technical requirement or a published gate, explain the
conflict and provide a compliant alternative. If the task explicitly authorizes changing the gate,
revise the authority first and then update the implementation; never silently generate an exception.

### 2.2 The `Q_OBJECT` project gate

When the project authority explicitly says that every `QObject` subclass must contain `Q_OBJECT`:

1. Treat it as a project-specific gate, not a Qt or KDE universal fact.
2. "Every" covers public, protected, private, and internal subclasses unless the authority records a
   formal scope and exception.
3. Add the macro to new or modified subclasses and verify moc/AUTOMOC, compilation, linking, and any
   relevant runtime meta-object behavior.
4. A tree-wide governance task audits all in-scope subclasses. A local task does not expand to a whole
   repository without authorization, but it reports missing macros in its scope.
5. Template, nested, header-only, or other moc/build-limited types cannot be silently skipped. Record a
   technical exception with reason, owner, and removal condition, or change the design so the gate is
   executable.
6. Macro placement and order remain governed by `Qt_Macro_Layout_Coding_Style.md`.

If the project has not published this gate, apply the normal Qt split: a subclass that uses its own
signals, slots callable through the meta-object, properties, enum metadata, QML/plugin registration,
or another self meta-object service requires `Q_OBJECT`; other subclasses are covered by a strong Qt
upstream recommendation, not an automatic whole-tree edit.

### 2.3 `QObject` lifetime

Do not describe the policy as "all QObject objects must never be deleted directly":

- Direct cross-thread deletion and synchronous deletion while the object processes a received event
  must be prevented.
- Prefer parent ownership when a dynamic child has the same lifetime as its owner.
- Direct destruction or controlled RAII can be valid for a same-thread, local, non-escaping, unparented
  object with no event being processed and no in-flight use.
- Use `deleteLater()` for event-driven, cross-thread, queued, or explicitly asynchronous worker
  destruction, and only when the object's thread can process deferred-delete events.
- A queued connection or asynchronous callback alone does not prove that `deleteLater()` is required.
- If owner, thread, or event-loop conditions cannot be proven in the task scope, preserve the existing
  lifetime and report the uncertainty rather than rewriting it in bulk.

`QObject` subclasses must not be copied, moved, or stored by value in containers that require those
semantics.

### 2.4 Optional recommendations

Qt/KDE upstream recommendations and modern C++ techniques guide new code but are not violations unless
the project explicitly promotes them to a gate:

- Respect the target standard, public ABI/API, QML/meta-object boundaries, and existing contracts.
- Preserve existing interfaces in maintenance work; do not refactor merely to adopt a recommendation.
- `std::span` is a non-owning contiguous-memory view in C++20, not a container. Check the borrowed
  lifetime.
- `std::variant` is a tagged union, not a container.
- C++20 makes many `std::vector` operations usable during constant evaluation, but a non-empty vector
  object is generally not a persistent constant expression. Prefer `std::array` for fixed data.
- Explain `constexpr`, structured bindings, concepts, ranges, coroutines, modules, and `<=>` only when
  they carry non-obvious semantics.

### 2.5 Project-specific conventions

Existing code is evidence of local style, not authority. Code frequency cannot create a new policy.
New conventions enter an authority only through an explicit project decision or an authorized document
revision. When authorities conflict, follow the highest current authority and report the conflict.

### 2.6 Conflict order

Within the task scope, resolve conflicts in this order:

1. Explicit user goal, file scope, and verification scope.
2. Qt/C++ technical requirements, target standard, and public API/ABI contracts.
3. Published project gates, including a published all-`QObject` `Q_OBJECT` gate.
4. The applicable topic guideline and documentation profile.
5. Existing local style.
6. Optional Qt/KDE recommendations.

## 3. AI/agent execution flow

1. **Scope first:** identify the requested files, code paths, and verification boundary.
2. **Technical boundary:** read the language standard, Qt version, build configuration, public API/ABI,
   and project authorities.
3. **`Q_OBJECT` decision:** apply the project gate when enabled; otherwise apply only Qt's technical
   conditions and upstream recommendations.
4. **moc/build check:** do not stop at adding macro text; verify generated code, compilation, and linking.
5. **Lifetime check:** inspect parent, owner, thread affinity, event processing, in-flight use, and
   event-loop availability before choosing parent ownership, RAII, direct destruction, or `deleteLater()`.
6. **Profile check:** read `.qdocconf`, `Doxyfile`, KApiDox, Doxyqml, ECM, and project entry points.
   With no confirmed configuration, use ordinary source comments and do not add a toolchain.
7. **Comment increment:** add only applicable non-obvious information for new or changed
   public/protected/QML-facing contracts and internal constraints.
8. **No invented facts:** versions, dates, issue IDs, owners, replacement APIs, defaults, thread models,
   and technical exceptions require project evidence.
9. **Verify and stop:** run only scope-matched checks. Report an unconfigured documentation toolchain;
   never claim zero-warning documentation without evidence.

QCH must use the selected QDoc or Doxygen base engine. Use ECMAddQch only when the project already
depends on ECM; do not switch engines or introduce full KApiDox solely to obtain `.qch` output.

## 4. Practical examples

### 4.1 New code may use a modern return type

When C++17 is enabled and an empty value really means "no valid color":

```cpp
std::optional<QColor> parseColorFromConfig(const QString &key)
{
    const QString colorString = readConfigValue(key);
    if (colorString.isEmpty()) {
        return std::nullopt;
    }

    const QColor color(colorString);
    if (!color.isValid()) {
        return std::nullopt;
    }

    return color;
}
```

Do not add comments that merely restate the code.

### 4.2 Preserve a legacy API

For an existing `bool` plus output-parameter contract, do not change it to `std::optional` merely
because the latter is recommended:

```cpp
bool MainWindow::loadConfig(QString *errorMsg)
{
    if (!m_configFile.exists()) {
        *errorMsg = QStringLiteral("Config file not found");
        return false;
    }

    if (!validateConfigSchema()) {
        *errorMsg = QStringLiteral("Config schema validation failed");
        return false;
    }

    // ...
}
```

### 4.3 A published `Q_OBJECT` gate

When the gate is enabled, even an internal subclass with no signal or property receives the macro:

```cpp
class InternalWorker : public QObject
{
    Q_OBJECT

public:
    explicit InternalWorker(QObject *parent = nullptr);
};
```

Verify that the type is in scope and that moc/AUTOMOC, compilation, and linking support it. Reference
a formal project exception for types that cannot be processed; do not silently omit the macro.

### 4.4 A real asynchronous lifetime

Dynamic children use parent ownership. A worker moved to another thread uses `deleteLater()` from its
finished signal, provided that its thread still processes deferred-delete events:

```cpp
QWidget *createWidget(QWidget *parent)
{
    auto *widget = new QWidget(parent);
    widget->setObjectName(QStringLiteral("editorPane"));
    return widget;
}

void bindWorkerLifetime(Worker *worker)
{
    // The worker thread must still process deferred-delete events.
    QObject::connect(worker, &Worker::finished, worker, &QObject::deleteLater);
}
```

## 5. What the AI should and should not do

### Should

1. Lock scope first, then apply technical requirements, project gates, and topic guidelines.
2. When a gate is enabled, add `Q_OBJECT` to every applicable subclass in the task scope and verify moc/build.
3. Distinguish Qt technical requirements, upstream recommendations, and project policy.
4. Protect public API/ABI, QML/meta-object contracts, and verified lifetimes.
5. Use only real versions, tracking data, thread models, and documentation profiles.
6. Close out with validation that was actually executed.

### Should not

1. Derive policy from code-frequency statistics.
2. Promote a recommendation to a gate without authorization or add `Q_OBJECT` to the whole tree when
   the gate is not enabled.
3. Exclude private/internal subclasses from an enabled all-subclass gate without a formal exception.
4. Treat every direct destruction as invalid or mechanically change every asynchronous path to
   `deleteLater()`.
5. Invent versions, dates, issue IDs, owners, replacement APIs, or technical exceptions.
6. Automatically modify authorities, introduce a generator, synchronize distributions, or expand a
   refactoring scope.

## 6. Related documents, verification, and version coupling

Related authorities:

- [`Qt6_CPP17_Coding_Style.md`](./Qt6_CPP17_Coding_Style.md)
- [`Qt6_QML_Coding_Style.md`](./Qt6_QML_Coding_Style.md)
- [`Qt_Macro_Layout_Coding_Style.md`](./Qt_Macro_Layout_Coding_Style.md)
- [`Qt6_KDE_API_Parameter_Style.md`](./Qt6_KDE_API_Parameter_Style.md)
- [`CPP_Code_Comment_Guidelines.md`](./CPP_Code_Comment_Guidelines.md)
- [`Qt6_CPP17_CLANG-FORMAT`](./Qt6_CPP17_CLANG-FORMAT)

Before delivery, check the task-appropriate subset of:

- The in-scope `QObject` subclasses contain `Q_OBJECT` when the published gate is enabled, or cite a
  formal exception.
- moc/AUTOMOC, compilation, linking, and relevant runtime meta-object checks pass.
- Comments and examples follow the 100-column soft baseline, `ReflowComments: false`, and the project's
  function-brace style.
- Documentation checks run only when a matching configuration and task requirement exist.
- Unconfigured generators are reported as unconfigured; no zero-warning claim is made.
- Root, `cn/`, and `en/` documents, links, versions, and dates are coupled only during an explicit
  documentation synchronization or release task.

---

**Document package version**: v1.1.0
**Last updated**: 2026-07-25
