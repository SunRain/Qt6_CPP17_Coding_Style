# Qt6 / KDE Shared-Library Public API Parameter Style
(`QString` owning + view borrow)

English | 简体中文 | Source

> Note: This document is the English translation of the current Qt6 / KDE API parameter guideline. If there is any discrepancy, the package baseline prevails.

Scope: exported Qt6/KF6 shared-library APIs (public headers / exported symbols), plus interface layers that include QML bindings (`Q_PROPERTY` / `Q_INVOKABLE` / signals/slots).  
Goal: keep public interfaces clear and maintainable; keep internal implementations performant and simple; allow view-based borrow APIs to be introduced gradually while avoiding overload traps in public headers.

Version / prerequisites (recommended to state explicitly in the project):
- Language: C++17 (`std::span` mentioned in this document only applies to C++20; C++17 projects should avoid exposing `std::span` in Public API, see 1.1)
- Qt/KF: Qt 6 / KF 6 (support for view types differs across minor versions; confirm the actual project baseline before introducing `QAnyStringView`, `QUtf8StringView`, etc.)

---

## 0. Terms and Core Principles

### Terms
- **Public API**: installed headers and exported symbols consumed by downstream users.
- **Internal API**: implementation-only `.cpp`, private headers, and non-exported layers.
- **Borrow**: the argument is only read/parsed during the current call; it is not stored and not used asynchronously.
- **Owning**: the argument is stored in members/caches, or used across calls / asynchronously.
- **View**: non-owning views such as `QStringView`, `QAnyStringView`, and `QByteArrayView`.
- **Qt Meta-Object boundary**: any interface exposed through `moc` and potentially invoked via `QMetaObject` (`signals`, `slots`, `Q_INVOKABLE`, `Q_PROPERTY` notifier-related functions, etc.).

### Core principles (mandatory)
- Decide the semantics first: **Borrow** or **Owning**. Do not let the parameter type decide the semantics for you.
- Public API must not extend the lifetime of **Borrow** inputs across calls (views, borrowed pointers, borrowed references; storing in members/caches/queues or capturing in delayed lambdas all count as extending the lifetime).
- If you need to store it / use it asynchronously / use it across threads, treat it as **Owning** immediately.
- Treat Qt Meta-Object boundaries as Owning by default: do not propagate non-owning views through signal/slot, `Q_INVOKABLE`, or QML call chains.

> Note: this does **not** mean “all pointer/reference parameters are forbidden.” For example, `QObject*` observer pointers and parent/child relationships follow a different contract (ownership, thread affinity, invalidation rules) and must be documented separately. But they still must not be stored across calls if they are only borrowed inputs.

### 0.1 Public API parameter quick decision tree (recommended)

> Goal: with minimal rules, make “semantics + boundary + compatibility” land in an implementable signature choice.

**First: determine whether this crosses a Qt Meta-Object / QML boundary (mandatory)**
- If the function/signal/property interface goes through `moc` (`signals` / `slots` / `Q_INVOKABLE` / `Q_PROPERTY` READ/WRITE/NOTIFY, etc.):
  - Prefer owning types for parameters/return values (for example `QString` / `QByteArray` / `QUrl` / `QVariant`)
  - Avoid exposing view types on that boundary; avoid same-name overloads where possible (see 1.4 and 5.6)
  - Clarification: the point here is that the *type* must be owning and safely copyable/convertible. This does not force pass-by-value vs `const&`, but view/borrowed types must not appear on that boundary.

**Second: decide semantics: Borrow vs Owning (must come first)**
- Borrow: only read/parse/compare during the call; do not store; do not use async; do not cross event loops/threads.
- Owning: stored in members/caches/queues, or used across calls/event loops/threads/async/delayed execution.

**Finally: pick parameter types based on semantics (Public C++ API)**
- Borrow text/bytes: prefer a **single view entry point**, and **pass by value**
  - Text: `foo(QAnyStringView)` / `foo(QStringView)` / `foo(QUtf8StringView)` / `foo(QLatin1StringView)`
  - Bytes: `foo(QByteArrayView)`
  - Note: these are mutually exclusive candidates. For a single semantics, pick exactly one as the canonical entry point; do not implement all of them as same-name overloads. If you need multiple encodings, prefer different function names (for example `setNameUtf8(...)` / `setNameLatin1(...)`, see 5.3).
