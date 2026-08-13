# AI 生成 C++/Qt/QML 代码注释规范

> 兼容入口：中文专题权威为 [`cn/CPP_Code_Comment_Guidelines.md`](cn/CPP_Code_Comment_Guidelines.md)；
> 英文镜像位于 [`en/CPP_Code_Comment_Guidelines.md`](en/CPP_Code_Comment_Guidelines.md)。

[English](en/CPP_Code_Comment_Guidelines.md) | [简体中文](cn/CPP_Code_Comment_Guidelines.md)

> 说明：本文档是 `cn/CPP_Code_Comment_Guidelines.md` 的根目录兼容入口。完整中文注释
> 规范以 `cn/` 对应文档为准。

## 1. 适用范围与权威关系

### 1.1 适用范围

本规范适用于 C++11 及以上项目，并为启用 Qt 6、QML、C++17 或 C++20 的项目提供
增量规则。它只规定以下内容：

- 注释应覆盖的契约、原因和非显然边界；
- public/protected、QML-facing、override、internal/private 和实现逻辑的注释强度；
- QDoc、Doxygen、KApiDox、Doxyqml 等文档工具链下的注释形式和验证边界；
- AI/agent 在当前任务范围内新增或修改注释时的执行规则。

本规范不要求为未触及的既有代码批量补注释，也不授权 AI/agent 新增文档工具链、修改
构建配置、扩大 API/ABI 承诺或同步整个文档包。

### 1.2 权威关系

1. [`cn/Qt6_CPP17_Coding_Style.md`](./cn/Qt6_CPP17_Coding_Style.md) 是 C++、Qt 生命周期、线程和
   基础格式总纲。
2. [`cn/CPP_Code_Comment_Guidelines.md`](./cn/CPP_Code_Comment_Guidelines.md) 负责注释内容、覆盖范围和生成文档规则；本文只同步呈现该专题。
3. [`Qt6_QML_Coding_Style.md`](./cn/Qt6_QML_Coding_Style.md) 只补充 QML 语境，并显式引用
   本文，不重复维护通用注释规则。
4. [`cn/Qt_Macro_Layout_Coding_Style.md`](./cn/Qt_Macro_Layout_Coding_Style.md) 负责 Qt、QML 和
   moc 宏的位置，不负责判断类型是否需要相应宏。
5. [`cn/Qt6_KDE_API_Parameter_Style.md`](./cn/Qt6_KDE_API_Parameter_Style.md) 负责 Public API
   参数、Borrow/Owning、view 生命周期和 QML/meta-object 边界类型。
6. [`cn/AI_CODING_BEHAVIOR.md`](./cn/AI_CODING_BEHAVIOR.md) 只说明 AI 如何应用上述权威，
   不得重新定义或扩大底层规则。

`Q_OBJECT` 是否必须存在，由总纲中的 Qt 技术条件或已发布项目门禁决定；本文只规定元对象、
QML 和生命周期等非显然契约何时需要注释。宏的存在本身不产生自动注释义务。

若总纲或专题存在事实错误，应在明确授权的文档修订任务中修正权威源；不得只在注释或
AI 行为文档中绕开、复制或固化错误。

## 2. 核心原则

### 2.1 注释目标

注释优先说明代码本身无法清楚表达的内容：

- 对调用方稳定的契约；
- 设计选择和约束来源；
- 失败、副作用、状态变化和外部可观测行为；
- 所有权、生命周期、线程亲和性、重入和异步时序；
- QML/C++、进程、平台、文件或安全边界；
- 为何不能采用看似更简单的实现。

注释不复述标识符、语法或逐行操作。能够通过准确命名、拆分函数或简化控制流消除的
歧义，应先改进代码；注释不能替代清晰设计。

### 2.2 触发规则

本文的“必须记录”均带有触发条件，不等于“所有对象必须具有同样的注释模板”：

- **新增或语义变化的导出 public/protected API 与 QML-facing 契约**：必须在当前
  profile 中形成可发现的接口说明，并记录调用方需要知道且代码签名无法表达的信息。
