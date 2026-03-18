# Qt6 C++17 Coding Style

English | 简体中文 | Source

> Note: This document is the English translation of the current Qt6 / C++17 coding guideline. If there is any discrepancy, the package baseline prevails.

## Guiding Principles

---
You are a senior Qt/KDE and modern C++17 developer. The following rules are mandatory and have the highest priority; if any rules conflict, the lower-numbered one wins.
All code must compile as modern C++17 (GCC ≥ 11, Clang ≥ 14, MSVC ≥ 2019), pass clang-format (using the bundled formatting baseline) and optionally clang-tidy (see Chapter 10 for a minimal example), and keep the project build/tests warning-free (where applicable). For more detailed conventions, refer to:
- https://wiki.qt.io/Qt_Coding_Style
- https://wiki.qt.io/Coding_Conventions
- https://community.kde.org/Policies/Frameworks_Coding_Style

---

## 0 Overview
- Compilers: GCC ≥ 11 | Clang ≥ 14 | MSVC ≥ 2019
- Standard: C++17 (`set(CMAKE_CXX_STANDARD 17)`)
- Warnings: enable `-Wall -Wextra -Wpedantic`, **no-warning commits**
- Formatting: keep a shared clang-format baseline file in the project (copy/symlink it to `.clang-format` if you want tooling auto-discovery); run `git clang-format --style=file` against the project baseline before committing
- Forbidden: exceptions, RTTI, `dynamic_cast`, raw `new` (explicit exception for `QObject`-derived types; see Chapter 6), single-statement control blocks without braces, 64-bit enums
- Use templates wisely, not just because you can
- Avoid C casts, prefer C++ casts (`static_cast`, `const_cast`, `reinterpret_cast`)
- Don't use `dynamic_cast`; use `qobject_cast` for QObjects, or refactor your design (e.g., introduce a `type()` method; see `QListWidgetItem`)
- Use constructors for simple casts: `int(myFloat)` instead of `(int)myFloat`

---

## 1 Files and Encoding
| Rule | Good | Bad |
|---|---|---|
| UTF-8 without BOM | Save as UTF-8 | UTF-8-BOM |
| `#include` order | clang-format groups: ① `"..."` ② `<Q...>/<Qt...>` ③ other `<...>` | reversed order |
| `#include` style | `#include <QString>` | `#include <QtCore/QString>` |
| include guards | `#ifndef MYWIDGET_H ...` | `#pragma once` (*tooling only*) |

### 1.1 Include Guards
- If you would include it with a leading directory, use that as part of the guard name
- Put guards below any license text

Example for `kaboutdata.h`:
```cpp
#ifndef KABOUTDATA_H
#define KABOUTDATA_H
```
Example for `kio/job.h`:
```cpp
#ifndef KIO_JOB_H
#define KIO_JOB_H
```
---

## 2 Naming
| Kind | Style | Good | Bad |
|---|---|---|---|
| Classes | PascalCase | `class MainWindow` | `class main_window` |
| Functions / variables | camelCase | `void updateData()` | `void updatedata()` |
| Member variables | `m_` prefix | `int m_count` | `int count_` |
| Static / global | `s_` prefix | `static QObject *s_instance` | `static QObject *instance` |
| Constants | `k` prefix | `constexpr int kMaxDepth = 3` | `const int MAX_DEPTH = 3` |
| Enum values | CamelCase + trailing comma | `enum class Direction { North, South, };` | `enum Direction { NORTH };` |
| Namespaces | all-lowercase | `namespace app::utils` | `namespace AppUtils` |

- Avoid short or meaningless names (e.g., "a", "rbarr", "nughdeget")
- Single-character variable names are only OK for counters and temporaries where the purpose is obvious
- Do not declare a variable until it is needed
- Variables and functions start with a lower-case letter; each consecutive word starts with an upper-case letter

---

## 3 Indentation and Braces (KDE Style)
| Rule | Good | Bad |
|---|---|---|
| Indentation | 4 spaces | Tabs |
| Single-statement `if/for/while` | braces are required | `if (x) {\n    return;\n}` |
| Left brace | attached for control statements; newline for functions/classes/structs | inconsistent |
| `else` placement | `} else {` | `}\nelse` |
| `case` indentation | indent case labels by 1 level | `    case 0:\n        break;` |

- For pointers/references, always use one space between the type and `*`/`&`, but no space between `*`/`&` and the variable name:
```cpp
char *x;
const QString &myString;
const char * const y = "hello";
```
- Surround binary operators with spaces
- No space after a cast (and avoid C-style casts)
```cpp
// Wrong
char* blockOfMemory = (char* ) malloc(data.size());

// Correct
char *blockOfMemory = reinterpret_cast<char *>(malloc(data.size()));
```
---