- Owning text/bytes (stored/async): the default public API starting point is `const QString&` / `const QByteArray&`
  - You may offer `setNameView(QAnyStringView)` as a convenience entry point, but it must convert to owning immediately and forward to a single implementation point (see Chapter 2)
- Pure C++ (non meta-object) APIs that want “single entry + natural moves”: sink style `QString name` + `std::move(name)` is optional (see 2.1), but do not present sink style as the mainstream default for Qt/KDE public setters.

### 0.2 Non-text parameters: by value vs `const&` (Public API general rule)

> Note: whether a type is implicitly shared affects the “copy cost model”, but signatures should express semantics first. For most non-text inputs, follow the basic rule: pass small values by value; pass large objects by `const&`.

**Rules (mandatory)**
- Small scalars / enums / flags: pass by value (do not write `const T&`)
  - `bool`, integers, `double`, pointers/handles
  - small `enum`, `QFlags`
  - small value types such as `std::chrono::duration` / `std::chrono::time_point`
- Large objects / containers / complex structures: prefer `const T&` for read-only inputs
  - for example `QSet<T>`, `QHash<K,V>`, `QMap<K,V>`, `QVector<T>`, `QList<T>`, or custom large structs
- View types (such as `QAnyStringView` / `QStringView` / `QByteArrayView`): must be passed by value; do not use `const View&` (see Chapter 1)

**Example**
```cpp
void setEnabled(bool enabled);                      // by value
void setMode(Mode mode);                            // enum by value
void setTimeout(std::chrono::milliseconds timeout); // small chrono by value

void setTags(const QSet<QString> &tags);            // const&
```

---

## 1) View: read-only / parsing entry points (recommended to pass by value)

### 1.1 Choosing the view type (Public C++ API)

| Scenario | Recommended parameter type | Notes |
|---|---|---|
| Text input (general-purpose, multiple sources, unspecified encoding) | `QAnyStringView text` | Good as a public entry parameter. It can be constructed from multiple string sources, and converted to `QString` when needed. |
| Text input (explicit UTF-16 / requires `QChar` semantics) | `QStringView text` | Good for algorithm / parsing entry points (for example, when chaining many `QStringView` operations). |
| Text input (explicit UTF-8, and the API wants to lock in the encoding) | `QUtf8StringView text` | Use only when the protocol / format is explicitly UTF-8. |
| Text input (explicit Latin1) | `QLatin1StringView text` | Use only when the protocol or historical context explicitly requires Latin1. |
| Binary / byte input | `QByteArrayView bytes` | Replaces `(const char*, size_t)`; only suitable for Borrow scenarios. |
| Non-Qt container slices | (avoid) If unavoidable: a project-wide span type (C++17) / `std::span<const T> items` (C++20) | Use only for very stable and simple slice inputs; avoid bringing large template systems into public headers. |

> Note: in Qt 6, `QLatin1String` and `QLatin1StringView` are the same type. The old name exists mainly for compatibility. Prefer the `QLatin1StringView` name in new code.

**Rules (mandatory)**
- Pass view parameters by value: `foo(QAnyStringView s)`. Do not write `const QAnyStringView&`.
- Use views only for Borrow semantics. Any scenario that may outlive the call must convert to an owning type.

---

### 1.2 `QAnyStringView` vs `QStringView`: how to choose (public API guidance)

**Recommended (Public API)**
- For boundary entry points (accepting “user input / config / protocol fields / command-line arguments / log keywords / etc.”), prefer `QAnyStringView`.
  - Purpose: downstream callers should not have to construct a `QString` just to call the API.
  - Cost: if the implementation needs extensive UTF-16 semantics internally, it will usually do one owning conversion.

**Recommended (Internal / algorithm layer)**
- For algorithm / parsing functions (lots of substring / trim / scan / `toInt` / etc., and you want uniform UTF-16 semantics), prefer `QStringView`.
  - Purpose: unify semantics and implementation; avoid dealing with multiple encodings inside algorithm code.

**Rules (mandatory)**
- Do not provide both `QStringView` and `QAnyStringView` overloads for the same semantics “just to support more input types” (see Chapter 5 on overload control).