- **internal/private 和实现逻辑**：只在存在非显然不变量、原因、所有权、线程、时序或
  协议边界时添加注释。
- **自解释代码**：无需注释。
- **未触及的既有代码**：除非任务明确要求专项治理，否则不批量补注释。

参数、返回、失败、副作用、状态、所有权、生命周期、线程、重入、版本等字段只在真实
存在且适用时填写，禁止为了模板完整而机械补齐。

## 3. 文档工具链 profile

### 3.1 先识别工具链

开始编写生成文档注释前，应先读取项目已有的 `.qdocconf`、`Doxyfile`、CMake/ECM 配置、
KApiDox 入口或 Doxyqml 配置。以下各项是文档 profile，不全是相互独立的生成器：QDoc
和 Doxygen 是基础引擎；KApiDox/ECMAddQch 是 KDE 集成；Doxyqml 是 Doxygen 的 QML
扩展。

| Profile | 主要对象 | 常见位置与形式 | 适用边界 |
|---|---|---|---|
| Qt QDoc：C++ | Qt 风格 C++ API | 通常在 `.cpp` 中使用 QDoc 注释和 `\` 命令 | 已有 QDoc、Qt 原生文档或 QCH 路线时使用 |
| Qt QDoc：QML | QML 类型、属性、信号和方法 | QML 类型通常写在 `.qml`；C++ 类型通常写在 `.cpp` | Qt 原生 QML 语义或 QCH 需求明确时使用 |
| 普通 Doxygen | 通用 C/C++ 库 API | 导出 API 通常在 public header；实现说明留在 `.cpp` | 一般第三方 Qt6/C++ 默认路线 |
| KDE KApiDox | KDE/Qt 公共库 API | 基于 Doxygen，强调安装头文件中的完整公共契约 | 仅在 KDE/ECM 文档体系中作为默认 |
| Doxyqml | Doxygen 中的 QML API | 解析 QML 源码中的文档注释 | Doxygen 路线需要发布 QML API 时增加 |
| 无生成器 | 普通源码注释 | 使用 `//` 或项目既有块注释 | 不引入生成器专用标记和伪元数据 |

没有可确认的 profile 时，按“无生成器”处理。AI/agent 不得自行新增依赖、配置、构建
目标或 QCH 输出。

### 3.2 模板与启用边界

本节区分“可供选择的配置模板”和“当前项目已经启用的工具链”：模板只是带占位符的参考
文件或配置片段，不代表项目已经安装依赖、接入构建或能够生成文档。规范中提供模板时，
必须标明“模板、非启用配置”，不得用模板存在证明 QDoc、Doxygen、KApiDox、Doxyqml 或
QCH 已经可运行。

六类 profile 不对应六份互相独立的配置文件：

本仓库提供的非启用模板见 [`templates/documentation/README.md`](templates/documentation/README.md)。
这些模板用于审阅和项目选型，不代表本仓库已经安装或接入相应工具链。

| Profile | 默认配置形态 | 说明 |
|---|---|---|
| Qt QDoc：C++ + QML | 一个 `.qdocconf` | C++ 和 QML 通常由同一 QDoc 配置组织 |
| 普通 Doxygen | 一个 `Doxyfile` | 配置输入目录、输出格式、命令和警告策略 |
| KDE KApiDox | Doxygen 配置 + KDE/ECM/KApiDox 入口 | 不是一份通用的独立配置文件 |
| Doxyqml | Doxygen 配置 + QML 扩展接入 | 不能脱离基础 Doxygen 单独启用 |
| 无生成器 | 无配置文件 | 只维护普通源码注释 |

#### 3.2.1 默认选择矩阵

以下矩阵是面向一般第三方 Qt6/QML 开源项目的**人工选择建议**，不是“工具链已启用”声明。
已有配置优先；没有文档生成需求时仍使用“无生成器”。

