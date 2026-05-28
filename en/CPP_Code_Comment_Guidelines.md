# C++ Code Comment Guidelines for AI-Generated Code (C++17/20 Additions)
(Compatible with C++11 and later; concise and non-redundant)

English | Simplified Chinese | Source

> Note: This document is the English translation of the current C++ comment guideline. If it differs from the guideline baseline, the guideline baseline prevails.

## Core Goal
When generating C++ code, add comments according to the following rules. Keep only essential information and avoid redundant comments that make code harder to read.

## Basic Comment Rules
1. Follow common C++ commenting conventions; prefer `//` for single-line comments.
2. Use `///` (a simplified Doxygen style) for high-level descriptions of classes/functions; avoid excessive formatting.
3. Keep comments concise and precise. Limit each comment to 80 characters where practical. Split complex explanations into multiple short comments.

## When Comments Are Required (General + C++17/20 Specific)
### General (All C++ Versions)
1. Classes/structs/enums: describe the core responsibility or design intent (e.g., "A struct that manages a thread-pool task queue").
2. Non-obvious member variables: explain the exact meaning (e.g., "Accumulated timeout in milliseconds for unprocessed tasks").
3. Functions: describe the purpose, non-obvious parameters, non-obvious return values, and key side effects (e.g., "Modifies the global config table; not thread-safe").
4. Complex logic blocks: add comments only when the algorithmic intent or branch decision is not obvious (e.g., "Use quicksort because the dataset is small and memory overhead matters more").
5. Macros/constants: explain where the value or logic comes from (e.g., "Max connections is 1024, 80% of the system FD limit").
6. Special handling: note failure conditions, error-handling constraints, thread-safety constraints, etc. (e.g., "Requires external locking; not thread-safe").

### Qt/QML and Qt/KDE Library-Specific Cases
(Prerequisite: the project uses Qt/QML, or organizes a C++ library in a Qt/KDE library style.)
1. **Public API contracts**:
   For public classes/structs/enums, public/protected methods, `virtual` overrides, factory functions, parsers/loaders, and start/stop/request/dispatch-style functions, document the API purpose, valid parameter ranges, return value meaning, failure conditions, key side effects, thread-safety or reentrancy limits, and whether the call changes object state.
2. **QML-exposed APIs and property contracts**:
   For `Q_INVOKABLE`, public slots, QML singletons, attached property providers, QML-facing models, and other interfaces exposed to QML, document the purpose, return value, failure conditions, side effects, and call constraints. Comment `Q_PROPERTY` only when its semantics, default value, side effects, stability, or failure behavior are not obvious. Prefer documenting these in the class description, related getter/setter, or property group description to avoid per-property comment noise.
3. **QObject lifecycle and ownership**:
   When `QObject`, `QQuickItem`, raw pointers, `QPointer`, event filters, lambda captures, or `deleteLater()` involve non-obvious ownership, document who owns the object, when it is released, whether the pointer is observation-only, and whether destruction depends on the event loop (e.g., "QObject parent ownership deletes the source with the session.").
4. **QML/C++ data-boundary contracts**:
   For `QVariantMap`, `QVariantList`, model roles, role names, QML-dependent string fields, restore keys, trace payloads, and other data structures that cross the QML/C++ boundary, document stable fields, invalid input handling, null/empty semantics, and return conventions.
5. **QAbstractItemModel / role / model data contracts**:
   For `QAbstractItemModel`, the relationship between model rows and business objects, role names, stable keys, restore paths, empty models, and invalid indexes, document the stable boundaries depended on by QML or tests.
6. **Qt event loop, asynchrony, and reentrancy**:
   For `Qt::QueuedConnection`, `QMetaObject::invokeMethod(..., Qt::QueuedConnection)`, deferred operations, transaction queues, debounce/throttle logic, timer lifetimes, reentrancy guards, signal ordering dependencies, and logic that must wait for the next event-loop turn, document the timing reason and protection intent instead of explaining Qt syntax.