---

### 1.3 Lifetime rules: what is allowed, what is absolutely forbidden

#### Allowed (Borrow scenarios)
- Perform read-only checks / parsing / comparisons inside the function body: `startsWith`, `contains`, `toInt`, `trimmed`, etc.
- Perform temporary conversions inside the function body only when you truly need an owning result: `text.toString()`, `text.toUtf8()`, etc.

#### Forbidden (mandatory)
- Do not store views in members or caches:
  - `m_nameView = name;` is forbidden for any `QStringView`, `QAnyStringView`, or `QByteArrayView`
- Do not capture a view for delayed use:
  - `QTimer::singleShot(..., [name]{ ... })` when `name` is a view
- Do not pass a view through queued connections / across threads and use it later.
- Do not return views into temporary objects:
  - `return QStringView(QString("tmp"));`

#### Typical correct style (convert to owning before storing / async use)
```cpp
void Foo::setName(QAnyStringView name)
{
    d->name = name.toString();
}

void Foo::postLogMessage(QAnyStringView msg)
{
    const QString owned = msg.toString();
    QMetaObject::invokeMethod(this, [owned] { qInfo() << owned; }, Qt::QueuedConnection);
}
```

---

### 1.4 Qt/QML boundaries: do not treat views as “reflectable API types”

**Rules (mandatory)**
- The following interfaces must not use view types as parameters or return values:
  - `signals` / `slots`
  - `Q_INVOKABLE`
  - `Q_PROPERTY` READ / WRITE / NOTIFY related functions and signals

**Why (in practical engineering terms)**
- QML / `QMetaObject` call chains require parameters that can be copied / converted safely. A view does not own its data; copying the view does not copy the underlying characters or bytes.
- Once a queued call chain is involved (cross-thread / cross-event-loop), the data referenced by the view can easily become invalid before delivery, producing a dangling reference.
- Even if some view types can be registered as metatypes, that does not mean they are semantically safe for delayed invocation.

**Recommended pattern (two-layer API)**
- Expose owning signatures to QML (usually `QString`, `QUrl`, `QVariant`, etc.).
- Provide a separate high-performance C++ view / borrow entry point (non-invokable, non-slot), and let the owning layer forward to it.
  - For `QObject` types, prefer a different method name for the C++ view entry point, so that modern `connect(..., &Type::method)` syntax does not become ambiguous because of overloads.

```cpp
class Foo : public QObject {
    Q_OBJECT
    Q_PROPERTY(QString name READ name WRITE setName NOTIFY nameChanged)
public:
    QString name() const;

public slots:
    void setName(const QString &name);        // QML / Meta-Object boundary: owning

public:
    void setNameView(QAnyStringView name);    // pure C++ entry: Borrow (view, C++ only)

signals:
    void nameChanged(const QString &name);    // signals must be owning as well
};
```

If you insist on same-name overloads (for example, `void setName(const QString&)` + `void setName(QAnyStringView)`), modern `connect` syntax must disambiguate explicitly:

```cpp
connect(sender, &Sender::nameChanged,
        foo, qOverload<const QString &>(&Foo::setName));
```

---

### 1.5 Performance and simplicity: avoid “view entry + repeated owning conversions”

**Rules (recommended)**
- If a function is fundamentally doing parsing / comparison, keep the logic on the view as much as possible; avoid eager `toString()`.
- If the function ultimately needs an owning result:
  - convert to owning exactly once; avoid repeated `toString()` / `toUtf8()`
  - once converted, base the remaining logic on the owning object

**Typical anti-pattern**
```cpp
// Anti-pattern: repeated owning conversions
bool parse(QAnyStringView text)
{
    return text.toString().trimmed() == text.toString().toLower();
}
```

**Typical good pattern**
```cpp
bool parse(QAnyStringView text)
{
    const QString owned = text.toString();
    const QString trimmed = owned.trimmed();
    return trimmed == owned.toLower();
}
```

---

### 1.6 Relation to `std::string_view` / `std::u16string_view` (public API guidance)

**Rules (recommended)**
- In Qt/KDE-style public APIs, prefer Qt view types (`QAnyStringView`, `QStringView`, `QByteArrayView`).
  - Reason: Qt views encode more explicit character semantics in the type system (UTF-16, Latin1, UTF-8, or raw bytes), and integrate more naturally with Qt APIs.