| 项目事实或目标 | 推荐 profile | 选择说明 |
|---|---|---|
| 已有 QDoc 配置，或明确采用 Qt 原生文档/QCH | Qt QDoc：C++ + QML | 保留 Qt 文档语义和 QCH 生成链 |
| 已有 Doxygen 配置 | 普通 Doxygen | 保留现有注释和文档入口 |
| 已有 Doxygen，且发布 QML 公共 API | 普通 Doxygen + Doxyqml | Doxyqml 只作为 QML 扩展加入 |
| 已有 Doxygen 风格注释但没有文档配置 | 普通 Doxygen 注释形式；工具链仍按无生成器处理 | 延续现有语法，不自动新增配置或依赖 |
| 一般第三方 Qt6/QML，已有 Doxygen 风格注释，需发布 C++ API | 普通 Doxygen | 无既有生成器时的默认 C++ 路线 |
| 上述项目还需发布 QML 公共 API | 普通 Doxygen + Doxyqml | 无 QML 公共 API 时不额外引入 Doxyqml |
| KDE/ECM 公共库 | KDE KApiDox；有 QML API 时加 Doxyqml | 使用 KDE 公共 API 和构建集成 |
| 不发布开发者 API 文档 | 无生成器 | 只维护源码注释，不生成文档配置 |

`普通 Doxygen` 是一般第三方 Qt6/C++ 公共 API 的默认路线；项目同时发布 QML 公共 API
时，在同一 Doxygen 路线上增加 `Doxyqml`，形成 `普通 Doxygen + Doxyqml`。这只是选择
建议，不覆盖已有 QDoc、KDE/ECM 决策，也不授权 AI/agent 自动新增工具链。需要 Qt 原生
QML 文档语义或 QCH 时，应明确选择 QDoc C++/QML 路线。

模板至少应把项目名称、版本、源码目录、public include 目录、QML module/import 路径、
输出目录和 QCH 元数据写成明确占位符，例如 `@PROJECT_NAME@`、`@VERSION@`、
`@SOURCE_DIRS@`、`@PUBLIC_INCLUDE_DIR@`、`@QML_IMPORT_PATH@` 和 `@OUTPUT_DIR@`。

只有满足以下条件，才能把 profile 视为当前项目已启用：

1. 项目存在可确认的 `.qdocconf`、`Doxyfile`、KApiDox/ECM 入口或 Doxyqml 集成配置。
2. 配置已经接入文档命令、构建目标或明确的手工生成入口，并使用了真实项目路径和版本。
3. 所需工具和依赖已由项目声明或安装，且至少完成一次与任务范围匹配的生成验证。

QCH 是选定 QDoc 或 Doxygen/KDE 工具链的输出或构建集成，不是独立 profile。需要 QCH 时，
模板可以包含 QHP/QCH 或 `ECMAddQch` 片段，但不能因此自动切换基础引擎。

AI/agent 应遵循以下边界：已有项目配置优先；用户明确选择 profile 后才能按模板补齐项目
参数；没有配置且用户未明确要求引入工具链时，继续按“无生成器”处理。模板本身不授权新增
依赖、构建目标、QCH 输出或零警告门禁。

### 3.3 注释形式和位置由 profile 决定

- Doxygen 合法支持 `///`、`/** ... */`、`/*! ... */` 以及 `@`、`\` 命令。项目应在
  同一 profile 内选择并保持主风格一致，不能把其他合法形式描述为解析错误。
- QDoc 通常使用 `/*! ... */` 和 `\` 命令。`\class`、`\qmltype` 等结构命令应按 QDoc
  配置和对象类型使用。
- 公共文档放在 header 还是 `.cpp`，由选定 profile 决定；不得制定跨 QDoc、Doxygen
  和 KApiDox 的唯一位置规则。
- 普通实现注释应紧邻受约束的代码。跨文件重复同一说明会增加失真风险，应保留一个
  权威位置并使用链接或生成器支持的复用机制。

### 3.4 QCH 不是新的注释语法

需要 Qt Assistant 的 `.qch` 文档时，应沿用项目已选定的基础引擎：QDoc 项目使用其
QHP/QCH 生成链；Doxygen/KDE 项目使用已有的 QHP、QHelpGenerator 或 ECMAddQch 集成。
新增 `.qch` 输出不应迫使项目改写注释 profile，也不授权 AI/agent 自动切换工具链。

## 4. 注释基础规范

1. 实现说明优先使用 `//`；生成文档注释使用当前 profile 的既有形式。
2. 注释语言应简洁、准确，与项目既有语言一致；不要混用不同术语表达同一概念。
3. 遵循项目 `ColumnLimit: 100` 的软基线，按语义分段。`ReflowComments: false` 只表示
   clang-format 不自动重排注释，不代表允许忽略可读性。
