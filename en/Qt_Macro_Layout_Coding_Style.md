# Qt Macro Layout Coding Style

English | 简体中文 | Source

> Note: This document is the English translation of the current Qt macro layout guideline. If there is any discrepancy, the package baseline prevails.

This guideline belongs to `Qt6_CPP17_Coding_Style.md` and expands its Qt meta-object / QML / moc macro-layout topic.

Scope: Qt 6 / KDE-style C++ code, especially public headers, QML toolkits, and reusable component libraries consumed downstream.

Authority boundaries:
- Macro placement, class-body layout, QML registration macros, metatype macros, d-pointer macros, and logging macros are owned by this document.
- Public API parameter types, Borrow / Owning, view lifetimes, and QML/meta-object boundary types are owned by `Qt6_KDE_API_Parameter_Style.md`; this document only references those conclusions.
- Baseline formatting, naming, lifetime, threading, and error handling are owned by `Qt6_CPP17_Coding_Style.md`.

## 1. Core Principles

- Keep Qt meta-object macros at the top of the class body: declare the Qt/QML contract first, then enter the C++ `public:` API.
- Public libraries, toolkit headers, ordinary app code, and internal code all use keyword-free macro spelling: `Q_SIGNALS:`, `public Q_SLOTS:`, and `Q_EMIT`.
- `Q_PROPERTY` is a meta-object/QML contract. It is not governed by C++ `public/private` access control, so place it before `public:`.
- Put `Q_DISABLE_COPY` / `Q_DISABLE_COPY_MOVE` in `private:` so disabled copy/move declarations do not read as business-level public API.
- `Q_ENUM` / `Q_FLAG` must immediately follow the matching enum / flags declaration.
- Macro placement should express Qt semantic boundaries. Line wrapping and indentation are handled by the project `.clang-format`.

## 2. Recommended Class Layout Template

```cpp
class Example : public QObject
{
    Q_OBJECT
    QML_NAMED_ELEMENT(Example)
    Q_CLASSINFO("DefaultProperty", "content")
    Q_PROPERTY(QString title READ title WRITE setTitle NOTIFY titleChanged FINAL)
    Q_PROPERTY(bool enabled READ enabled WRITE setEnabled NOTIFY enabledChanged FINAL)

public:
    enum class Mode {
        Compact,
        Expanded,
    };
    Q_ENUM(Mode)

    explicit Example(QObject *parent = nullptr);
    ~Example() override;

    QString title() const;
    void setTitle(const QString &title);

    bool enabled() const;
    void setEnabled(bool enabled);

    Q_INVOKABLE void refresh();

public Q_SLOTS:
    void reset();

Q_SIGNALS:
    void titleChanged();
    void enabledChanged();

protected:
    bool event(QEvent *event) override;

private:
    Q_DISABLE_COPY_MOVE(Example)

    QString m_title;
    bool m_enabled = true;
};
```

## 3. Core Macro Placement Rules