- `std::*_view` is more appropriate for internal APIs or cross-library “pure STL interface” scenarios.
  - If the project must expose `std::string_view` publicly, document the encoding assumption (UTF-8 / Latin1 / unspecified) and lifetime contract explicitly, and provide an equivalent Qt entry point or conversion strategy.

---

### 1.7 Return value rules (Public API)

**Default rule (recommended)**
- Public API should return owning types by default (for example `QString` / `QByteArray`).
  - For `QString`, returning by value is often still acceptable because of implicit sharing.

**Only consider returning a view if all of the following are true**
- You can provide a clear lifetime contract that downstream code can follow safely (for example, the view points to stable storage inside the object and remains valid while the object lives and stays unmodified).
- You are willing to document the invalidation conditions explicitly and accept strict review scrutiny.

---

### 1.8 `QByteArrayView`: applied pattern for protocol / serialization scenarios (Public API)

**Rules (recommended)**
- Borrow-only: pass `foo(QByteArrayView bytes)` by value and finish parsing / comparison / slicing inside the function.
- If you need to store it / use it asynchronously / use it across threads: convert to owning early, and only once (for example `const QByteArray owned = bytes.toByteArray();`).
- Do not construct binary inputs from a bare pointer alone: `QByteArrayView(ptr)` scans until the first `Byte(0)` to infer the length. For binary data, always use `(ptr, len)` or construct from a container.

**Example**
```cpp
bool isValidPacket(QByteArrayView packet)
{
    if (!packet.startsWith("PKT/"))
        return false;
    return packet.contains('\n');
}

void Foo::enqueuePacket(QByteArrayView packet)
{
    const QByteArray owned = packet.toByteArray();
    m_queue.enqueue(owned);
}
```

---

## 2) Owning: for values that are stored / become members (`QString`-first, optional view convenience)

### 2.0 Owning parameters: signature choice and implementation constraints (supplement)

**Semantic recap**
- Owning = the input will be stored in members/caches/queues, or used across calls/event loops/threads/async or delayed execution.
- Owning semantics does not mean “the parameter must be by value” or “the parameter must be `const&`”. It constrains **lifetime and implementation**.

**Rules (mandatory)**
- For Owning scenarios: the stored member type must be owning (for example `QString` / `QByteArray`). Do not store views (`QAnyStringView` / `QStringView` / `QByteArrayView`) in members, caches, queues, or delayed-execution contexts.
- If the API crosses a Qt Meta-Object / QML boundary: do not use views as parameters/return values (see 1.4).

**Recommended Public API signatures (default starting point)**
- For Qt implicitly shared value types that will be stored as members/caches (`QString` / `QByteArray` / `QImage` / ...):
  - Default public API starting point: `const T&`
  - Why: aligns with established Qt/KF style; the signature is straightforward; expectations and compatibility are usually more stable downstream; and implicit sharing often makes “storing into a member” a shallow shared copy (see examples in this chapter).

**Optional: add a view convenience entry point (prefer a different method name)**
- If you truly need a single entry that accepts many string/byte sources, you may add a view entry point as a convenience:
  - The view entry point must convert to owning immediately (`toString()` / `toByteArray()`) and forward to the owning entry point or a single implementation point.
  - Prefer a different name (for example `setName()` + `setNameView()`), to avoid connect()/overload-resolution ambiguity (see 1.4 and 5.6).

**Optional: sink style in pure C++ Public API (by value + move)**
- Consider making the owning type a by-value canonical entry point (see 2.1) only if all of the following hold:
  - Pure C++ interface (not `slots` / `Q_INVOKABLE` / QML boundary)
  - You explicitly want a single entry point that naturally supports moves, and you accept the long-term signature-style tradeoff
- Note: do not describe sink style as the mainstream default for Qt6/KDE public setters. For published APIs, do not change signatures without evidence (compatibility: see Chapter 6).

**Minimal template (Owning)**
```cpp
void Foo::setName(const QString &name)
{
    d->name = name; // Owning: store as a member (often a shallow shared copy)
}

void Foo::setNameView(QAnyStringView name)
{
    setName(name.toString()); // convenience: convert to owning immediately, then forward
}
```