4. 注释应靠近其约束对象，并与代码在同一变更中同步更新或删除。
5. 公共文档描述调用方可观察的行为，不泄露可替换的实现细节。
6. internal/private 注释优先解释“为什么”和“不变量”，不把内部细节包装成稳定承诺。
7. 示例代码同样必须符合项目 clang-format 基线，不得含未使用变量或与示例目标无关的
   流水账注释。

## 5. 分层注释义务

以下矩阵只适用于当前任务新增或语义发生变化的对象：

| 对象 | 默认注释义务 |
|---|---|
| 导出的 public/protected API | 记录稳定契约，以及真实存在且非显然的调用约束 |
| QML-facing 类型、方法、属性和信号 | 记录 QML 可见行为，以及适用的默认值、失败语义、handler 和参数 |
| override | 默认继承基类文档；只记录相对基类的行为或约束差异 |
| internal/private API | 只记录非显然不变量、生命周期、线程、时序和协议边界 |
| 实现逻辑 | 解释设计原因和约束来源，不复述代码步骤 |
| TODO/FIXME | 记录真实的可追踪上下文；按任务性质补充日期、事件或移除条件 |

## 6. 注释触发条件与示例

本章保留常见高价值场景，但每一项都以“存在非显然语义”为触发条件。示例展示信息类型，
不规定所有项目必须使用同一种注释标记。

### 6.1 通用 C++ 场景

#### 6.1.1 类、结构体和枚举

当类型的职责、生命周期、稳定语义或与其他类型的关系无法由名称表达时，应说明其用途。
公开枚举应说明整体用途和非显然枚举值；internal/private 的自解释枚举无需逐值注释。

```cpp
/// 表示一次可取消的索引任务；析构对象不会等待后台任务完成。
class IndexJob;

/// 选择缓存未命中后的恢复策略。
enum class RecoveryPolicy {
    UseStaleData, ///< 返回过期数据，并在后台刷新。
    FailRequest,  ///< 不返回缓存内容，直接报告失败。
};
```

无需为只承担显然数据聚合职责、且成员名称已经完整表达语义的 private 结构体补充类型说明。

#### 6.1.2 非直观成员变量

当单位、取值域、时间基准、所有权、失效条件或跨调用含义不明显时，应记录相应约束。

```cpp
// 单调时钟时间戳；只用于计算退避时间，不可持久化。
std::chrono::steady_clock::time_point m_nextRetryAt;
```

`int count`、`bool isReady` 等已经自解释的状态不需要同义注释。

#### 6.1.3 函数契约

对新增或语义变化的公共函数，记录调用方必须知道且签名无法表达的内容。只写适用字段，
不要机械生成“参数说明”“返回值说明”等空话。

```cpp
/// 提交任务，并在同一任务已排队时合并请求。
/// @param task 要入队的任务；成功时其内容由队列持有。
/// @param timeout 等待入队的最长时间；零表示不等待。
/// @return 任务已入队或已与现有任务合并时返回 true。
/// @note 本函数会修改队列状态，且不可从完成回调中重入。
bool enqueue(Task task, std::chrono::milliseconds timeout);
```

简单 getter/setter、默认构造/析构和名称已完整表达行为的内部函数无需例行注释。

#### 6.1.4 复杂逻辑和算法选择

只有算法意图、分支原因、不变量或性能取舍不直观时才注释。注释应解释选择原因，而不是
翻译条件表达式。