## 4 Line Length and Wrapping
- Soft limit: 100 columns. Put binary operators at the beginning of the wrapped line (clang-format). Use "leading comma" style for constructor initializer lists (KDE/clang-format).
```cpp
// Good
if (longCondition1
    && longCondition2) {
}

// Bad
if (longCondition1 &&
    longCondition2) {
}
```

Constructor initializer list example (leading comma):
```cpp
Foo::Foo(int a, int b)
    : m_a(a)
    , m_b(b)
{
}
```

---

## 5 Optional Modern C++17 Best Practices (Used in Qt6/KF6)

> **Note**: the items below are **optional recommendations**, not mandatory rules.
> - ✅ **Encouraged**: prefer these modern patterns in new code
> - 🔄 **Gradual migration**: existing code may remain unchanged; no forced refactors
> - 🤔 **Trade-offs**: decide based on team familiarity, performance needs, and readability

| Scenario | Recommended | Traditional (still acceptable) |
|---|---|---|
| Optional return value | `std::optional<QColor> tryColor()` | `bool getColor(QColor *out)` |
| `variant` access | `std::visit([](auto& v){ ... }, var)` | hand-written `switch(type)` |
| Structured bindings | `auto [it, inserted] = map.insert({k, v});` | `QPair<It,bool> res = ...` |
| Compile-time constants | `constexpr int kSize = 256;` | `const int kSize = 256;` or `#define` |
| `nodiscard` | `[[nodiscard]] int calc() const;` | no attribute (compiler does not enforce) |
| `maybe_unused` | `[[maybe_unused]] auto idx = ...;` | `Q_UNUSED(idx);` |
| Atomics | `std::atomic<int> value; value.fetch_add(1)` | `QAtomicInt` or a mutex |
| Binary buffer views | `QByteArrayView buf` | `(const char*, size_t)` |
| Path composition | `std::filesystem::path p = dir / "file.txt";` | `QDir::cleanPath(dir + "/file.txt")` |
| Timing | `auto t0 = std::chrono::steady_clock::now();` | `QElapsedTimer` |
| Fold expressions | `(stream << ... << args);` | hand-written loop concatenation |

**Usage guidelines**:
- New features / new files: prefer modern patterns
- Maintaining old code: keep style consistent; avoid mixing styles
- Team work: choose based on team consensus; standardize
- Performance-sensitive code: measure; "zero-cost abstractions" like `std::optional` typically have no runtime cost
- `[[nodiscard]]`: mandatory for "may fail" APIs (see §5.1); optional elsewhere depending on team preference

### Optional for C++20+ (Only when C++20 is enabled)

| Scenario | Recommended | Notes |
|---|---|---|
| Binary buffer views | `std::span<const std::byte> buf` | requires C++20 |
| Atomic view | `std::atomic_ref<int>(val).fetch_add(1)` | requires C++20 |

## 5.1 Error Handling and Assertions (No-Exception Codebase)

> This standard forbids exceptions (`throw`/`try`/`catch`), so the project must adopt a unified, auditable "failure reporting" strategy to avoid inconsistency and swallowed errors.

### 5.1.1 General Rules (Mandatory)

- **Forbidden**: `throw`/`try`/`catch` in project code; do not use exceptions as control flow.
- **Required**: any function that "may fail" must **explicitly report failure**; do not swallow errors via implicit default values.
- **Required**: failure-returning APIs must be `[[nodiscard]]` to prevent callers from ignoring failures (e.g., `[[nodiscard]] bool ...`, `[[nodiscard]] std::optional<T> ...`).
- **Required**: use assertions only for *programmer errors* (broken preconditions/invariants that should not happen). For recoverable errors (user input, I/O, network, plugin loading, etc.), use return-value-based failure reporting.
- **Should**: be clear about who owns error reporting. Low-level library functions should not spam `qWarning()` unconditionally; log or surface user-visible errors at the boundary layer (UI/CLI/service entry points).

### 5.1.2 Choosing a Failure-Reporting Style (Recommended)

> Goal: keep styles consistent within a module so code review can quickly audit how failures are handled and propagated.

**A. No error reason needed: `std::optional<T>`**
- **Recommended**: use `std::optional<T>` for "result present/absent"; return `std::nullopt` on failure.
- **Required**: document the meaning of `nullopt` in the API comment (not found / not applicable / parse failed, etc.).

**B. Error reason needed (text): `bool + QString *error`**
- **Recommended**: return `bool` and use an extra `QString *error` (nullable) for the failure reason.
- **Required**: on failure, if `error != nullptr`, write a diagnosable reason; on success, either leave untouched or clear (standardize within the team).