| Macro | Recommended position | Notes |
|---|---|---|
| `Q_OBJECT` | First line inside a `QObject`-derived class body | Must appear before `public:` as the meta-object entry point |
| `Q_GADGET` / `Q_GADGET_EXPORT` | First line inside a non-`QObject` value type class body | For enums, properties, and QML value types |
| `Q_NAMESPACE` / `Q_NAMESPACE_EXPORT` | Top of the namespace | For namespace-level enum meta-objects |
| `QML_ELEMENT` / `QML_NAMED_ELEMENT` | Immediately after `Q_OBJECT` / `Q_GADGET` | Declares the QML registration contract |
| `QML_SINGLETON` | QML registration macro area | Keep it adjacent to `QML_NAMED_ELEMENT` |
| `QML_UNCREATABLE` | QML registration macro area | Explains that the type is QML-visible but not constructible |
| `QML_ATTACHED` | QML registration macro area | Attached property provider goes near the top of the class |
| `Q_CLASSINFO` | After QML registration macros and before `Q_PROPERTY` | Often used for meta information such as `DefaultProperty` |
| `Q_INTERFACES` | Class top area after `Q_OBJECT` | For plugin/interface implementation classes |
| `Q_PLUGIN_METADATA` | After `Q_OBJECT`, adjacent to `Q_INTERFACES` | For plugin implementation classes |
| `Q_PROPERTY` | Centralized at the class top, before `public:` | QML/meta-object API contract |
| `Q_ENUM` | Immediately after the enum declaration | Do not separate it from the corresponding enum |
| `Q_FLAG` | Immediately after `Q_DECLARE_FLAGS` | Do not separate it from the corresponding flags |
| `Q_ENUM_NS` / `Q_FLAG_NS` | Immediately after namespace enum / flags declaration | Namespace enum meta-object |
| `Q_DECLARE_FLAGS` | After the flag enum | Usually placed directly after the enum in the class body |
| `Q_DECLARE_OPERATORS_FOR_FLAGS` | Outside the class, inside the namespace or near the header end | Must not be placed inside the class body |
| `Q_INVOKABLE` | Before public/protected method declarations | QML-callable APIs usually belong in `public:` |
| `Q_SIGNALS:` | After public API and before `private:` | Unified replacement for `signals:` |
| `public Q_SLOTS:` | After public methods and before `Q_SIGNALS:` | Public slot callable through QML/Qt meta-call paths |
| `protected Q_SLOTS:` | Protected API area | Slot available to subclasses |
| `private Q_SLOTS:` | Private area, before or after ordinary private methods | Only when internal meta-object invocation is needed |
| `Q_EMIT` | At signal emission sites inside function bodies | Unified replacement for bare `emit` |
| `Q_DISABLE_COPY` | Top of `private:` | Disables copy only |
| `Q_DISABLE_MOVE` | Top of `private:` | Disables move only |
| `Q_DISABLE_COPY_MOVE` | Top of `private:` | Preferred for `QObject`-derived classes |
| `Q_DECLARE_PRIVATE` / `Q_DECLARE_PUBLIC` | `private:` | Internal bridge for the d-pointer pattern |
| `Q_D` / `Q_Q` | Near the beginning of function bodies | Used inside d-pointer functions |
| `Q_DECLARE_METATYPE` | After the complete type definition, near the header end | Namespace types normally use fully qualified names outside the namespace |
| `Q_DECLARE_TYPEINFO` | After the complete type definition | Declares value-type performance / move semantics |
| `Q_DECLARE_INTERFACE` | After the full interface class definition | Near the end of plugin interface headers |
| `Q_DECLARE_LOGGING_CATEGORY` | Outside classes in headers, inside the namespace | Declares a logging category |
| `Q_LOGGING_CATEGORY` | After includes in `.cpp`, inside a namespace or anonymous namespace | Defines a logging category |
| `Q_DECLARE_TR_FUNCTIONS` | Top of a non-`QObject` class body, before `public:` | Use when a translation context is needed |
| `Q_MOC_INCLUDE` | After includes and before class declarations | Adds includes for moc only |
| `Q_REVISION` | Next to the corresponding property/method/signal | For versioned QML APIs |
| `Q_PRIVATE_SLOT` | `private:` area | Bridges private d-pointer slots |
| `Q_UNUSED` | Near the beginning of function bodies | Only when the parameter name cannot be removed |
| `Q_ASSERT` / `Q_ASSERT_X` | Preconditions or invariants inside function bodies | Do not use for user-input error handling |
| `Q_FALLTHROUGH` | At intentional switch fallthrough points | Write it exactly where fallthrough happens |
| `Q_UNREACHABLE` | End of theoretically unreachable branches | Use with care |

## 4. `Q_PROPERTY` Rules

Recommended:

```cpp
class ThemeProvider : public QObject
{
    Q_OBJECT
    QML_ELEMENT
    Q_PROPERTY(QString themeName READ themeName WRITE setThemeName NOTIFY themeNameChanged FINAL)
    Q_PROPERTY(bool darkMode READ darkMode WRITE setDarkMode NOTIFY darkModeChanged FINAL)

public:
    explicit ThemeProvider(QObject *parent = nullptr);
};
```

Requirements:

- Keep `Q_PROPERTY` declarations centralized at the class top, after `Q_OBJECT` / `QML_*` / `Q_CLASSINFO`.
- Do not scatter `Q_PROPERTY` declarations near their getters/setters.
- Do not add obvious comments to every simple property. Comment only when defaults, side effects, stability, or failure behavior are not obvious.
- QML/meta-object boundaries prefer owning types such as `QString`, `QUrl`, and `QVariant`; do not expose `QStringView` / `QAnyStringView` / `QByteArrayView`.
- Prefer `FINAL` for QML-facing, public stable properties that are explicitly not intended to be overridden. Do not force `FINAL` for properties that need subclass extension, test-double overrides, or are still evolving as API.
- If property getters/setters are QML public API, place them in `public:`.

## 5. `Q_DISABLE_COPY` / `Q_DISABLE_COPY_MOVE` Rules

Recommended:

```cpp
class RuntimeCore : public QObject
{
    Q_OBJECT

public:
    explicit RuntimeCore(QObject *parent = nullptr);

private:
    Q_DISABLE_COPY_MOVE(RuntimeCore)

    QVariantMap m_state;
};
```

Requirements:

- `QObject`-derived classes are non-copyable and non-movable by default.
- Prefer `Q_DISABLE_COPY_MOVE(Class)` in Qt6 projects.
- If the project Qt baseline does not provide `Q_DISABLE_COPY_MOVE`, use:

```cpp
private:
    Q_DISABLE_COPY(ClassName)
    ClassName(ClassName &&) = delete;
    ClassName &operator=(ClassName &&) = delete;
```

- Do not place it in `public:` unless the project has an explicit and consistent existing convention.
- Place it at the top of `private:`, before private member variables.

## 6. `Q_SIGNALS:` Rules

Recommended:

```cpp
Q_SIGNALS:
    void titleChanged();
    void requestFailed(const QString &reason);
```

Requirements:

- All Qt C++ code uses `Q_SIGNALS:`; do not use `signals:`.
- Place it after public API and slots, and before `private:`.
- Signal parameters must be suitable for the Qt meta-object boundary. Use owning types across QML or queued connections.
- Signal names express facts that already happened, such as `titleChanged()`. Do not use command-style names.
- Signals usually do not need an access specifier. `Q_SIGNALS:` itself declares the Qt signal area.

## 7. `Q_SLOTS` Rules

Recommended spelling:

```cpp
public Q_SLOTS:
    void reset();

protected Q_SLOTS:
    void refreshFromBackend();

private Q_SLOTS:
    void handleTimeout();
```

Requirements:

- All Qt C++ code uses `Q_SLOTS`; do not use `slots`.
- `public Q_SLOTS:` is for QML, old-style meta-calls, or slots that external objects need to invoke.
- `protected Q_SLOTS:` is for subclass-extensible or reusable slots.
- `private Q_SLOTS:` is only for internal Qt meta-object invocation.
- If a method is only a receiver for modern function-pointer `connect()`, it does not need to be declared as a slot. Use an ordinary private method.
- Place slot sections after ordinary public/protected/private methods, before `Q_SIGNALS:` or inside the corresponding access section.

## 8. `Q_EMIT` Rules

Recommended:

```cpp
void Example::setTitle(const QString &title)
{
    if (m_title == title) {
        return;
    }

    m_title = title;
    Q_EMIT titleChanged();
}
```

Requirements:

- All Qt C++ code uses `Q_EMIT`; do not write bare `emit`.
- `Q_EMIT` only appears inside function bodies.
- Do not emit a changed signal when state did not change.
- Update object state before emitting the signal, unless the signal explicitly means "about to happen".

## 9. enum / flags Macro Rules

Recommended:

```cpp
class ManualFocusNav : public QObject
{
    Q_OBJECT

public:
    enum Direction {
        Up,
        Down,
        Left,
        Right,
    };
    Q_ENUM(Direction)

    enum BubbleDirectionFlag {
        BubbleUp = 1 << Up,
        BubbleDown = 1 << Down,
    };
    Q_DECLARE_FLAGS(BubbleDirections, BubbleDirectionFlag)
    Q_FLAG(BubbleDirections)
};

Q_DECLARE_OPERATORS_FOR_FLAGS(ManualFocusNav::BubbleDirections)
```

Requirements:

- `Q_ENUM` immediately follows the enum.
- `Q_DECLARE_FLAGS` immediately follows the flag enum.
- `Q_FLAG` immediately follows `Q_DECLARE_FLAGS`.
- `Q_DECLARE_OPERATORS_FOR_FLAGS` is outside the class body.
- An enum exposed to QML or tests is a stable contract and needs a concise comment that explains its purpose.

## 10. QML Registration Macro Rules

Recommended:

```cpp
class ActionIdsFacade final : public QObject
{
    Q_OBJECT
    QML_NAMED_ELEMENT(ActionIds)
    QML_SINGLETON
    Q_PROPERTY(QString navUp READ navUp CONSTANT)

public:
    explicit ActionIdsFacade(QObject *parent = nullptr);
};
```

Requirements:

- `QML_ELEMENT` / `QML_NAMED_ELEMENT` immediately follows `Q_OBJECT`.
- Keep `QML_SINGLETON` adjacent to the QML name macro.
- Put `QML_UNCREATABLE` in the QML registration macro area and provide a clear reason.
- Put `QML_ATTACHED` at the top of the attached provider class.
- Put `QML_DECLARE_TYPEINFO(Type, QML_HAS_ATTACHED_PROPERTIES)` after the type definition and before the include-guard `#endif`.

## 11. Plugin and Interface Macro Rules

Recommended:

```cpp
class BackendPlugin : public QObject, public BackendInterface
{
    Q_OBJECT
    Q_PLUGIN_METADATA(IID BackendInterface_iid FILE "backend.json")
    Q_INTERFACES(BackendInterface)

public:
    explicit BackendPlugin(QObject *parent = nullptr);
};
```