```cpp
// 先按稳定键排序，确保增量刷新不会改变同优先级项目的可见顺序。
std::stable_sort(items.begin(), items.end(), comparePriority);
```

#### 6.1.5 宏、常量和协议值

当数值来自协议、系统限制、兼容要求或测量结论时，应说明来源和约束；名称已经表达的
纯数学常量无需注释。

```cpp
// 协议保留 0 作为“未分配”，因此有效序号从 1 开始。
constexpr std::uint32_t kFirstSequenceNumber = 1;
```

#### 6.1.6 错误、所有权、线程和时序

当失败条件、资源所有权、线程安全、重入、锁顺序或延迟执行要求非显然时，应在最接近
约束的位置记录。

```cpp
// 调用方持有 m_registryMutex；这里不能发出可能同步回调注册表的信号。
removeEntryLocked(id);
```

### 6.2 Qt/QML 与 Qt/KDE library 场景

#### 6.2.1 导出的 Qt 公共 API

对导出的 public/protected 类型、方法、factory、parser/loader 以及 start/stop/request/
dispatch 类接口，记录稳定契约和适用的失败、副作用、状态、线程或重入约束。是否承诺
API/ABI 兼容由项目发布政策决定，不能从“公共”二字自动推导。

```cpp
/// 启动一次异步加载；已有请求会被取消并以 Canceled 结束。
/// @param source 要加载的绝对 URL；空 URL 或不支持的 scheme 无效。
/// @return 请求标识；参数无效时返回无效标识，且不会改变当前状态。
RequestId startLoad(const QUrl &source);
```

#### 6.2.2 QML-facing 类型、方法、属性和信号

对 `Q_INVOKABLE`、public slot、QML singleton、attached property provider、QML-facing
model 等接口，说明 QML 可见的用途、失败语义、副作用和调用约束。

QML 属性只在适用时说明行为、默认值、有效范围、只读性、副作用或失败语义；不得对每个
`Q_PROPERTY` 机械堆注释。宏位置仍由 `Qt_Macro_Layout_Coding_Style.md` 决定。

QML 信号应在触发条件、对应 handler 或参数语义不明显时说明这些信息。例如，QDoc
profile 可以写成：

```qml
/*!
    \qmlsignal DownloadView::retryRequested(int attempt)

    当前传输可重试时触发。\a attempt 从 1 开始；对应的处理器为
    \c onRetryRequested。
*/
signal retryRequested(int attempt)
```

简单内部信号、名称已完整表达语义的 notifier，以及无额外行为的属性无需为生成文档
机械补齐字段。

#### 6.2.3 QObject 生命周期与所有权

当 `QObject`、`QQuickItem`、裸指针、`QPointer`、event filter、lambda 捕获或
`deleteLater()` 涉及非显然所有权时，说明谁持有对象、何时失效、在哪个线程销毁以及
是否依赖事件循环。

```cpp
// worker 没有 parent，因为它会移入工作线程；finished 在其所属线程安排延迟销毁。
connect(worker, &Worker::finished, worker, &QObject::deleteLater);
```

不要把注释写成“所有 QObject 都禁止直接删除”或“所有异步对象都必须调用
`deleteLater()`”。具体生命周期必须遵循总纲，并由可证明的 parent、owner、线程亲和性、
事件处理状态和事件循环条件决定。

对 shared-owned QObject，还应记录自定义 `deleteLater()` deleter、最后一个 owner 的释放时机、
目标事件循环停止顺序，以及 `destroyed()` 或等价的实际析构证据。

#### 6.2.4 QML/C++ 数据边界

对 `QVariantMap`、`QVariantList`、model role、role name、QML 使用的字符串字段、restore
key 或 trace payload，记录稳定字段、无效输入、空值语义和返回约定。

```cpp
// QML 将空 sourceUrl 解释为“清除当前预览”；缺少该键表示保持现状。
constexpr auto kSourceUrlKey = "sourceUrl"_L1;
```

#### 6.2.5 Model 和 role 契约