**C. Error reason needed (structured): `bool + ErrorCode`**
- **Recommended**: for enumerable failure reasons, define `enum class ErrorCode` and return it via `ErrorCode *out` (or expose as `lastError()`), avoiding excessive string construction.

### 5.1.3 Third-Party Exception Boundary (Mandatory)

- **Required**: if a third-party dependency may throw, catch at the module boundary and translate into one of the failure-reporting styles above. Exceptions must not cross Qt event loop / signal-slot boundaries.

### 5.1.4 Examples (Qt6 + C++17)

**❶ `optional`: return empty on parse failure**
```cpp
#include <optional>
#include <QStringView>

[[nodiscard]] std::optional<int> parsePort(QStringView text)
{
    bool ok = false;
    const int port = text.toInt(&ok);
    if (!ok || port <= 0 || port > 65535) {
        return std::nullopt;
    }
    return port;
}
```

**❷ `bool + error`: carry diagnosable info**
```cpp
#include <QFileInfo>
#include <QString>

[[nodiscard]] bool loadConfig(const QString &path, QString *error)
{
    if (!QFileInfo::exists(path)) {
        if (error) {
            *error = QStringLiteral("Config file not found: %1").arg(path);
        }
        return false;
    }
    return true;
}
```

---

## 6 Qt 6-Specific Conventions
| Rule | Recommendation | Good | Bad |
|---|---|---|---|
| `Q_OBJECT` | Required for every `QObject`-derived type | `Q_OBJECT` present | Missing it breaks `qobject_cast` |
| Signal/slot connections | Use new-style `connect` | `connect(sender, &Sender::valueChanged, receiver, &Receiver::update);` | `SIGNAL/SLOT` strings |
| String literals | Use `QStringLiteral("...")` or `u"..."_qs` | `QStringLiteral("hello")` | `QString("hello")` |
| Heavy work | Prefer `QtConcurrent::run(&Worker::doWork)` | `QtConcurrent::run(&Worker::doWork)` | manual `new Thread` |
| `QObject` semantics | Pointer semantics only (pass/store pointers/references) | `QVector<Foo *> foos;` | `QVector<Foo> foos;` |
| Memory management | Parent-child (preferred) / `deleteLater()` / RAII | explicit and traceable ownership | raw `new` + manual `delete` |

- For smart pointers, prefer Qt-provided ones (`QScopedPointer`, `QSharedPointer`, `QWeakPointer`, `QPointer`)
- **`QObject` lifetime (exception rules)**:
  - Default: raw `new` is still **forbidden**.
  - Exception: raw `new` is allowed for `QObject`-derived objects, but destruction must be one of:
    - ✅ **Parent-child tree (preferred)**: `new T(parent)`, destroyed by the parent's destructor
    - ✅ **Event-loop-safe destruction**: no parent and involves event loop / async / queued connections / cross-thread → use `obj->deleteLater()`
  - ❌ Forbidden: manual `delete` for `QObject`-derived objects
  - Non-`QObject`: manage resources with RAII and smart pointers (e.g., `std::unique_ptr` / `QScopedPointer`)
  - Caution: using `std::unique_ptr` / `std::shared_ptr` / `QScopedPointer` for `QObject` can conflict with `deleteLater()` and thread/event-loop timing; prefer `QPointer` when you need dangling protection

### 6.1 Value Semantics/RAII and QObject (Team-Ready Version)

#### Conclusion (Read This First)
- `QObject` and its derived types **should not, and cannot** be forced into value semantics (copyable/movable). They are typical **identity types**: semantics are bound to object identity (address), object tree, meta-object system, signal-slot connections, and thread affinity.
- RAII still **applies** to `QObject`, but the practical pattern is: **wrap non-Qt resources with RAII as members**, while `QObject` itself follows **parent ownership / `deleteLater()`**. Treating a whole `QObject` as a copyable/movable "value object" is a misconception.
- Team guidance: split "value semantics/RAII" into two clearly-scoped rule sets: (1) general C++ types (non-`QObject`) → prefer value semantics + RAII; (2) `QObject`-derived types → pointer semantics + Qt lifetime model.

#### Why QObject Does Not Fit Value Semantics (By Dimension)
- **Copy/move semantics**: `QObject` disables copy. "Moving a QObject" changes its address/identity and breaks internal state (connections, events, properties, etc.). Qt does not provide usable move semantics for it.
- **Identity**: a `QObject` often represents a concrete object (`objectName`, dynamic properties, `eventFilter`, `findChild`, etc.); equivalence is not "value equality".
- **Ownership tree**: the parent owns and destroys children. Copy/move cannot meaningfully define "how to copy/move the subtree", and easily introduces double-free/dangling references. Also, parent/child must live in the same thread.
- **Meta-object system**: `Q_OBJECT`, reflection, properties, and dynamic properties are tied to the concrete instance; copy/move does not "duplicate runtime meta state/registration".
- **Signal-slot connections**: connections are built around sender/receiver identity (addresses). Copy/move cannot correctly "copy connection relationships", and move breaks existing connection semantics.
- **Thread affinity**: `QObject` has thread affinity; events and most member calls should happen on the owning thread. Cross-thread destruction must go through the event loop and queued connections.
- **Event loop / deferred delete**: `deleteLater()` destroys safely via the event loop to avoid UAF from pending events/queued slots, which differs from the traditional RAII timing of "destroy immediately at scope end".