### Recommended style (Public C++ API)
- For Qt implicitly shared value types that will be stored as members / caches (`QString`, `QByteArray`, `QImage`, ...):
  - provide an owning entry point: `const QString&` (or the corresponding type)
  - optionally provide a `QAnyStringView` convenience entry point

Calibration: for most public setters, providing only the `const QString&` entry point is already reasonable. Whether you should add a separate view convenience overload depends on whether the API truly needs to accept multiple non-`QString` string sources, and whether you are willing to carry the extra public overload long-term.

Explanation: `QString` implicit sharing means “copying a `QString` into a member” is usually a shallow shared copy. A view entry point, on the other hand, often has to construct a new `QString` before storage. That is why keeping the `QString` entry point still has real value and also aligns better with established Qt/KDE style.

### Rules (mandatory)
- For Owning scenarios, the stored member type must itself be owning (for example `QString`), not a view.
- If both owning and view entry points are provided:
  - the view overload must convert to owning immediately and forward to a single implementation point
- When content needs normalization / modification, mutate the library-owned copy, not the caller’s object through a non-const reference.

> Tip: if the class is a `QObject` and is often used as a `connect(..., &Type::setName)` target, prefer naming the view entry point differently (for example `setNameView(QAnyStringView)`) to avoid function-pointer ambiguity. Otherwise, call sites will need `qOverload` / `static_cast` disambiguation (see 1.4 and 5.6).

### 2.1 Optional: owning sink directly in Public C++ API (pass by value + move)

> ⚠️ Compatibility note: do not change an **already-published** public function signature (including switching from `const T&` to sink style, or adding new overloads) merely for “modernization” / speculative performance. If a change is necessary, state the benefit, impact scope (source/ABI/QML), migration plan, and versioning strategy (see Chapter 6).

In the following scenarios, you may consider using `QString` by value as the canonical public entry point, to reduce overloads and support moves naturally (especially when callers pass temporaries):
- this is a pure C++ Public API (not a `slot`, `Q_INVOKABLE`, or QML boundary)
- you want to reduce overload complexity in public headers by using a single entry

> Calibration: this is an **optional** style for pure C++ Public API. It should not be generalized as the default style for Qt6 / KDE Frameworks 6 public setters. For public APIs that need to store strings, `const QString&` remains the safer and more common default.

```cpp
class Foo {
public:
    void setName(QString name);            // canonical sink (pure C++)
    void setNameView(QAnyStringView name); // optional convenience entry (view -> owning)
};

void Foo::setName(QString name)
{
    d->name = std::move(name);
}

void Foo::setNameView(QAnyStringView name)
{
    setName(name.toString());
}
```

### Example (recommended template)
```cpp
class Foo {
public:
    void setName(const QString &name);     // owning-friendly / shared-data-friendly
    void setNameView(QAnyStringView name); // convenience (view -> owning)
private:
    // d-pointer / private members...
};
```

```cpp
void Foo::setName(const QString &name)
{
    d->name = name; // uses implicit sharing (usually a shallow shared copy)
}

void Foo::setNameView(QAnyStringView name)
{
    setName(name.toString()); // if it must be stored, convert to owning immediately
}
```

---

## 3) Internal: prefer sink style in the implementation layer (pass by value + move)

Sink style should not be understood as “Internal only.” It can also be valid in the right pure C++ Public API. The more conservative interpretation is:
- keep `const QString&` as the mainstream public owning style, optionally with a separate view convenience entry point
- converge internal code on a single sink implementation

### Recommended template
```cpp
static void setNameImpl(FooPrivate *d, QString name)
{
    d->name = std::move(name);
}

void Foo::setName(const QString &name)    { setNameImpl(d, name); }
void Foo::setNameView(QAnyStringView name){ setNameImpl(d, name.toString()); }
```

Benefits:
- single logic path, less duplication
- if the caller passes a temporary `QString`, the implementation can move naturally
- the public API does not need a pile of `T&&` overloads just to support moves

---

## 4) Lifetime / async / threads (must collapse to Owning)

If **any** of the following is true, treat the input as Owning immediately:
- it will be stored as a member / cache
- it will be used across event loops (queued signals, `QMetaObject::invokeMethod(Qt::QueuedConnection)`, task queues, etc.)
- it will be used across threads
- it will be captured by a delayed closure