对 `QAbstractItemModel`、model row 与业务对象的关系、role name、stable key、restore
path、空 model 和无效 index 的返回语义，只记录 QML、下游或测试依赖的稳定边界。

```cpp
/// 返回条目的稳定标识；排序和过滤不会改变该值。
/// @param index 当前模型中的条目索引。
/// @return 条目的稳定标识；index 无效时返回空字符串。
QString itemId(const QModelIndex &index) const;
```

#### 6.2.6 事件循环、异步与重入

对 queued connection、deferred operation、transaction queue、debounce/throttle、timer
生命周期、reentrancy guard、信号顺序和必须等待下一轮事件循环的逻辑，说明时序原因和
防护意图，不解释 Qt API 的字面语法。

```cpp
// 延迟到下一轮事件循环，避免在 rowsRemoved 通知期间重入 model reset。
QMetaObject::invokeMethod(this, [this] { resetPendingRows(); }, Qt::QueuedConnection);
```

#### 6.2.7 诊断、错误码和外部可观测契约

deny code、trace event kind、debug payload、日志类别或 contract test 依赖的稳定字符串
只有在确为外部可观测协议时才应记录依赖方和兼容边界。

```cpp
// QML 错误页和遥测聚合均使用该值；修改它属于协议变更。
constexpr auto kAccessDeniedCode = "access-denied"_L1;
```

#### 6.2.8 Public/internal/private 边界

public header 中记录下游可见契约；private header 和 `.cpp` 只记录实现不变量。不要把
KDE Frameworks 的稳定性承诺自动套用到没有相同发布政策的一般 Qt 库。

#### 6.2.9 平台、安全和文件访问边界

对 debug-only 行为、`file://` 读取、sandbox/portal 权限、Wayland/X11/Linux-only 分支、
平台 fallback 或拒绝策略，记录限制原因、失败行为和安全边界。

```cpp
// 沙箱内不直接访问宿主路径；缺少 portal 授权时返回 PermissionDenied。
return openThroughPortal(url);
```

### 6.3 C++17 专属场景

以下规则只在项目启用 C++17 或更高版本，且特性承载非显然领域语义时触发。

#### 6.3.1 结构化绑定

绑定来源或各元素含义不明显时，说明领域含义；不要注释“这里使用结构化绑定”。

```cpp
// 结果依次为持久设备标识和 UI 显示的信号强度百分比。
const auto &[deviceId, signalStrength] = probeResult;
```

#### 6.3.2 `if constexpr`

当编译期分支承担特殊策略时，说明区分原因，不复述类型条件。

```cpp
// 整数时间戳来自旧协议，需要在编译期选择无损转换路径。
if constexpr (std::is_integral_v<T>) {
    return fromLegacyTimestamp(value);
}
```

#### 6.3.3 `std::optional` 和 `std::variant`

当空值或分支具有非显然业务语义时，说明其含义和默认行为。

```cpp
// 空值表示尚未探测，而不是设备不存在。
std::optional<DeviceInfo> m_deviceInfo;

// Error 分支保留可重试信息；调用方不得将它折叠为空结果。
using QueryResult = std::variant<Data, Error>;
```

#### 6.3.4 折叠表达式

复杂聚合规则应说明整体目标；普通求和、全真判断等自解释表达式无需注释。

```cpp
// 任一权限明确拒绝即终止；Unknown 不参与拒绝判断。
return (... || isDenied(permissions));
```

#### 6.3.5 内联变量

仅在初始化顺序、共享状态或链接边界不明显时说明。内联变量表示跨翻译单元的同一实体，
但不自动等同于单例模式。

```cpp
// 头文件定义的共享默认值；所有包含该头的翻译单元引用同一实体。
inline const QString kDefaultProfile = QStringLiteral("standard");
```

### 6.4 C++20 专属场景

以下规则只在项目启用 C++20 或更高版本，且特性包含非显然约束时触发。

#### 6.4.1 Concepts

概念名称和约束表达式不能说明领域目的时，补充设计意图；不要逐条翻译 requires 表达式。