7. **Diagnostics, error codes, and observability contracts**:
   For deny codes, trace event kinds, debug payloads, logging categories, stable strings depended on by contract tests, and similar items, document that they are externally observable contracts and describe the boundary depended on by QML, tests, or tools.
8. **Library API public/internal/private boundaries**:
   Adopt the Qt/KDE library comment rules for public/private headers, private implementation, exported symbols, thread affinity, signal semantics, deprecation notes, and replacement paths. Do not make API/ABI compatibility a default promise. If the project explicitly does not guarantee compatibility, comments should state the public/internal/private boundary and call constraints instead of promising stability.
9. **Platform, security, and file-access boundaries**:
   For debug-only behavior, `file://` reads, sandbox/portal permissions, Wayland/X11/Linux-only branches, platform fallbacks, and rejection policies, document the reason for the limit, failure behavior, and security boundary.

### C++17-Specific Cases
(Prerequisite: the project enables C++17 or later.)
1. **Structured bindings**:
   When the bound object is complex (e.g., nested structs or multi-element tuples), comment the actual meaning of each binding variable (e.g., "[id: unique user identifier, score: exam score]").
2. **`if constexpr`**:
   Comment the distinguishing logic of the compile-time branches (e.g., "Bitwise optimization only for integral types"), rather than repeating the syntax.
3. **`std::optional` / `std::variant`**:
   - For `std::optional`: comment the default behavior when empty (e.g., "Empty means the device was not detected").
   - For `std::variant`: comment the primary type branch in practice (e.g., "Prefer handling `string`; fall back to the default value for other types").
4. **Fold expressions**:
   Comment the aggregation logic (e.g., "Sum the parameter pack; ignore negatives"), not the syntax itself.
5. **Inline variables**:
   Comment the cross-translation-unit sharing intent (e.g., "Global config singleton; initialization order matters").

### C++20-Specific Cases
(Prerequisite: the project enables C++20 or later.)
1. **Concepts**:
   Comment the core purpose of the constraint (e.g., "Requires random-access iterators"), rather than restating the constraint itself.
2. **Coroutines**:
   - Comment the await condition for `co_await` (e.g., "Wait for the IO completion signal").
   - Comment the meaning of produced values for `co_yield` (e.g., "Yield the processing result of the current frame").
3. **Ranges**:
   For complex pipelines (e.g., `views::filter | views::transform`), comment the overall data-processing goal (e.g., "Filter valid users and extract IDs").
4. **Modules**:
   Comment the dependency intent of `import`/`export` (e.g., "Import base utilities; no cyclic dependencies").
5. **`constexpr` containers (e.g., `constexpr std::vector`)**:
   Comment the special compile-time initialization logic (e.g., "Precompute the prime table at compile time to reduce runtime overhead").
6. **Three-way comparison (`<=>`)**:
   Comment special comparison rules (e.g., "Compare by length first, then by content"). If it is the default member-wise comparison, no comment is needed.

## When Comments Are Not Needed
1. Simple, obvious operations (e.g., `i++`, `if (flag)`).
2. Variables with clear meaning (e.g., `int count`, `bool is_ready`) when the name already explains the role.
3. Routine functions (e.g., simple getters/setters, constructors/destructors without special logic) when the name already explains the function.
4. Comments that merely restate code (e.g., `// assign a` for `a = 5;`).
5. Purely syntactic explanations of C++17/20 features (e.g., no need to comment "this is a structured binding" or "this is a concepts constraint"); only comment non-obvious logic.
6. Line-by-line narration, historical chat context, or design notes that are not kept in sync with the code.
7. TODOs without an owner, trigger condition, or follow-up action.
8. Per-item comment noise for every `Q_PROPERTY`, simple signal, or obvious role.
9. Treating KDE/Qt library compatibility promises as default requirements. Record API/ABI stability only when the project explicitly commits to it.

---

**Document Package Version**: v1.0.7
**Last Updated**: 2026-05-28