Requirements:

- Put `Q_PLUGIN_METADATA` and `Q_INTERFACES` at the class top.
- Put `Q_DECLARE_INTERFACE` after the interface class definition.
- Plugin metadata file paths must be stable. Do not build them dynamically at runtime.

## 12. metatype Macro Rules

Recommended:

```cpp
namespace hlui {

struct ActionEvent
{
    QString actionId;
    bool pressed = false;
};

} // namespace hlui

Q_DECLARE_METATYPE(hlui::ActionEvent)
```

Requirements:

- `Q_DECLARE_METATYPE` must appear after the complete type definition.
- Namespace types normally use fully qualified names outside the namespace.
- If the type is used in queued connections, ensure it is copyable/destructible and register the metatype when needed.
- Do not register views or raw borrowed-lifetime objects as cross-thread payloads.

## 13. Logging Macro Rules

Header:

```cpp
Q_DECLARE_LOGGING_CATEGORY(hluiRuntime)
```

`.cpp`:

```cpp
Q_LOGGING_CATEGORY(hluiRuntime, "deckshell.hlui.runtime")
```

Requirements:

- Put `Q_DECLARE_LOGGING_CATEGORY` outside classes in headers.
- Put `Q_LOGGING_CATEGORY` after includes in `.cpp`.
- The category string is a diagnostics contract and should be stable.
- Do not define `Q_LOGGING_CATEGORY` in headers to avoid duplicate definitions.

## 14. d-pointer Macro Rules

Recommended:

```cpp
class PublicApiPrivate;

class PublicApi : public QObject
{
    Q_OBJECT

public:
    explicit PublicApi(QObject *parent = nullptr);
    ~PublicApi() override;

private:
    Q_DECLARE_PRIVATE(PublicApi)
    Q_DISABLE_COPY_MOVE(PublicApi)

    QScopedPointer<PublicApiPrivate> d_ptr;
};
```

Function body:

```cpp
void PublicApi::refresh()
{
    Q_D(PublicApi);
    d->refresh();
}
```

Requirements:

- Put `Q_DECLARE_PRIVATE` / `Q_DECLARE_PUBLIC` in `private:`.
- Put `Q_D` / `Q_Q` near the beginning of function bodies.
- Use a d-pointer only for ABI/API boundaries or complex private state. Do not introduce it for small classes without a need.

## 15. Comment Rules

- Class descriptions use `/// @brief` or concise Doxygen comments.
- QML-facing classes, roles, stable strings, trace payloads, error codes, and lifetime boundaries must be commented.
- Do not add per-item comments to simple `Q_PROPERTY`, simple signals, or ordinary getters/setters.
- Comments explain contracts, boundaries, and reasons. They should not explain Qt syntax.
- Keep each comment concise; for Chinese comments, prefer no more than 80 characters per line.

## 16. Forbidden Patterns

- Do not add or mix `signals:`, `slots:`, or bare `emit`; all Qt C++ code uses `Q_SIGNALS:`, `Q_SLOTS`, and `Q_EMIT`.
- Do not scatter `Q_PROPERTY` declarations across multiple access sections.
- Do not put `Q_DISABLE_COPY` at the class top or inside the property/meta-object area.
- Do not put `Q_DECLARE_OPERATORS_FOR_FLAGS` inside the class body.
- Do not expose borrowed view types such as `QStringView`, `QAnyStringView`, or `QByteArrayView` in signal/slot/QML invokable APIs.
- Do not move macros just to satisfy moc in a way that makes C++ API sections confusing.

## 17. Final Recommended Order

```text
class / struct declaration
  Q_OBJECT / Q_GADGET
  QML_* registration macros
  Q_CLASSINFO / Q_INTERFACES / Q_PLUGIN_METADATA
  centralized Q_PROPERTY declarations
  public:
    enum / Q_ENUM / Q_DECLARE_FLAGS / Q_FLAG
    constructors/destructor
    ordinary public API
    Q_INVOKABLE API
  public Q_SLOTS:
  protected:
  protected Q_SLOTS:
  Q_SIGNALS:
  private Q_SLOTS:
  private:
    Q_DISABLE_COPY_MOVE / Q_DISABLE_COPY
    Q_DECLARE_PRIVATE
    private helper
    member variables
outside class:
  Q_DECLARE_OPERATORS_FOR_FLAGS
  Q_DECLARE_METATYPE / Q_DECLARE_TYPEINFO / Q_DECLARE_INTERFACE
  QML_DECLARE_TYPEINFO
#endif
```