#### Rules (Executable Spec Text)

**A. Value semantics / RAII for general C++ types (non-`QObject`)**
- **Required**: manage resources via RAII; avoid raw `new`/`delete` (see Chapters 0/6).
- **Should**: prefer value semantics (copyable or movable) and by-value containers. Exclusive-ownership types should be movable-but-not-copyable. Consider `std::shared_ptr` / `QSharedPointer` only for truly shared ownership.
- **Forbidden**: using raw pointers to express ownership; transferring ownership implicitly via "return a raw pointer + verbal agreement".

**B. `QObject`-derived types (special case: pointer semantics + Qt lifetime model)**
- **Required**: `QObject`-derived types **must not be copyable/movable**, and the class should declare it explicitly for clear compile-time diagnostics. Recommend `Q_DISABLE_COPY(Class)` and explicitly delete moves: `Class(Class &&) = delete; Class &operator=(Class &&) = delete;`.
- **Forbidden**: pass/return `QObject`-derived types by value. Do not store `QObject`-derived types by value in containers that require move/copy such as `std::vector` / `QVector` / `QList`.
- **Required**: `QObject`s that need `moveToThread()` **must not have a parent**. Do not create cross-thread parent/child relationships (parent/child must be in the same thread).
- **Required**: ownership must be one of the following and be traceable:
  - **parent ownership**: `new T(parent)`, destroyed by the parent
  - **explicit ownership**: define a single owner (typically a `QObject`/manager); release via `deleteLater()` or safe deletion on the owning thread (see cross-thread example below)
- **Forbidden**: manual `delete` for `QObject`-derived objects (declared in Chapter 6).
- **Should**: prefer `deleteLater()` when the object participates in the event loop / async callbacks / queued connections / timers / networking; when destruction is requested from a non-owning thread; or when self-destruction is needed during slot/event handling. Ensure the object's thread has an event loop that can process deferred deletes.
- **Should**: if an object may be destroyed asynchronously (parent destruction, `deleteLater`, cross-thread), use weak references like `QPointer<T>` to avoid dangling pointers; do not cache raw pointers across async boundaries.
- **Should**: when a child `QObject` has exactly the same lifetime as its owner, prefer "direct member + parent" (e.g., `QTimer m_timer{this};`) over heap allocation. This is *not* value semantics; copy/move/by-value containers are still forbidden.
- **Should**: creation APIs should use a `QObject *parent` parameter to express ownership. Return values are typically raw pointers to express "non-owning" (owned by parent/owner); document the lifetime in the API comment.
- **Should**: default convention: raw pointer/reference parameters and return values are **borrowing (non-owning)**. If ownership is transferred, it must be explicit via setting a parent, naming (e.g., `takeOwnership...`), or documentation.
- **Optional (with rationale)**: owning a `QObject` via `std::unique_ptr` / `QScopedPointer` is allowed only when **not using `deleteLater()`**, **not crossing threads**, and **the destructor runs on the object's owning thread**. If deferred deletion is required, use a custom deleter that calls `deleteLater()` and ensure an event loop exists.
- **Optional (strict constraints)**: shared ownership is usually not suitable for `QObject`. If truly needed, **do not set a parent**, and use `QSharedPointer` / `std::shared_ptr` with a custom deleter (`deleteLater`). Prefer exposing `QPointer` / `QWeakPointer` externally.

#### 6.1.1 Notes on QIODevice (Qt I/O Device Objects)

> Scope: any type derived from `QIODevice` (including your own derived types). `QIODevice` itself is `QObject`-derived, so all `QObject` rules above still apply. This subsection adds why you must be extra careful, and practical engineering rules.

- **Semantics**: `QIODevice` is not a "data container"; it is an **I/O endpoint with state and callbacks** (open/close, buffers, async notification signals, etc.), heavily dependent on identity and the event loop. Therefore, **value semantics are forbidden (copy/move/by-value containers)**.
- **Lifetime strategy (more biased toward `deleteLater()` than general QObject)**:
  - **Should**: if the device participates in async notifications (e.g., `readyRead` / `bytesWritten` / error signals, socket notifiers, queued connections, timer-driven I/O), prefer `deleteLater()` to avoid UAF from pending events/queued slots.
  - **Should**: call `close()` before scheduling destruction (when applicable) to stop I/O/callbacks sooner, and avoid leaking device pointers into long-lived objects when destruction order is unclear.
