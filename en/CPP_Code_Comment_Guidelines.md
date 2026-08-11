# C++/Qt/QML Code Comment Guidelines

[English](./CPP_Code_Comment_Guidelines.md) | [Simplified Chinese](../cn/CPP_Code_Comment_Guidelines.md) |
[Source](../CPP_Code_Comment_Guidelines.md)

> This document defines comment scope, content, documentation profiles, and AI/agent boundaries for
> C++/Qt/QML projects. It does not enable a documentation toolchain or create an API/ABI promise.

## 1. Scope and authority

This guideline applies to C++11 and later projects and adds rules for Qt 6, QML, C++17, and C++20
when those technologies are enabled. It governs:

- Comment content, coverage, and design rationale.
- Different obligations for public/protected, QML-facing, override, internal/private, and implementation
  code.
- Comment syntax and validation boundaries for QDoc, Doxygen, KApiDox, and Doxyqml profiles.
- How AI/agents add or update comments within an explicitly authorized task.

It does not require batch-commenting untouched legacy code, add dependencies, change build files,
expand API/ABI commitments, or synchronize a complete documentation package without authorization.

Authority order:

1. `Qt6_CPP17_Coding_Style.md` is the C++/Qt lifetime, threading, and formatting baseline.
2. This document owns comment content, coverage, and documentation-generation rules.
3. `Qt6_QML_Coding_Style.md` adds QML-specific context and must explicitly reference this guideline;
   it does not duplicate common comment rules.
4. `Qt_Macro_Layout_Coding_Style.md` owns the placement and order of Qt/moc macros, not the decision
   whether a type needs a macro.
5. `Qt6_KDE_API_Parameter_Style.md` owns public API parameter, ownership, view-lifetime, and
   QML/meta-object boundary rules.
6. `AI_CODING_BEHAVIOR.md` explains how AI applies these authorities and must not redefine them.

The presence of `Q_OBJECT` does not create an automatic comment obligation. Whether the macro is
required comes from Qt technical conditions or a published project gate.

## 2. Core principles

Comments should explain information that a clear signature and implementation cannot express:

- Stable contracts visible to callers.
- Design choices and their constraints.
- Failure, side effects, state changes, and observable behavior.
- Ownership, lifetime, thread affinity, reentrancy, and asynchronous ordering.
- QML/C++ and platform, process, file, or security boundaries.
- Why an apparently simpler implementation is not valid.

Do not restate identifiers, syntax, or individual statements. Improve naming, function boundaries, or
control flow first when that removes ambiguity.

Every "must document" rule has a trigger. It does not require every object to use the same template:

- New or semantically changed exported public/protected and QML-facing contracts must have discoverable
  documentation in the selected profile, covering non-obvious caller-visible facts.
- Internal/private code is documented only for non-obvious invariants, rationale, ownership, threading,
  timing, or protocol boundaries.
- Self-explanatory code needs no comment.
- Untouched legacy code is not batch-updated unless the task explicitly requests comment governance.

Document parameter, return, failure, side-effect, state, ownership, lifetime, threading, reentrancy,
and version fields only when they are real and applicable.

## 3. Documentation toolchain profiles

### 3.1 Identify the profile first

Before writing generator-specific comments, inspect `.qdocconf`, `Doxyfile`, CMake/ECM configuration,
KApiDox entry points, and Doxyqml integration. QDoc and Doxygen are base engines; KApiDox and
ECMAddQch are KDE integrations; Doxyqml is a Doxygen QML extension.

| Profile | Main objects | Typical location and form | Selection boundary |
|---|---|---|---|
| Qt QDoc: C++ | Qt-style C++ API | Usually `.cpp` with QDoc comments and `\` commands | Existing QDoc, Qt-native docs, or QCH route |
| Qt QDoc: QML | QML types, properties, signals, methods | QML types in `.qml`; C++-implemented QML types in `.cpp` | Qt-native QML semantics or explicit QCH requirement |
| Plain Doxygen | General C/C++ library API | Exported API usually in public headers; implementation notes in `.cpp` | Default C++ route for general third-party Qt 6 libraries |
| KDE KApiDox | KDE/Qt public library API | Doxygen-based; complete contract in installable public headers | KDE/ECM documentation stack |
| Doxyqml | QML API in a Doxygen route | Parses documentation in QML sources | Add when a Doxygen project publishes QML API |
| No generator | Source comments only | `//` or the existing block-comment style | No developer API documentation; no generator-specific commands |

With no confirmed profile, use ordinary source comments and do not add dependencies, configuration,
build targets, QCH output, or a warning gate.

### 3.2 Templates versus enabled toolchains

Configuration templates are references with placeholders. They do not prove that a generator is
installed, connected to the build, or able to produce documentation. The six profiles do not map to
six independent files:

| Profile combination | Configuration shape | Boundary |
|---|---|---|
| Qt QDoc C++ + QML | One `.qdocconf` | C++ and QML are normally organized by one QDoc profile |
| Plain Doxygen | One `Doxyfile` | Inputs, outputs, commands, and warning policy are project-specific |
| KDE KApiDox | Doxygen config plus KDE/ECM/KApiDox entry | Not a portable standalone file |
| Doxyqml | Doxygen config plus QML extension integration | Cannot run without base Doxygen |
| No generator | No configuration file | Maintain source comments only |

Non-enabled templates are listed in [`templates/documentation/README.md`](../templates/documentation/README.md).

#### 3.2.1 Default selection matrix

This is a manual selection aid for general third-party Qt 6/QML open-source projects, not an enabled-
toolchain declaration. Existing configuration takes precedence; projects that do not publish developer
API documentation remain in no-generator mode.

| Project fact or goal | Recommended profile | Reason |
|---|---|---|
| Existing QDoc configuration, or explicit Qt-native docs/QCH | Qt QDoc: C++ + QML | Preserve Qt documentation semantics and QCH generation |
| Existing Doxygen configuration | Plain Doxygen | Preserve the existing comments and entry point |
| Existing Doxygen and published QML API | Plain Doxygen + Doxyqml | Add Doxyqml only as the QML extension |
| Doxygen-style comments but no documentation configuration | Plain Doxygen comment form; toolchain remains no-generator | Keep syntax without adding dependencies or config |
| General third-party Qt 6/QML library publishing C++ API | Plain Doxygen | Default C++ route when no generator is already selected |
| The same project also publishes QML API | Plain Doxygen + Doxyqml | Do not add Doxyqml when no QML API is published |
| KDE/ECM public library | KDE KApiDox; add Doxyqml for QML API | Use KDE public API and build integration |
| No developer API documentation | No generator | Keep source comments and do not create generator config |

Plain Doxygen is the default route for a general third-party Qt 6/C++ public API. Add Doxyqml on the
same Doxygen route only when QML is a published public API. This is a selection recommendation, not an
authorization for AI/agents to add a toolchain. Choose QDoc C++/QML when Qt-native QML semantics or
QCH is the explicit goal.

Templates must use placeholders for project name, version, source directories, public include paths,
QML module/import paths, output directories, and QCH metadata, such as `@PROJECT_NAME@`, `@VERSION@`,
`@SOURCE_DIRS@`, `@PUBLIC_INCLUDE_DIR@`, `@QML_IMPORT_PATH@`, and `@OUTPUT_DIR@`.

A profile is considered enabled only when:

1. A real `.qdocconf`, `Doxyfile`, KApiDox/ECM entry, or Doxyqml integration is present.
2. The configuration is connected to a documented command, build target, or manual generation entry
   using real project paths and version metadata.
3. Required tools and dependencies are declared or installed, and a task-appropriate generation check
   has completed.

QCH is an output or build integration of the selected QDoc or Doxygen/KDE engine, not a separate
profile. Use QDoc's QHP/QCH chain, or Doxygen's QHP/QHelpGenerator or ECMAddQch integration already
adopted by the project.

### 3.3 Syntax and location are profile-specific

- Doxygen supports `///`, `/** ... */`, `/*! ... */`, and both `@` and `\` command prefixes. Choose
  one primary style within the selected profile; other legal forms are not parse errors.
- QDoc commonly uses `/*! ... */` and `\` commands such as `\class` and `\qmltype` when the
  selected object and configuration require them.
- Header versus `.cpp` placement is selected by the generator. Do not impose one location across QDoc,
  Doxygen, and KApiDox.
- Keep implementation comments next to the constrained code. Avoid duplicating one contract in multiple
  authoritative locations.

## 4. Basic comment rules

1. Use `//` for implementation rationale; use the selected profile's form for generated documentation.
2. Keep comments concise, precise, and consistent with the project's terminology.
3. Follow the project's 100-column soft baseline and split by meaning. `ReflowComments: false` means
   clang-format will not reflow comments; it does not excuse unreadable paragraphs.
4. Keep a comment next to the contract it explains and update it with the same change.
5. Describe caller-observable behavior, not replaceable implementation details.
6. Internal comments explain why and invariants, not accidental implementation facts.
7. Examples must satisfy the project's clang-format baseline and contain no unused variables or narration.

## 5. Layered comment obligations

| Object | Default obligation for new or semantically changed code |
|---|---|
| Exported public/protected API | Stable contract and applicable non-obvious caller constraints |
| QML-facing type, method, property, or signal | QML-visible behavior, with applicable defaults, failure semantics, handlers, and parameters |
| Override | Inherit the base contract; document only behavioral differences |
| Internal/private API | Non-obvious invariants, lifetime, threading, timing, or protocol boundaries |
| Implementation logic | Rationale and constraint source, not code narration |
| TODO/FIXME | A real issue, owner, or other traceable context; future cleanup also has a date/event or removal condition |