```cpp
// 限制为可稳定序列化的消息类型，避免异步队列持有借用视图。
template<SerializableMessage T>
void enqueue(T message);
```

#### 6.4.2 Coroutines

记录挂起条件、恢复线程、取消、产出值和资源生命周期中非显然的部分。

```cpp
// 恢复发生在会话线程；取消后不再发出 progressChanged。
co_await session.waitForReady();
```

#### 6.4.3 Ranges

复杂 pipeline 应说明整体数据目标或顺序要求，不逐个解释 `filter`、`transform` 等操作。

```cpp
// 保留可见条目并提取稳定标识，顺序必须与当前 model 一致。
auto visibleIds = items | std::views::filter(isVisible) | std::views::transform(itemId);
```

#### 6.4.4 Modules

只在导出边界、依赖方向、初始化或兼容约束非显然时注释。普通 `import`/`export` 不需要
“导入模块”“导出函数”式说明。

```cpp
// 只导出值类型合同；Qt private 适配器留在模块实现单元。
export module library.api;
```

#### 6.4.5 编译期预计算和容器

C++20 使 `std::vector` 的许多操作可用于常量求值，但非空 `std::vector` 对象通常不能
作为一般的持久常量表达式。固定编译期数据通常优先使用 `std::array`。只有预计算原因
或成本取舍不明显时才注释。

```cpp
// 编译期生成固定查找表，避免在每次解析时重复计算校验值。
constexpr std::array kChecksumTable = makeChecksumTable();
```

#### 6.4.6 三路比较运算符

默认逐成员比较无需注释；只有排序字段、大小写、浮点、无效值等特殊规则存在时才说明。

```cpp
// 版本先按主版本号排序；相同主版本下，预发布版本排在正式版本之前。
std::strong_ordering operator<=>(const Version &other) const;
```

## 7. Doxygen、QDoc 与元数据规则

### 7.1 参数、模板参数和返回值

- 在启用生成文档的 profile 中，新增或语义变化的导出 API 应具有可发现的接口说明；没有
  额外限制时可以保持简短，但不能用空标签代替用途说明。
- 在 Doxygen profile 且项目启用完整性检查时，已文档化的公共函数应覆盖真实存在的
  `@param`、`@tparam` 和适用的返回语义。