- **Stack object boundary**:
  - **Optional**: `QFile` and similar can be stack-allocated in functions that are purely synchronous, do not leak pointers, and do not establish async connections. Once you connect signals or hand out a `QIODevice*` to async logic / store it across scopes, switch to parent/explicit owner + `deleteLater()`.
- **Owning smart pointers (strict version)**:
  - **Forbidden**: except for "pure synchronous scenarios", do not manage `QIODevice` and derived types with *default deleters* in owning smart pointers (`std::unique_ptr` / `QScopedPointer` / `std::shared_ptr` / `QSharedPointer`).
  - **Pure synchronous scenario (all must hold)**:
    1. Single-threaded synchronous use; no `moveToThread()`
    2. No async signals/callbacks such as `readyRead` / `bytesWritten`; no connections to long-lived objects
    3. Do not leak `QIODevice*`/references (no storing, no async boundaries, do not pass into async APIs)
    4. Do not call `deleteLater()`; expect immediate destruction at scope end
  - **Optional (controlled exception)**: if you must express ownership via a smart-pointer style while the object participates in the event loop, allow a smart pointer with a custom deleter that calls `deleteLater()` (e.g., `QSharedPointer<T>(new T, &QObject::deleteLater)`), and **do not set a parent**. You must also ensure the object's thread event loop exists.
- **Threads and affinity**:
  - **Required**: use the device only on its owning thread (thread affinity). Cross-thread interactions must use signals/queued connections.
  - **Required**: devices that need `moveToThread()` **must not have a parent**. Destruction must be queued to the owning thread to call `deleteLater()`.
- **API ownership expression (standardize within the team)**:
  - **Should**: treat "using a device" as dependency injection: accept `QIODevice *device` / `QIODevice &device` as borrowed dependencies; the caller ensures lifetime outlives usage.
  - **Required**: if a function/object needs to "take over device lifetime", express ownership via `QObject *parent` (or an explicit owner). Smart-pointer ownership rules are described above.
- **Common `QIODevice` family types (non-exhaustive)**:
  - QtCore: `QBuffer`, `QFile`, `QSaveFile`, `QTemporaryFile`, `QProcess`
  - QtNetwork: `QAbstractSocket` (`QTcpSocket`, `QUdpSocket`), `QSslSocket`, `QLocalSocket`, `QNetworkReply`
  - QtSerialPort: `QSerialPort`
  - Note: `QTextStream` / `QDataStream` are *not* `QIODevice` subclasses; they are stream wrappers around a `QIODevice`.

#### Typical Examples (Qt6 + C++17)

**❶ Wrong: treating `QObject` as a value type / mixing `deleteLater()` with owning smart pointers**
```cpp
#include <QObject>
#include <QVector>
#include <QWidget>
#include <memory>

class Bad : public QObject
{
    Q_OBJECT
public:
    explicit Bad(QObject *parent = nullptr)
        : QObject(parent)
    {
    }
};

class BadOwner : public QObject
{
    Q_OBJECT
public:
    explicit BadOwner(QObject *parent = nullptr)
        : QObject(parent)
        , m_widget(std::make_unique<QWidget>())
    {
    }

    void scheduleDelete()
    {
        m_widget->deleteLater(); // ❌ deleteLater may run before unique_ptr destruction → dangling / double delete
    }

private:
    std::unique_ptr<QWidget> m_widget; // ❌ owning smart pointer conflicts with Qt async destruction
};

void badExamples()
{
    QVector<Bad> list; // ❌ QObject-derived is not copyable/movable; by-value containers do not work
}
```

**❷ Recommended: parent ownership + RAII members (resources as members; QObject follows Qt lifetime)**
```cpp
#include <QObject>
#include <QPointer>
#include <QTimer>
#include <memory>

class Controller : public QObject
{
    Q_OBJECT
public:
    explicit Controller(QObject *parent = nullptr)
        : QObject(parent)
        , m_timer(this) // ✅ child QObject: same lifetime; direct member + parent
        , m_backend(std::make_unique<Backend>()) // ✅ non-QObject resource: RAII
    {
        m_timer.setInterval(1000);
        connect(&m_timer, &QTimer::timeout, this, &Controller::poll);
        m_timer.start();
    }

    Q_DISABLE_COPY(Controller)
    Controller(Controller &&) = delete;
    Controller &operator=(Controller &&) = delete;

private slots:
    void poll();

private:
    struct Backend {
        void poll();
    };

    QTimer m_timer;
    std::unique_ptr<Backend> m_backend;
    QPointer<QObject> m_maybeGone; // ✅ weak ref to avoid dangling (owned by external owner/parent)
};
```

