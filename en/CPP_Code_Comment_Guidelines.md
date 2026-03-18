# C++ Code Comment Guidelines for AI-Generated Code (C++17/20 Additions)
(Compatible with C++11 and later; concise and non-redundant)

English | 简体中文 | Source

> Note: This document is the English translation of the current C++ comment guideline. If there is any discrepancy, the package baseline prevails.

## Core Goal
When generating C++ code, add comments according to the following rules. Keep only essential information and avoid redundant comments that make code harder to read.

## Basic Comment Rules
1. Follow common C++ commenting conventions; prefer `//` for single-line comments.
2. Use `///` (a simplified Doxygen style) for high-level descriptions of classes/functions; avoid excessive formatting.
3. Keep comments concise and precise; limit each comment to ~80 characters. For longer explanations, split into multiple short comments.

## When Comments Are Required (General + C++17/20 Specific)
### General (All C++ Versions)
1. Classes/structs/enums: describe the core responsibility or design intent (e.g., "A struct that manages a thread-pool task queue").
2. Non-obvious member variables: explain the exact meaning (e.g., "Accumulated timeout (ms) for unprocessed tasks").
3. Functions: describe the purpose, non-obvious parameters, non-obvious return values, and key side effects (e.g., "Modifies the global config table; not thread-safe").
4. Complex logic blocks: only add comments when the algorithmic intent or branch decision is not obvious (e.g., "Use quicksort because the dataset is small and memory overhead matters more").
5. Macros/constants: explain where the value/logic comes from (e.g., "Max connections is 1024, 80% of the system FD limit").
6. Special handling: note failure conditions/error-handling constraints, thread-safety constraints, etc. (e.g., "Requires external locking; not thread-safe").
7. Qt/QObject lifecycle: when creating a `QObject`-derived object without relying on the parent-child tree for automatic destruction, add a short comment indicating that it is destroyed via `deleteLater()` (e.g., "Destroyed by event loop to avoid manual delete").

### C++17-Specific
(Prerequisite: the project enables C++17 or later)
1. **Structured bindings**:
   When the bound object is complex (e.g., nested structs or multi-element tuples), comment the meaning of each binding (e.g., "[id: unique user identifier, score: exam score]").
2. **`if constexpr`**:
   Comment the *distinguishing logic* of the compile-time branches (e.g., "Bitwise optimization only for integral types"), rather than repeating the syntax.
3. **`std::optional` / `std::variant`**:
   - For `std::optional`: comment the default behavior when empty (e.g., "Empty means the device was not detected").
   - For `std::variant`: comment the primary type branch in practice (e.g., "Prefer handling `string`; fall back to default for other types").
4. **Fold expressions**:
   Comment the *aggregation logic* (e.g., "Sum the parameter pack; ignore negatives"), not the syntax itself.
5. **Inline variables**:
   Comment the *cross-translation-unit sharing intent* (e.g., "Global config singleton; initialization order matters").

### C++20-Specific
(Prerequisite: the project enables C++20 or later)
1. **Concepts**:
   Comment the core intent of the constraint (e.g., "Requires random-access iterators"), rather than restating the constraint.
2. **Coroutines**:
   - Comment the *await condition* for `co_await` (e.g., "Wait for the IO completion signal").
   - Comment the meaning of produced values for `co_yield` (e.g., "Yield the processing result of the current frame").
3. **Ranges**:
   For complex pipelines (e.g., `views::filter | views::transform`), comment the overall data-processing goal (e.g., "Filter valid users and extract IDs").
4. **Modules**:
   Comment the dependency intent of `import`/`export` (e.g., "Import base utilities; no cyclic dependencies").
5. **`constexpr` containers (e.g., `constexpr std::vector`)**:
   Comment any special compile-time initialization logic (e.g., "Precompute prime table at compile time to reduce runtime overhead").
6. **Three-way comparison (`<=>`)**:
   Comment any special comparison rules (e.g., "Compare by length first, then by content"). If it is the default member-wise comparison, no comment is needed.

## When Comments Are Not Needed
1. Simple, obvious operations (e.g., `i++`, `if (flag)`).
2. Variables with obvious meaning (e.g., `int count`, `bool isReady`).
3. Routine functions (e.g., simple getters/setters, constructors/destructors without special logic).
4. Comments that merely restate code (e.g., `// assign a to 5` for `a = 5;`).
5. Purely syntactic explanations of C++17/20 features (e.g., "this is a structured binding", "this is a concept"); only comment non-obvious logic.

---

**Document Package Version**: v1.0.6  
**Last Updated**: 2026-01-17