---

## 5) Overload and ambiguity control (public headers must stay restrained)

The more overloads there are in public headers:
- the easier it becomes for downstream code to hit unexpected overload resolution changes (especially after adding views)
- the higher the compatibility risk becomes (adding a “better match” overload can change what old call sites select)
- the harder it is to keep QML / Meta-Object behavior consistent
- the higher the long-term maintenance cost becomes

Therefore: overloads are an API design cost, not a performance optimization tool.

---

### 5.1 Design principle: treat overloads as a “capability set,” not a “type universe”

**Rules (mandatory)**
- Every overload must add a clear and independent capability (semantic difference, encoding difference, lifetime difference), not merely “accept more types.”

**Rules (recommended)**
- Define one canonical overload for each same-name function family, and have other entry points forward to it.
- Keep the overload set as small as possible for the same semantics (see 5.2 and 5.3).

---

### 5.2 Text parameters: recommended minimal overload sets

#### Borrow-only (read / parse)
- Recommended: provide exactly one view entry point (usually `QAnyStringView`, or a deliberately chosen `QStringView`, `QUtf8StringView`, or `QLatin1StringView`).
- Not recommended: multiple view overloads with the same semantics.

```cpp
// Recommended: single entry point
bool isValidId(QAnyStringView id);

// Not recommended: multiple view overloads for the same semantics
bool isValidId(QStringView id);
bool isValidId(QAnyStringView id);
```

#### Owning (store / async)
- Recommended: `const QString&` as the owning entry point; optionally add one `QAnyStringView` convenience entry point.
- Avoid adding further same-semantics overloads such as `const char*`, `QStringView`, `QLatin1StringView`, or `QUtf8StringView`.

---

### 5.3 Encoding and `const char*`: public API should not “guess the encoding”

**Rules (recommended)**
- Public API should avoid `const char*` text entry points whenever possible: the encoding semantics are unclear, and they compete easily with `QString` / view overloads.
- If you must support byte / character pointer input, use an explicitly encoded API so that the name and parameter type together make the encoding explicit:
  - `fromUtf8(QByteArrayView utf8)` / `setNameUtf8(QUtf8StringView utf8)`
  - `fromLatin1(QLatin1StringView latin1)`

```cpp
void setNameUtf8(QUtf8StringView name);
void setNameLatin1(QLatin1StringView name);
```

---

### 5.4 Default arguments and overloads: avoid distinguishing semantics only through defaults

**Rules (recommended)**
- Do not rely on “default arguments + overloads” together to express semantic differences. This significantly increases ambiguity at the call site and amplifies future evolution risks.

Prefer one of the following:
- Option A: one function with explicit default arguments (single semantic contract)
- Option B: different function names for different semantics (especially across QML boundaries)

---

### 5.5 Constructor overloads: prefer factory functions to reduce ambiguity

Constructors are inherently more likely to trigger implicit conversions and ambiguity.

**Rules (recommended)**
- Keep public constructors lightly overloaded. For complex inputs (especially different encodings / different sources), prefer named factory functions:
  - `Foo Foo::fromUtf8(QUtf8StringView)`
  - `Foo Foo::fromString(QString)`
- If a constructor takes a single parameter, it should usually be `explicit`.

---

### 5.6 QML / Meta-Object visible methods: avoid overloads (strong recommendation)

**Rules (recommended)**
- For methods exposed to QML (`Q_INVOKABLE` / `slots`):
  - avoid overloads whenever possible
  - use different names for different semantics / encodings: `setName()`, `setNameUtf8()`
- If you still want C++ overload convenience:
  - only mark the owning version as `Q_INVOKABLE` / slot; keep other overloads as pure C++ methods outside the meta-object system, so QML does not see multiple overloads
  - note that even if only one overload enters the meta-object system, same-name pure C++ overloads still make `&Type::method` ambiguous in modern `connect()` syntax; a different method name (for example `setNameView()`) is preferred, otherwise disambiguate explicitly:

```cpp
connect(sender, &Sender::nameChanged,
        foo, qOverload<const QString &>(&Foo::setName));
```

---

### 5.7 “Downstream call-surface” checks before adding overloads (ideally as compile tests)

