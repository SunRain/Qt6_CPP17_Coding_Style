# C++ 注释规范与 AI 编码行为完整审查结论

> 文档性质：只读审查结论、后续修订依据和 AI/agent 执行边界，不替代当前已发布规范。
>
> 执行边界：本文不是可直接粘贴给 AI/agent 的代码生成提示词。除非任务明确授权实施，AI/agent 不得据此修改源码、现行规范、构建配置、文档工具链、版本信息或全仓库既有代码。
>
> 审查对象：[`CPP_Code_Comment_Guidelines.md`](../CPP_Code_Comment_Guidelines.md) 与
> [`AI_CODING_BEHAVIOR.md`](../AI_CODING_BEHAVIOR.md)。
>
> 核对日期：2026-08-06。

## 1. 最终裁决

两份文档的核心方向成立：注释应描述契约、原因和非显然边界，避免重复代码字面行为；AI 应区分强制规范、可选推荐和项目特定约定。

现有内容仍不能直接视为完整的 Qt 6/QML、KDE library 和 Doxygen 最佳实践。主要问题包括：

- Qt QDoc、KDE KApiDox/Doxygen 和普通源代码注释的规则边界没有明确区分。
- 若干项目强化政策被表述为 Qt 的普遍事实。
- C++17/C++20 类型分类和示例存在事实错误。
- 注释覆盖义务过于统一，没有按 public、QML-facing、internal/private 和 override 分层。
- 文档生成警告门禁缺少当前仓库的生成器配置支撑。
- `AI_CODING_BEHAVIOR.md` 中的部分代码示例不符合项目自己的 clang-format 基线。
- 审查结论同时包含事实判断、项目政策和实施路线，若直接作为 AI/agent 输入，容易把建议升级为门禁并扩大修改范围。

因此，最终修订方向是：保留“契约优先、避免流水账”的核心目标；先建立权威关系和文档工具链 profile，再修正注释义务、Qt/C++ 事实、生命周期政策和示例；最后把可执行范围收敛到可检测的触发条件、明确的文件范围和停止条件。

## 2. 审查依据与边界

### 2.1 上游与开源项目依据