**❸ Cross-thread: thread affinity and destruction strategy (no cross-thread delete; use queued + deleteLater)**
```cpp
#include <QObject>
#include <QThread>

class Worker : public QObject
{
    Q_OBJECT
public slots:
    void doWork();
signals:
    void finished();
};

void startWorker(QObject *owner)
{
    auto *thread = new QThread(owner); // ✅ thread owned by owner
    auto *worker = new Worker();       // ✅ no parent; deleteLater on its own thread later
    worker->moveToThread(thread);

    QObject::connect(thread, &QThread::started, worker, &Worker::doWork);
    QObject::connect(worker, &Worker::finished, thread, &QThread::quit);
    QObject::connect(thread, &QThread::finished, worker, &QObject::deleteLater);
    QObject::connect(thread, &QThread::finished, thread, &QObject::deleteLater);

    thread->start();
}
```

### 6.2 Signal/Slot Connections and Thread Semantics (Mandatory + Recommended)

> Goal: avoid common crash sources such as lambda dangling captures, cross-thread UI access, and mismatched parameter types for queued connections; and make it easy for code review to audit "which thread the slot runs on" and "how the connection lifetime ends".

#### 6.2.1 `connect` Syntax and Lifetime (Mandatory)

- **Required**: use new-style `connect` only (function pointer/member function pointer/functor with context). Do not use `SIGNAL()`/`SLOT()` strings.
- **Required**: when connecting with a lambda/functor, you must provide a **context** (typically the receiver/owner). Do not use the overload that connects a functor **without** a context.
- **Required**: lambdas/functors must not capture raw pointers that may be destroyed before the connection is disconnected. If you capture `this`, the context must be `this` (or a longer-lived owner), and prefer capturing `QPointer<T>` for weak-reference protection.
- **Recommended**: use `Qt::UniqueConnection` at repeated connection points (may be called multiple times) to prevent duplicate connections. If manual disconnection is needed, store `QMetaObject::Connection` and call `disconnect()` at the right time.

Examples (recommended / forbidden):
```cpp
// ✅ Recommended: with context; auto-disconnect when this is destroyed
connect(sender, &Sender::valueChanged, this, [this](int v) { onValue(v); });

// ❌ Forbidden: no context; capturing this easily causes UAF
connect(sender, &Sender::valueChanged, [this](int v) { onValue(v); });
```

#### 6.2.2 Thread Semantics (Mandatory)

- **Required**: do not access GUI objects from non-GUI threads. GUI updates must be performed on the GUI thread (queued connection or `QMetaObject::invokeMethod`).
- **Required**: if sender/receiver is clearly cross-thread (or receiver may `moveToThread()`), explicitly use `Qt::QueuedConnection`.
- **Forbidden**: use `Qt::DirectConnection` across threads (unless the slot is fully thread-safe and does not touch GUI / thread-affine Qt objects, and you document the rationale in code review).
- **Forbidden (by default)**: `Qt::BlockingQueuedConnection`, unless you have a clear synchronization need and can prove it will not deadlock (must be justified in review).

#### 6.2.3 Parameter Types for Queued Connections (Mandatory)

- **Required**: any custom type passed via queued connections must be recognized by the Qt meta-type system: at least `Q_DECLARE_METATYPE(T)`, and ensure it is registered before use (recommended: `qRegisterMetaType<T>()` once in `main()` or module init).

Example (custom type used as a queued parameter):
```cpp
struct Payload { int id = 0; };
Q_DECLARE_METATYPE(Payload)

// Once at program start or module init:
// qRegisterMetaType<Payload>("Payload");
```

### 6.3 Meta-Object System and Properties (Q_OBJECT / Q_PROPERTY / Metatypes) (Mandatory + Recommended)

#### 6.3.1 Macros and Type Choice (Mandatory)

- **Required**: every `QObject`-derived class must contain `Q_OBJECT`.
- **Recommended**: for value types that need reflection (enums/properties) but do not need QObject lifetime/signals, use `Q_GADGET`. For enums exposed from a namespace, use `Q_NAMESPACE` (with `Q_ENUM_NS` / `Q_FLAG_NS`).

#### 6.3.2 Q_PROPERTY (Mandatory)

- **Required**: mutable properties must have `NOTIFY`. Setters must return early when the value does not change to avoid pointless signals and binding churn.
- **Required**: be explicit about thread affinity. If a property may be updated cross-thread, switch back to the object's thread via queued invocation before modifying and emitting.