### 5.1 Public C++ API

For exported public/protected types, methods, factories, and loaders, document the stable contract
and applicable failure, side-effect, state, threading, or reentrancy constraints. Do not infer API/ABI
compatibility from visibility alone.

```cpp
/// Starts an asynchronous load; an existing request is canceled first.
/// @param source Absolute URL to load; empty or unsupported schemes are invalid.
/// @return A request ID; invalid input returns an invalid ID and leaves state unchanged.
RequestId startLoad(const QUrl &source);
```

### 5.2 QML-facing API

Document QML-visible behavior for `Q_INVOKABLE`, public slots, QML singletons, attached-property
providers, models, properties, and signals when defaults, ranges, read-only state, side effects,
failure semantics, trigger conditions, handlers, or parameter meaning are not obvious. Do not add
per-property or per-signal noise for self-explanatory members.

### 5.3 QObject lifetime and ownership

Document non-obvious ownership, invalidation, thread affinity, and event-loop destruction. Prefer
parent ownership when a dynamic child shares its owner's lifetime. `deleteLater()` is for event-driven,
cross-thread, or queued destruction when the object's thread can process deferred-delete events. Do not
write a universal "never delete QObject" rule.

For a shared-owned QObject, also document the custom `deleteLater()` deleter, when the last owner is
released, the target event-loop shutdown order, and `destroyed()` or equivalent evidence of actual
destruction.

```cpp
// The worker has no parent because it moves to a worker thread.
connect(worker, &Worker::finished, worker, &QObject::deleteLater);
```

### 5.4 C++17 and C++20 features

Comment domain meaning, not feature names. In particular, `std::span` is a non-owning C++20 view, and
`std::variant` is a tagged union, not a container. C++20 makes many `std::vector` operations usable
in constant evaluation, but a non-empty vector object is generally not a persistent constant expression;
use `std::array` for fixed compile-time data.

## 6. Doxygen, QDoc, and metadata

- In an enabled generator profile, new or changed exported APIs need discoverable documentation.
- When a Doxygen completeness gate is enabled, cover real `@param`, `@tparam`, and applicable return
  semantics; do not create empty tags.
- Use `@since`/`\since` only with a real project version. `@deprecated`/`\deprecated` includes a
  real version, reason, and replacement path. AI/agents must not invent versions, issue IDs, or owners.
- Overrides inherit the base contract by default. Doxygen may use `@copydoc`; QDoc may use `\reimp`.
  Record any behavior or pre/postcondition difference explicitly.
- Do not globally ban `\class` or `\fn`; use structure commands only when the selected generator or
  non-adjacent documentation requires them.

## 7. Forbidden comment patterns

Do not add line-by-line narration, comments that restate code, stale design history, empty metadata,
untraceable TODOs, invented versions or owners, or Qt/KDE compatibility/lifetime policies presented as
universal facts. Do not mix generator commands before identifying the profile, claim generated output
or zero warnings without a configured toolchain, or expand a comment task into repository-wide cleanup.

## 8. AI/agent workflow

1. Identify the allowed files, code paths, language standard, Qt version, public ABI/API, and project authority.
2. Read existing QDoc, Doxygen, KApiDox, Doxyqml, ECM, and QCH configuration; with no confirmed profile,
   use ordinary source comments and do not add a toolchain.
3. Classify the object as public/protected, QML-facing, override, internal/private, or implementation.
4. Comment only new or changed non-obvious contracts and constraints; do not invent metadata.
5. Respect a published `Q_OBJECT` project gate when present; otherwise apply Qt's technical conditions
   and upstream recommendations without automatic whole-tree edits.
6. Preserve proven ownership and lifetime; choose parent ownership, RAII, direct destruction, or
   `deleteLater()` only when owner, thread, event state, and event-loop conditions are demonstrable.
7. Run only scope-matched format, link, build, test, or documentation checks. Report an unconfigured
   generator as unconfigured; do not claim zero-warning output.

## 9. Verification checklist

- [ ] New or changed public/protected and QML-facing contracts record applicable non-obvious facts.
- [ ] Overrides do not duplicate the base contract; differences are explicit.
- [ ] Internal comments explain invariants, rationale, lifetime, threading, timing, or protocols.
- [ ] No mechanical fields, stale text, or invented metadata were added.
- [ ] Comment syntax, commands, and placement match the selected profile.
- [ ] Text follows the 100-column soft baseline and examples follow clang-format.
- [ ] Relative links work and comments stay synchronized with code.
- [ ] Generator/QCH checks run only when configuration and task scope require them.
- [ ] Unconfigured toolchains are reported as unconfigured.

---

**Document package version**: v1.2.0
**Last updated**: 2026-08-11