- [Qt C++ Documentation Style](https://doc.qt.io/qt-6/qtwritingstyle-cpp.html)
- [Qt QML Documentation Style](https://doc.qt.io/qt-6/qtwritingstyle-qml.html)
- [QObject](https://doc.qt.io/qt-6/qobject.html#details)
- [QObject::deleteLater()](https://doc.qt.io/qt-6/qobject.html#deleteLater)
- [Doxygen: Documenting the code](https://www.doxygen.nl/manual/docblocks.html)
- [Doxygen: Special commands](https://www.doxygen.nl/manual/commands.html)
- [Doxygen: Configuration](https://www.doxygen.nl/manual/config.html)
- [KDE Frameworks Documentation Policy](https://community.kde.org/Policies/Frameworks_Documentation_Policy)
- [KApiDox](https://invent.kde.org/frameworks/kapidox)
- [ECMAddQch](https://api.kde.org/ecm/module/ECMAddQch.html)
- [LLVM Coding Standards](https://llvm.org/docs/CodingStandards.html)
- [Google C++ Style Guide: TODO Comments](https://google.github.io/styleguide/cppguide.html#TODO_Comments)

### 2.2 项目内权威

- [`Qt6_CPP17_Coding_Style.md`](../Qt6_CPP17_Coding_Style.md)：C++、Qt 生命周期和基础格式总纲。
- [`Qt6_QML_Coding_Style.md`](../cn/Qt6_QML_Coding_Style.md)：QML 专题增量。
- [`Qt_Macro_Layout_Coding_Style.md`](../Qt_Macro_Layout_Coding_Style.md)：Qt/QML/moc 宏布局。
- [`Qt6_KDE_API_Parameter_Style.md`](../Qt6_KDE_API_Parameter_Style.md)：Public API 参数和所有权语义。
- [`Qt6_CPP17_CLANG-FORMAT`](../Qt6_CPP17_CLANG-FORMAT)：格式基线，其中 `ColumnLimit` 为 100，`ReflowComments` 为 `false`，函数定义左花括号换行。

### 2.3 文档工具链与输出 profile 差异

以下项目不是同级的“六种生成器”：QDoc 和 Doxygen 是文档引擎；KApiDox 是 Doxygen 的 KDE 包装层；ECMAddQch 是基于 Doxygen 的 QCH/CMake 集成；Doxyqml 是 QML 解析扩展；“无生成器”是项目模式。规范应按完整工具链选择 profile，不能把包装层、输出格式和引擎混为一类。

| Profile / 工具 | 角色与主要文档位置 | 常见注释与命令 | 适用结论 |
|---|---|---|---|
| Qt QDoc C++ | 文档引擎；C++ 文档位于 `.cpp` 实现文件 | `/*! ... */`、反斜杠命令 | 仅在项目明确采用 Qt Project 的 QDoc 体系时选择 |
| Qt QDoc QML | QDoc 的 QML profile；QML 类型在 `.qml`，C++ 实现的 QML 类型在 `.cpp` | `\qmltype`、`\qmlproperty`、`\qmlsignal` 等 | QML 文档需要独立的 QDoc 规则 |
| 普通 Doxygen | 文档引擎；声明处或定义处均可，公共库通常以 public header 为主 | `///`、`/** ... */`、`/*! ... */`，`@` 与 `\` 均受支持 | C++ 公共 API 的常用候选，不是 Qt 强制默认 |
| Doxygen + Doxyqml | Doxygen 加 QML 解析扩展；C++ public header 与公开 `.qml` API | Doxygen/Doxyqml 路线 | 已选择 Doxygen 且需要统一生成 QML API 时的可选组合 |
| KDE KApiDox | Doxygen 的 KDE 包装层；公共 API 以 public header 为主 | KDE 惯用 `/** ... */` 与 `@` 命令 | KDE Framework、KDE 应用或遵循 KDE 元数据结构的项目 |
| KApiDox QML | KApiDox 加 Doxyqml；公开 QML API 位于 `.qml` | Doxygen/Doxyqml 路线 | KDE 项目的 QML API，不能等同于 QDoc |
| ECMAddQch | 基于 Doxygen 和 `qhelpgenerator` 的 QCH/CMake 集成 | `ecm_add_qch()`、Doxygen tag file、导出 QCH target | 已依赖 ECM 且需要生成、安装或跨库链接 `.qch` 的项目 |
| 无生成器项目 | 不发布开发者 API 文档；注释放在最接近维护点的位置 | 普通 `//` 或项目约定 | 不提供公共 SDK/API 的终端应用，不应虚构生成文档门禁 |

### 2.4 第三方 Qt 开源项目推荐选型

- 发布 C++ 公共 API：Doxygen 是常用、可维护的候选，但不是 Qt 上游规定的唯一默认。
- 同时发布 QML 公共 API：如果项目已采用 Doxygen，可增加 Doxyqml；QML-first 或需要 Qt 原生 QML 文档语义时，也可以选择 QDoc。
- 不发布公共 SDK/API：可以不引入 API 文档生成器，只维护用户文档和必要源码注释。
- 明确采用 Qt Project 文档体系，或以 Qt 原生 C++/QML 文档为主要产物：选择 QDoc。
- 已属于 KDE 生态并采用相应元数据结构：选择 KApiDox。

需要额外生成 Qt Compressed Help（`.qch`）时，不应仅为该输出切换基础文档引擎：

- 已依赖 ECM：在 Doxygen 基础上使用 ECMAddQch，生成 QCH、tag file、安装规则和可供其他库引用的 CMake target。
- 已选择 Doxygen 但不依赖 ECM：配置 `GENERATE_QHP=YES`、`QHP_NAMESPACE`、`QHP_VIRTUAL_FOLDER` 和 `QHG_LOCATION`，由 Qt `qhelpgenerator` 生成 `.qch`。
- 已选择 QDoc：沿用 QDoc 的 QHP 配置和 `qhelpgenerator` 输出，不为 `.qch` 重新切换引擎。
- 不应仅为 `.qch` 切换基础引擎，也不应仅为此引入完整 KApiDox。

### 2.5 AI/agent 的 profile 识别与工具链边界

AI/agent 只能从项目已经存在的配置识别文档 profile，不得因为新增几条注释而自行引入或切换生成器：

1. 存在 `.qdocconf` 或 QDoc 构建入口时，使用 QDoc profile。
2. 存在 `Doxyfile`、Doxygen CMake 配置或 KApiDox 入口时，使用对应 Doxygen profile。
3. 存在 `ecm_add_qch()` 时，使用 ECMAddQch 作为 QCH 集成。
4. 没有可确认的配置时，按“无生成器”处理，使用项目已有的普通源码注释约定，不新增工具链配置。
5. `.qch`、HTML、tag file 和零警告只在任务明确要求文档产物且已有或获授权新增配置时处理。

## 3. 最终权威关系

1. `Qt6_CPP17_Coding_Style.md` 继续作为总纲。
2. `CPP_Code_Comment_Guidelines.md` 只负责注释内容、覆盖范围和生成文档规则。
3. `Qt6_QML_Coding_Style.md` 只维护 QML 增量，必须显式引用 `CPP_Code_Comment_Guidelines.md`，不重复通用注释全文。
4. `Qt_Macro_Layout_Coding_Style.md` 只决定宏的布局；是否需要 `Q_OBJECT` 由 Qt 技术硬要求、Qt 上游强烈建议和已经明确发布的项目门禁共同决定，不能由 AI/agent 自行升级。
5. `AI_CODING_BEHAVIOR.md` 只解释 AI 如何应用现有权威，不应重新定义或扩大底层规则。
6. 当总纲或专题存在事实错误时，应修正权威源；不能只在 AI 文档中绕开或复制错误。

## 4. `CPP_Code_Comment_Guidelines.md` 审查结论

### 4.1 应保留的内容

- 保留“只注释关键信息，避免冗余”的核心目标。
- 保留对失败条件、副作用、线程安全、所有权、异步时序和外部可观测契约的关注。
- 保留 QML/C++ 数据边界、model role、稳定字段、平台限制和安全边界等专题。
- 保留“不默认承诺 API/ABI 兼容性”；只有项目明确承诺时才记录稳定性和兼容窗口。
- 保留禁止逐行解释、重复代码语义、记录聊天历史和维护失真的注释。

### 4.2 必须修订的内容

1. **增加适用范围和权威关系**：文首应明确本规范与总纲、QML 专题、宏布局和 API 参数专题的关系。
2. **增加工具链选择规则**：下游项目必须先确认完整文档工具链，再应用对应语法和位置规则；Doxygen、QDoc、KApiDox 和 Doxyqml 是场景化选择，不是统一的 Qt 强制默认。没有现行配置时，不要求 AI/agent 新增生成器。
3. **取消 80 字硬限制**：改为遵循项目 100 列软基线，按语义分段；`ReflowComments: false` 表示工具不自动重排注释，不代表可以忽略可读性。
4. **不禁止合法 Doxygen 形式**：Doxygen 支持多种注释块和命令前缀。规范只能要求选定 profile 内保持一致，不能把其他合法形式描述为解析错误。
5. **不统一规定注释位置**：Qt QDoc 与 KDE Doxygen 对 C++ 公共文档位置的惯例不同，位置必须由 profile 决定。
6. **按接口层级规定义务**：public/protected、QML-facing、internal/private 和 implementation comment 应采用不同覆盖强度。
7. **公共 API 按适用性记录契约**：用途、参数、返回、失败、副作用、状态变化、所有权、生命周期、线程和重入信息只在适用时填写，不机械堆齐所有字段。
8. **公共枚举记录稳定语义**：说明枚举用途和非显然枚举值；internal/private 的自解释枚举不要求逐值注释。
9. **补齐 QML 属性契约**：在行为、默认值、有效范围、只读性、副作用或失败语义非显然且适用时进行说明；不机械补齐空字段。
10. **补齐 QML 信号契约**：在触发条件、对应 handler 或参数语义非显然且适用时进行说明；简单内部信号不要求为生成文档而堆注释。
11. **限制内部注释范围**：internal/private 代码只注释非显然不变量、算法原因、所有权、线程、时序和协议边界。
12. **明确 override 规则**：默认继承基类文档，只记录行为、前后置条件或失败语义差异。Doxygen 可按需使用 `@copydoc`，QDoc 可使用 `\reimp`。
13. **补充生成文档完整性**：在选定 profile 且任务要求生成文档时检查 `@param`、`@tparam` 和返回语义；`@since` 只用于项目有版本化约定时新增的稳定 API，`@deprecated` 应包含真实版本和替代路径。AI/agent 不得编造版本、owner 或 issue ID。
14. **按生成器使用结构命令**：不能全局禁止 `\class`、`\fn`。QDoc 类文档可能需要 `\class`；`\fn` 只在注释无法紧邻实现时使用。
15. **修正 TODO 规则**：TODO 应包含 bug ID、owner 或其他可追踪上下文之一。只有未来日期或事件驱动的 TODO 才要求具体日期、事件或移除条件。
16. **收敛 C++17/20 专题**：可以把特性逐项说明压缩为“只注释非显然语义”的统一规则，但这是维护性优化，不是上游硬性要求。
17. **限定文档警告门禁**：当前仓库没有 Doxyfile、QDoc 配置或生成构建入口。零警告只能作为配置完善后的本仓库门禁，或作为下游项目门禁；生成 `.qch` 应沿用已确认的 QDoc 或 Doxygen 工具链，并按是否依赖 ECM 选择对应集成方式。

### 4.3 推荐的注释义务矩阵

| 对象 | 默认义务（仅适用于新增或语义发生变化的接口） |
|---|---|
| 导出的 public/protected API | 记录稳定契约和适用的调用约束；只填写实际存在且非显然的字段 |
| QML-facing 类型、方法、属性和信号 | 记录 QML 可见行为以及适用的默认值、失败语义、handler 和参数 |
| override | 只记录相对基类的行为差异；无差异时继承基类文档 |
| internal/private API | 只记录非显然不变量、生命周期、线程、时序和协议边界 |
| 实现逻辑 | 解释“为什么”和约束来源，不复述“做了什么” |
| TODO | 已有真实 bug ID、owner 或其他可追踪上下文时再记录；未来清理型 TODO 还应记录明确日期或事件 |

## 5. `AI_CODING_BEHAVIOR.md` 审查结论

### 5.1 应保留的内容

- 保留强制规范、可选推荐、项目特定约定的三级分类。
- 保留新代码优先使用项目允许的现代写法、维护代码尊重既有接口合同的原则。
- 保留不因可选推荐主动扩大重构范围的行为边界。
- 保留相对链接、文档包版本和中英文同步要求。

### 5.2 必须修订的内容

1. **重写 AI 决策优先级**：先确认任务允许修改的范围和目标；在该范围内，先满足 Qt/C++ 技术硬要求、目标标准、public ABI/API 和已发布项目权威，再应用专题规范、现有局部风格和可选推荐。用户明确要求修改权威时，应先修改权威并同步执行，不得一边保留规则一边生成违规代码。
2. **删除 90% 统计启发式**：代码频率不能覆盖现行规范，也不能自动形成新的项目政策。
3. **限制权威文档自动更新**：AI 只能在明确的项目决策或用户授权下把新约定写入规范，不能从代码统计自行推导并固化规则。
4. **修正 `std::span` 分类**：`std::span` 是 C++20 连续内存 view，不是 C++17 容器；`std::variant` 也不应归类为容器。
5. **修正 `constexpr std::vector` 示例**：C++20 主要使其操作成为 `constexpr`；非空 vector 对象通常不能作为一般常量表达式。固定编译期数据应优先使用 `std::array`。
6. **分层落实 `Q_OBJECT`**：类型声明自身 signal、元对象可调用 slot、`Q_PROPERTY`、`Q_ENUM`、QML/插件元数据，或依赖自身派生类元对象身份时，`Q_OBJECT` 是 Qt 技术硬要求。Qt 还强烈建议其他 QObject 子类使用该宏，但一般开源 Qt 库不得据此自动建立全仓库门禁。导出的 public QObject 具体类可以在项目明确决策后设为门禁；private/internal 辅助类、模板、嵌套类、header-only 或其他 moc/build 受限类型只能在理由可追踪时处理。除非任务明确要求全树治理，AI/agent 只处理当前新增或修改的类型。
7. **取消 QObject 绝对删除禁令**：不能把“禁止直接 `delete` 所有 QObject”写成 Qt 普遍规则，也不应由 AI/agent 因此批量改写现有所有权。硬约束是不得跨线程直接删除，也不得在对象处理所接收事件时同步删除；其余场景按现有 owner、线程亲和性、事件处理状态和可证明的生命周期决定。
8. **限定直接销毁边界**：只有同线程、函数局部、不逃逸、无 parent、没有正在处理事件且没有可观测在途使用时，才可新增直接析构、栈生命周期或受控 RAII。动态子对象生命周期与 owner 一致时优先 parent ownership；不得混用 parent ownership 和其他 owning 机制。无法在当前代码范围内证明条件时，保留既有生命周期并报告不确定性，不主动重写。
9. **限定延迟销毁边界**：对象正在处理事件、销毁请求来自其他线程，或已有明确的事件驱动 worker 生命周期时，在对象所属线程安排销毁，通常使用 `deleteLater()`。queued connection 或异步回调的存在本身不自动禁止同线程析构，也不自动证明 `deleteLater()` 可用。必须说明事件循环前提：主事件循环停止后调用不会删除对象；无线程事件循环时对象通常在线程结束时销毁。
10. **替换 `deleteLater()` 示例**：创建 QWidget 后立即调度删除没有业务意义。应改为 parent ownership 示例，以及已有事件循环的 worker 完成信号连接到 `deleteLater()` 的异步生命周期示例；不得为示例臆造业务 owner 或线程模型。
11. **保留拒绝策略的治理属性**：“用户要求违反强制规范时拒绝”属于项目治理政策。Qt/KDE/Doxygen 最佳实践不能证明它必须删除，也不能证明必须增加豁免机制；AI/agent 应提供符合硬约束的替代实现，并在用户明确授权改规时先修改权威。
12. **限制文档和示例修改范围**：清理未使用的 `kDefaultAlpha`、流水账注释和格式偏差只适用于明确授权的文档修订或当前触及的示例，不得借此开展全仓库重构。
13. **禁止编造追踪和版本信息**：`@since`、`@deprecated`、TODO 的版本、日期、bug ID、owner 和替代 API 必须来自项目已有事实；缺少事实时不得编造，也不得为了填字段虚构发布版本。
14. **统一示例格式**：多个函数把左花括号放在签名同行，但项目基线要求函数定义左花括号换行；所有被修改的示例应通过发布的 clang-format 基线。
15. **约束现代 C++ 推荐**：现代写法必须受 C++ 标准版本、public ABI、QML/meta-object 边界和现有 API 契约约束，不能只因“新代码”就机械采用。
16. **限定版本联动**：根目录、中文和英文分发版、文档包版本、日期和链接只在明确的文档发布或同步任务中联动；普通代码生成不得自行修改这些文件。

### 5.3 AI/agent 可执行边界

以下规则用于防止把本审查结论误用为无边界的自动重构指令：

1. **范围先行**：只修改用户明确点名或任务合同允许的文件和代码路径；不因发现其他潜在问题而扩展范围。
2. **profile 只读识别**：先读取现有 `.qdocconf`、Doxyfile、KApiDox、ECM 或项目入口；没有可确认 profile 时不新增生成器、依赖、构建目标或 QCH 配置。
3. **规范分层执行**：技术硬要求直接执行；上游强烈建议只作为新代码的默认倾向；项目门禁只有在权威文档明确发布后才执行。
4. **`Q_OBJECT` 受触发条件约束**：只在当前新增/修改类型满足 Qt 元对象硬要求，或当前项目已明确要求该范围的门禁时处理；不对既有全仓库 QObject 子类机械补宏。
5. **QObject 生命周期保守决策**：优先保留已验证的 parent/owner 模型；只有直接析构条件在当前代码中可证明时才新增直接析构；只有事件循环和对象线程前提明确时才新增 `deleteLater()`。
6. **注释按变化增量**：只为新增或语义变化的 public/protected/QML-facing 契约补充适用的非显然信息；不为完整矩阵机械填充参数、版本、失败、线程等字段。
7. **事实不可臆造**：缺少 release version、bug ID、owner、替代 API、QML 默认值或线程模型时，不生成虚构内容；保留事实缺口并报告。
8. **权威文件不可自改**：实现任务中发现总纲与专题冲突时，先遵循当前最高权威并报告冲突；只有文档修订任务明确授权时才修改权威文件。
9. **验证与停止**：只运行与当前修改范围匹配的格式、构建、文档或测试验证；缺少工具链时报告“未配置”，不得用未执行的零警告或生成结果替代证据。

## 6. 对前序审查建议的最终裁决

| 前序建议 | 最终处理 | 原因 |
|---|---|---|
| 将“80 字”改为“80 列” | 撤回 | 项目格式基线是 100 列，且不自动重排注释 |
| 禁止混用 `///`、`/** */`、`@` 和 `\` | 撤回并改写 | Doxygen 均支持，应按生成器 profile 统一主风格 |
| 所有公共 API 文档必须放 header | 撤回并限定 | Qt QDoc 与 KDE Doxygen 的文档位置惯例不同 |
| 强制违规必须改成显式豁免 | 撤回 | 这是项目治理选择，不是外部注释最佳实践 |
| 压缩 C++17/20 章节 | 降级为维护建议 | 有助于去冗余，但不是事实正确性要求 |
| 当前仓库执行 Doxygen/QDoc 零警告 | 改为条件门禁 | 当前没有可执行的生成器配置 |
| 所有 QObject 子类添加 `Q_OBJECT` 只是纯项目强化政策 | 部分撤回并分层 | 元对象功能属于 Qt 硬要求，其他 QObject 子类属于上游强烈建议；是否升级为门禁仍是明确的项目决策 |
| 禁止直接 `delete` 所有 QObject 可作为一般库默认强化政策 | 撤回并改写 | Qt 只在跨线程直接删除、事件处理中同步删除等条件下形成硬约束；其余场景按可证明的所有权和线程条件决定 |
| 公共契约、QML 契约、版本与弃用信息 | 保留并收紧适用范围 | 符合 Qt/KDE/Doxygen 公共 API 文档实践 |
| override 不重复基类全文 | 保留并按生成器细化 | Doxygen 与 QDoc 的复用命令不同 |
| 删除 90% 风格统计规则 | 保留 | 统计不能覆盖项目权威 |

## 7. 唯一修订路线

1. 在 `CPP_Code_Comment_Guidelines.md` 文首建立权威关系和文档工具链 profile 矩阵。
2. 用 public/protected、QML-facing、override、internal/private 和实现逻辑五类对象重写注释义务。
3. 按已有配置确定 QDoc、Doxygen、KApiDox、Doxyqml 或无生成器 profile；不把任何工具链写成 Qt 上游唯一默认，也不授权 AI/agent 自行新增工具链。
4. 按工具链分别规定注释形式、命令、放置位置和警告门禁，不再维护冲突的通用规则。
5. 修正 `AI_CODING_BEHAVIOR.md` 中的 `std::span`、`constexpr std::vector`、`Q_OBJECT` 分层规则、QObject 可证明的销毁决策和 `deleteLater()` 示例。
6. 同步修正 `Qt6_CPP17_Coding_Style.md` 中涉及 `Q_OBJECT` 和 QObject 删除的交叉事实，避免 AI 文档与权威源继续冲突。
7. 清理所有示例的未使用代码、流水账注释和格式偏差，并用项目 clang-format 基线验证。
8. 只在明确的文档发布任务中同步根目录、`cn/` 和 `en/` 对应文档，并更新真实版本、日期和相对链接。
9. 仅在实际工具链配置存在后启用对应警告门禁，并验证任务要求的 HTML、tag file 或 QCH 输出。
10. 将第 5.3 节的范围、profile、事实来源、验证和停止条件写入 `AI_CODING_BEHAVIOR.md`，避免审查结论被直接当成全仓库重构授权。

## 8. 后续修订验收标准

- [ ] 文档明确声明总纲、专题和 AI 行为说明之间的权威关系。
- [ ] 文档明确区分 QDoc/Doxygen 引擎、KApiDox/ECMAddQch 集成、Doxyqml 扩展和无生成器模式。
- [ ] 文档工具链由现有配置或明确项目决策确定；没有配置时 AI/agent 不新增生成器，`.qch` 输出不迫使项目切换基础引擎。
- [ ] 不再存在通用 80 字/80 列规则，与 100 列项目基线一致。
- [ ] public、QML-facing、override 和 internal/private 的注释义务互不混淆。
- [ ] `std::span`、`std::variant` 和 `constexpr std::vector` 的标准版本与类型分类准确。
- [ ] `Q_OBJECT` 规则明确区分 Qt 技术硬要求、Qt 上游强烈建议和已发布的项目门禁；只处理任务范围内的类型，并记录 moc/build 技术例外。
- [ ] QObject 销毁规则按 parent ownership、可证明的直接销毁条件、线程亲和性和事件循环决策，不保留绝对禁令，也不因 queued connection 机械改用 `deleteLater()`。
- [ ] `deleteLater()` 示例具有真实生命周期场景并说明事件循环前提。
- [ ] TODO、弃用和版本标记规则与选定 profile 一致，且没有编造 bug ID、owner、版本、日期或替代 API。
- [ ] 所有示例通过项目 clang-format 基线，且无未使用变量或流水账注释。
- [ ] 未配置文档工具链时不声称已经通过生成文档零警告门禁。
- [ ] 只有明确的文档发布任务才同步根目录、中文和英文分发版，并使用真实版本和日期。
- [ ] AI/agent 只修改任务允许的文件，不自动更新权威、不批量补注释、不引入工具链，也不把上游建议自动升级为项目门禁。
- [ ] 验证命令和完成声明与实际修改范围及现有工具链一致；缺少配置时明确报告未验证项。

## 9. 文档状态

本文只保存审查结论、后续修订合同和 AI/agent 执行边界。本次修订没有修改两份目标规范，也没有声称上述规则已经落地。只有用户或任务合同明确授权实施时，才按第 7 节推进，并以第 8 节作为验收清单；普通代码生成不得把本文解释为全仓库整改授权。