Example (recommended style):
```cpp
class Person : public QObject
{
    Q_OBJECT
    Q_PROPERTY(QString name READ name WRITE setName NOTIFY nameChanged)
public:
    QString name() const { return m_name; }
    void setName(const QString &name)
    {
        if (m_name == name) {
            return;
        }
        m_name = name;
        emit nameChanged();
    }
signals:
    void nameChanged();
private:
    QString m_name;
};
```

#### 6.3.3 Q_ENUM / Q_FLAG (Recommended)

- **Recommended**: prefer `enum class` and expose enums via `Q_ENUM` / `Q_ENUM_NS` for QML/debugging/reflection.
- **Recommended**: use `QFlags` + `Q_FLAG` / `Q_FLAG_NS` for bit flags, and standardize operators via `Q_DECLARE_FLAGS` / `Q_DECLARE_OPERATORS_FOR_FLAGS`.

#### 6.3.4 Metatype Registration (Mandatory)

- **Required**: any custom type used as a queued-connection parameter, stored in `QVariant`, or used across the QML boundary (property/signal/method parameter) must be registered as a metatype (see 6.2.3). Across modules, ensure registration happens on a reachable path before use to avoid "registered only by static init in one .cpp" flakiness.

---

## 7 Memory and Singletons
```cpp
// Good: function-local static
Thing& thing() {
    static Thing inst;
    return inst;
}

// Good: Q_GLOBAL_STATIC
Q_GLOBAL_STATIC(Thing, s_thing)

// Bad: global raw pointer
static Thing* g_thing = new Thing;
```

---

## 8 Lambdas and auto
```cpp
// Good: multi-line formatting
auto l = []() -> bool {
    doSomething();
    return true;
};

// Bad: single line mixed with multi-line
auto l = []() { doSomething();
    return true; };
```

---

## 9 Project Template Layout
```
MyApp/
├── CMakeLists.txt
├── Qt6_CPP17_CLANG-FORMAT
├── .clang-tidy
├── src/
│   ├── main.cpp
│   ├── MainWindow.h
│   └── MainWindow.cpp
├── qml/              (optional)
├── resources/
│   └── resources.qrc
├── translations/
└── tests/
```

---

## 10 Config Files (Copy to Project Root)

### clang-format Baseline (Qt/C++)

This package includes the complete formatting baseline. This document keeps only the key summary to avoid YAML drift. If you want clang-format to auto-discover it, copy/symlink the chosen baseline to `.clang-format`.

Key conventions (summary):
- 4-space indentation (`IndentWidth: 4`, `TabWidth: 4`, `UseTab: Never`)
- Pointer style: `Type *var` (`PointerAlignment: Right`)
- Line width: `ColumnLimit: 100`
- Include grouping: regroup blocks (`IncludeBlocks: Regroup`); treat `<Q...>` and `<Qt.../...>` as Qt; sort within a block case-sensitively (`SortIncludes: CaseSensitive`)
- Braces: attached for control statements; newline for functions/classes/structs (`BraceWrapping`); no single-line blocks (`AllowShortBlocksOnASingleLine: Never`)
- Switch/case: indent case labels (`IndentCaseLabels: true`, `IndentCaseBlocks: false`)
- Wrapping: binary operators at line start (`BreakBeforeBinaryOperators: All`); leading commas for constructor initializers (`BreakConstructorInitializers: BeforeComma`)
- Comments: do not reflow (`ReflowComments: false`); align trailing comments (`AlignTrailingComments`)

Key configuration reference (wrapping/indent/init list/braces/spaces/comments/sorting) (see the baseline notes in this directory for the authoritative configuration summary):