> ⚠️ Compatibility note: adding overloads is not a “zero-risk optimization”. Even without changing call-site code, overload resolution changes can alter behavior or introduce ambiguity errors. For already-published public APIs, treat overload additions as compatibility decisions.

Before adding / adjusting overloads (especially when introducing views next to `const QString&`), at least verify that these typical call sites:
- compile without ambiguity
- select the overload you expect (or at least an acceptable one)

Suggested sample calls:
- `QString s; foo(s);`
- `foo(QStringLiteral("abc"));`
- `foo(u"abc"_qs);` (if the project uses `_qs`)
- `foo(QStringView{u"abc"});`
- `foo(QLatin1StringView{"abc"});`
- `foo(QUtf8StringView{u8"abc"});`

The most practical implementation is a compile-only test target under `tests/` that includes the headers and compiles representative calls (it does not even need to run, as long as CI compiles it).

---

## 6) Evolution and compatibility (non-mandatory; handled in review / refactor decisions)

- Recommended: if the project promises long-term stability (especially ABI), avoid changing already-published public function signatures (parameter types, cv qualifiers, default arguments, etc.). Prefer evolution through new overloads / new functions.
- Allowed: review / refactor / version-evolution work may still choose to change public APIs, including signatures. But treat that as an explicit compatibility decision, and record in review:
  - why the compatibility break is justified
  - the impact scope (downstream calls, binary compatibility, source compatibility, QML compatibility)
  - the migration plan (replacement APIs, examples, transition strategy, and if needed, major-version / SONAME policy)

---

## 7) Code Review Checklist

- Is the parameter semantically Borrow or Owning, and does the implementation match?
- Does the Borrow entry use a view passed by value, or was it accidentally written as `const& view`?
- Is there any risk of a view / borrowed reference being stored or used later (member, cache, queue, closure capture, queued connection)?
- Do Qt/QML boundaries use only owning types (`QString`, `QByteArray`, `QVariant`, etc.) and avoid views?
- In Owning scenarios, is there a `QString` (or matching implicitly shared type) entry point to benefit from implicit sharing? Is the view entry point only a convenience layer that converts to owning immediately?
- Can the public-header overload set introduce ambiguity or implicit-conversion traps? Have representative call sites been compile-checked?
- Will a `QObject` interface become ambiguous in modern `connect()` syntax because of overloads? Was that resolved through renaming or explicit `qOverload`?
- If the function is only pure parsing / comparison and the project style allows it, should it be marked `noexcept`? (enhancement, not a main rule here)
- If the public API returns a view, are its lifetime and invalidation conditions clearly documented in the signature / docs, and should `[[nodiscard]]` be considered? (enhancement, project-specific)

---

## Appendix A) Implicitly shared data (`QSharedDataPointer` / `QExplicitlySharedDataPointer`): `detach()` conventions (implementation-side)

> This appendix exists to reduce debates in code review about whether to hand-write `detach()`. It is implementation-side guidance and does not change the main rules in this document about Public API parameter semantics and type choices.

### A.1 `QSharedDataPointer`: you usually do not need to call `detach()` manually

**Rule (recommended)**
- When using `QSharedDataPointer<T>`, you usually should not write `d.detach()` by hand.
- Non-const access triggers detaching when needed, so code like `d->field = ...;` will detach when necessary.
- Note: for read-only access, prefer a const access path (make the function `const` when you can), to avoid “read-only but still detaches” overhead in non-const access paths.

**Example**
```cpp
// Usually no need: d.detach();
d->type = value; // non-const access triggers detaching when needed
```

### A.2 `QExplicitlySharedDataPointer`: you must call `detach()` explicitly

**Rule (mandatory)**
- `QExplicitlySharedDataPointer<T>` does not detach automatically.
- When you are about to modify shared data and the intended semantics are “copy-on-write”, you must call `detach()` first and then write.

**Example**
```cpp
d.detach();      // explicit detach
d->type = value; // modification affects only this instance
```

**References (Qt docs)**
- `QSharedDataPointer`: `https://doc.qt.io/qt-6/qshareddatapointer.html`
- `QExplicitlySharedDataPointer`: `https://doc.qt.io/qt-6/qexplicitlyshareddatapointer.html`