- 不要生成“输入参数”“返回结果”等重复签名的空描述。
- 参数方向标记、命令前缀和段落顺序遵循现有 profile；不要在同一文档块中随意混用
  `@` 与 `\` 风格。
- `void`、显然 getter 或无额外语义的返回值不需要为了字段齐全而添加空 `@return`。

### 7.2 版本和弃用

- `@since` 或 `\since` 只用于项目具有版本化约定、且真实发布版本已经确定的稳定 API。
- `@deprecated` 或 `\deprecated` 应包含真实版本、弃用原因和可执行的替代路径。
- AI/agent 不得猜测版本号、发布日期、替代 API、bug ID 或 owner。

### 7.3 Override 和文档继承

- override 默认继承基类契约；没有语义差异时不复制基类全文。
- Doxygen profile 可按项目惯例使用 `@copydoc`；QDoc profile 可对无差异的重实现使用
  `\reimp`。
- 行为、前后置条件、失败、副作用或线程语义发生变化时，应显式记录差异，不能只使用
  继承命令掩盖变化。

### 7.4 结构命令

- 不得全局禁止 `\class`、`\fn` 等结构命令。
- QDoc 类文档可按需要使用 `\class`；`\fn` 只在注释无法紧邻被记录的实现时使用。
- Doxygen 的结构命令只在配置或非邻接文档确有需要时使用；能由源码关联推导时不重复
  声明实体。

### 7.5 TODO/FIXME

TODO/FIXME 必须包含真实的 bug ID、owner 或其他可追踪上下文之一。只有未来日期或事件
驱动的清理任务，才要求具体日期、事件或移除条件。格式形态为
`TODO(<真实工单号>): <后续动作和触发条件>`；尖括号内容不能作为源码占位文本提交。

缺少真实追踪信息时，不得由 AI/agent 编造占位 ID 或人员名称。应解决问题、报告事实缺口，
或按任务约定保持现状。

## 8. 禁止和无需添加注释的场景

1. 简单操作和显然控制流，例如 `i++`、`if (isReady)`。
2. 名称已完整表达含义的变量、getter/setter、默认构造/析构和薄转发函数。
3. 复述代码字面行为的注释，例如“给 `count` 加一”或“返回结果”。
4. 只说明 C++17/20、Qt 或 QML 语法名称，不说明领域语义或约束的注释。
5. 对每行代码、每个 `Q_PROPERTY`、简单 signal、显然 role 或 enum value 机械补注释。
6. 为满足模板而生成不适用的参数、返回、失败、线程、版本或所有权字段。
7. 将聊天历史、临时实现过程、已失效设计或可从版本控制获得的信息写入源码。
8. 与代码不一致、重复多个权威位置、无法随变更维护的说明。
9. 将 Qt/KDE 项目的 API/ABI 承诺、线程模型或生命周期政策自动写成一般 Qt 库的默认事实。
10. 在未识别 profile 时混入生成器专用命令，或声称已生成文档、QCH、tag file 或零警告。
11. 编造版本、日期、bug ID、owner、默认值、线程模型、失败语义或替代 API。
12. 因发现注释缺口而扩大任务范围，批量改写未触及文件或自行修改权威规范。

## 9. AI/agent 执行流程

### 9.1 范围先行

只处理用户点名或任务合同允许的文件、接口和语义变化。发现范围外问题时记录事实，不自动
扩展为全仓库注释治理、格式化或文档发布任务。

### 9.2 识别 profile 和接口层级

1. 查找现有 QDoc、Doxygen、KApiDox、Doxyqml、ECM 和 QCH 配置。
2. 无配置时使用普通源码注释，不新增工具链。
3. 将对象归类为 public/protected、QML-facing、override、internal/private 或实现逻辑。
4. 只应用该层级和当前 profile 的规则。

### 9.3 识别语义增量

比较修改前后的调用方可见行为，确定是否新增或改变以下内容：失败、副作用、状态、
所有权、生命周期、线程、重入、时序、平台、安全或稳定数据字段。没有非显然增量时，
不为了“看起来完整”添加注释。

### 9.4 写最小充分注释

- 先写最重要的契约或原因，再补真实且适用的限制。
- 能通过命名或结构表达的内容留在代码中。
- 缺少事实时不猜测；保留事实缺口并在交付说明中报告。
- 实现任务不得自行修改权威文档。只有明确的文档修订任务才能更新 `cn/` 中的注释专题或相关总纲，再同步本文。

### 9.5 验证并停止

只运行与当前范围及现有工具链匹配的格式、链接、文档生成、构建或测试验证。达到任务
验收条件后停止；不要借注释修改同步根目录、英文版、版本号、日期或无关示例，除非任务
明确要求进行文档包发布。

## 10. 验证清单

提交注释相关变更前，按当前任务范围检查：

- [ ] 新增或变化的 public/protected 和 QML-facing 契约已记录适用的非显然信息。
- [ ] override 没有复制基类全文，行为差异已单独说明。
- [ ] internal/private 注释只解释不变量、原因、生命周期、线程、时序或协议边界。
- [ ] 没有机械补齐字段、逐行解释、过期说明或编造元数据。
- [ ] 注释形式、命令和位置符合项目已选定的 profile。
- [ ] 文本符合 100 列软基线，示例符合项目 clang-format 基线。
- [ ] 相对链接有效，注释与相关代码保持同步。
- [ ] 只有在配置存在且任务要求时，才运行并通过 QDoc、Doxygen、KApiDox、Doxyqml 或
      QCH 生成验证。
- [ ] 未配置生成工具链时，完成声明明确写明“未配置”，不声称生成文档零警告。
- [ ] AI/agent 没有扩大文件范围、自动修改权威或同步未授权的分发版本。

---

**文档包版本**：v1.2.0

**最后更新**：2026-08-13