| Key (Formatting Baseline) | Meaning | Example |
|---|---|---|
| `BraceWrapping.AfterControlStatement: Never` | `{` does not start on a new line for `if/for/while` | `if (ok) {` |
| `BraceWrapping.AfterFunction: true` | `{` starts on a new line for function definitions | `void f()`<br>`{` |
| `BraceWrapping.AfterClass: true` / `AfterStruct: true` | `{` starts on a new line for `class/struct` | `class Foo`<br>`{` |
| `BraceWrapping.BeforeElse: false` / `BeforeCatch: false` | `else/catch` stays on the same line as `}` | `} else {` |
| `SpaceBeforeParens: ControlStatements` | space before parens in control statements; no space after function names | `if (ok)` / `foo()` |
| `SpacesInAngles: Never` | no spaces inside template angle brackets | `std::vector<int>` |
| `SpaceBeforeRangeBasedForLoopColon: true` | spaces around `:` in range-for | `for (auto v : xs)` |
| `SpacesInLineCommentPrefix.Minimum: 1` | at least one space after `//` | `// comment` |
| `ReflowComments: false` | do not reflow/wrap comments | `// long long long ...` (not auto-wrapped) |
| `AlignTrailingComments.Kind: Always` + `SpacesBeforeTrailingComments: 1` | align trailing comments; at least one space before them | `int a = 1;   // a`<br>`int bb = 2;  // b` |
| `IncludeBlocks: Regroup` | group `#include`s by category and insert blank lines | `"Foo.h"`<br>`<Q...>`<br>`<...>` |
| `IncludeCategories` | include category priorities: ① `"..."` ② `<Qt|Q...>` ③ other `<...>` | `#include "Foo.h"`<br>`#include <QString>`<br>`#include <vector>` |
| `SortIncludes: CaseSensitive` | sort within a group (case-sensitive) | `#include <QByteArray>` before `#include <QWidget>` |
| `SortUsingDeclarations: LexicographicNumeric` | sort `using` declarations by lexicographic+numeric order | `using Foo2 = ...;` before `using Foo10 = ...;` |
| `IndentWidth: 4` + `TabWidth: 4` + `UseTab: Never` | indent with 4 spaces; do not use tabs | `if (ok) {`<br>`    foo();`<br>`}` |
| `ContinuationIndentWidth: 4` | continuation indent is 4 spaces | `foo(arg1,`<br>`    arg2);` |
| `ColumnLimit: 100` | soft limit of 100 columns | `auto s = QStringLiteral("...");` (wrap when too long) |
| `BreakBeforeBinaryOperators: All` | binary operators at the beginning of wrapped lines | `if (a`<br>`    && b) {` |
| `BreakBeforeTernaryOperators: true` | `?`/`:` at the beginning of wrapped lines | `auto x = cond`<br>`    ? a`<br>`    : b;` |
| `AllowShortBlocksOnASingleLine: Never` | disallow `{ ... }` on one line | `if (ok) {`<br>`    foo();`<br>`}` |
| `AllowShortIfStatementsOnASingleLine: Never` | disallow `if (x) y;` one-line if | `if (ok) {`<br>`    foo();`<br>`}` |
| `AllowShortLoopsOnASingleLine: false` | disallow `for (...) x;` one-line loops | `for (...) {`<br>`    work();`<br>`}` |
| `BreakConstructorInitializers: BeforeComma` + `PackConstructorInitializers: BinPack` | use leading commas in ctor initializer lists | `Foo::Foo()`<br>`    : m_a(a)`<br>`    , m_b(b)` |
| `ConstructorInitializerIndentWidth: 4` | indent initializer list by 4 spaces | `Foo::Foo()`<br>`    : m_a(a)` |
| `IndentCaseLabels: true` + `IndentCaseBlocks: false` | indent case labels; indent case bodies by nesting | `switch (x) {`<br>`    case 0:`<br>`        break;`<br>`}` |
| `AlwaysBreakTemplateDeclarations: Yes` | put `template <...>` on its own line | `template <typename T>`<br>`class Foo` |
| `BreakInheritanceList: BeforeColon` | when wrapping inheritance lists, put `:` on the next line (KDE style) | `class Foo`<br>`    : public Bar` |

### .clang-tidy (Minimal "Zero Warning" Set)

Below is a minimal example that you can copy into a product repo root (adjust as needed):
```
Checks: >
  -*,performance-*,readability-*,-readability-magic-numbers,modernize-*,
  -modernize-use-trailing-return-type,bugprone-*,cppcoreguidelines-*,
  -cppcoreguidelines-pro-bounds-pointer-arithmetic
WarningsAsErrors: ''
HeaderFilterRegex: '.*'
```

---

## 11 Pre-Commit Checklist (Copy & Paste)

```
- [ ] UTF-8 without BOM
- [ ] `#include` order & include guards are correct
- [ ] `clang-format --dry-run` shows no diff
- [ ] clang-tidy has no warnings
- [ ] No exceptions/RTTI/dynamic_cast
- [ ] Member variables `m_xxx`, static `s_xxx`
- [ ] Braces for single-statement `if/for/while`
- [ ] Trailing comma in enums
- [ ] `QStringLiteral` / `u""_qs`
- [ ] `QObject`: no copy/move/by-value containers; use parent-child/`deleteLater()`, no manual `delete`; non-`QObject` uses `std::unique_ptr`/RAII
- [ ] Use QtConcurrent for long-running threaded tasks
- [ ] No exceptions: do not add `throw`/`try`/`catch`; "may fail" APIs report failure explicitly and are `[[nodiscard]]`
- [ ] Signals/slots: lambda/functor connect must have context; cross-thread uses explicit `Qt::QueuedConnection`; queued parameter types are registered as metatypes
- [ ] Meta-object: mutable `Q_PROPERTY` must have `NOTIFY`; setters emit only when changed; custom types are registered when used with QVariant/QML
```

---

**Document Package Version**: v1.0.6  
**Last Updated**: 2026-01-17

---
