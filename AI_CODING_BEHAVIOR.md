# AI 编码行为与执行边界（中文）

[English](en/AI_CODING_BEHAVIOR.md) | [简体中文](cn/AI_CODING_BEHAVIOR.md)

> 说明：本文档是当前 AI 编码行为规范的中文权威版本，用于约束 AI/agent 的代码生成与
> 修改。若与其他分发版存在差异，以本文档和项目其他权威规范为准。

> 本文档只解释 AI/agent 如何应用现有项目权威，不重新定义或扩大底层规则。

> **项目政策前置条件**：本文把“所有 `QObject` 子类必须添加 `Q_OBJECT`”作为项目
> 强制门禁使用。该门禁必须已经由项目权威或明确决策发布；它不是 Qt 普遍规则。

---

## 1. 文档职责与权威关系

本文档的职责是约束 AI/agent 的决策顺序、范围、事实来源和验证行为。它不授权 AI/agent
自行修改规范、构建配置、文档工具链、版本信息或任务范围外的代码。

项目权威关系如下：

1. [`Qt6_CPP17_Coding_Style.md`](./Qt6_CPP17_Coding_Style.md) 是 C++、Qt 生命周期、线程和
   基础格式总纲。
2. [`Qt6_QML_Coding_Style.md`](./Qt6_QML_Coding_Style.md) 只维护 QML 专题增量，并引用
   注释规范，不重复维护通用规则。
3. [`Qt_Macro_Layout_Coding_Style.md`](./Qt_Macro_Layout_Coding_Style.md) 负责 Qt、QML 和
   moc 宏的位置、顺序与类体布局，不决定项目是否建立 `Q_OBJECT` 门禁。
4. [`Qt6_KDE_API_Parameter_Style.md`](./Qt6_KDE_API_Parameter_Style.md) 负责 Public API
   参数、Borrow/Owning、view 生命周期和 QML/meta-object 边界类型。
5. [`CPP_Code_Comment_Guidelines.md`](./CPP_Code_Comment_Guidelines.md) 负责注释内容、
   覆盖范围、生成器 profile 和文档验证规则。
6. 本文只说明 AI 如何应用上述权威；权威之间发生事实冲突时，应报告冲突，并在获得
   文档修订授权后修正权威源。

## 2. 规范分类与执行优先级

### 2.1 强制规范

强制规范只包括以下两类：

1. Qt/C++ 技术硬要求、目标语言标准、构建约束以及 public API/ABI 合同。
2. 项目权威明确标记为“必须”“禁止”或可执行门禁的项目政策。

用户要求生成违反技术硬要求或已发布项目门禁的代码时，AI/agent 应说明冲突并提供
符合约束的替代实现。只有任务明确授权修改该门禁时，才先修订权威，再同步代码；
不得保留原规则并暗中生成例外。

### 2.2 `Q_OBJECT` 项目强制门禁

若项目权威明确规定“所有 `QObject` 子类必须添加 `Q_OBJECT`”，AI/agent 必须按以下
规则执行：

1. 该要求是**项目特定的强制门禁**，不能写成 Qt 或 KDE 的普遍技术事实。
2. “所有”默认覆盖项目内 public、protected、private 和 internal 的 `QObject` 子类，
   不得自行缩窄为导出类、QML-facing 类或具体类。
3. 新增或修改 `QObject` 子类时，必须同步添加 `Q_OBJECT`，并确认 moc、AUTOMOC、qmake/
   CMake 和最终链接链路都能处理该类型。
4. 如果任务本身是落实门禁的全树治理任务，必须审计任务范围内的全部 `QObject` 子类。
   普通局部任务不得无授权扩展为全仓库重构，但应报告范围内已存在的缺失项。
5. 模板类、嵌套类、header-only 类型或其他 moc/build 无法处理的类型不得静默跳过。
   项目必须记录正式技术例外、理由、责任人和移除条件，或先调整类型/构建设计使门禁
   可执行。没有正式例外时，AI/agent 不得自行豁免。
6. `Q_OBJECT` 的位置、宏顺序和类体布局仍由 `Qt_Macro_Layout_Coding_Style.md` 决定；
   本节只决定“是否必须存在”。

如果项目尚未正式发布该门禁，则恢复普通分层：类型自身使用 signal、slot、property、
enum、QML/插件元数据或其他自身元对象能力时，`Q_OBJECT` 属于 Qt 技术硬要求；其他
普通 `QObject` 子类添加 `Q_OBJECT` 属于 Qt 上游强烈建议，不自动触发全树修改。

### 2.3 `QObject` 生命周期

`QObject` 不得按值复制或放入要求复制语义的容器。销毁策略不能写成“所有对象禁止直接
`delete`”：

- 跨线程直接删除，以及对象正在处理其所接收事件时同步删除，必须阻止。
- 动态子对象与 owner 同生命周期时，默认优先使用 parent ownership。
- 同线程、函数局部、不逃逸、无 parent、未处理事件且无在途使用时，直接析构或受控
  RAII 可以成立。
- 只有事件处理、跨线程销毁请求或明确的事件驱动 worker 生命周期需要延迟销毁时，才
  使用 `deleteLater()`。
- `deleteLater()` 依赖对象所属线程能够处理 deferred delete；queued connection 或异步
  回调本身不自动证明必须使用它。
- 无法在当前代码范围证明 owner、线程和事件循环条件时，保留既有生命周期并报告，
  不主动批量改写。

### 2.4 可选推荐（含 Qt/KDE 上游强烈建议）

上游强烈建议和现代 C++ 写法默认用于新代码，但除非项目权威明确升级为门禁，否则不
视为违规：

- 新代码只有在目标标准、public ABI/API、QML/meta-object 边界和既有合同允许时采用。
- 维护代码优先保持已有接口和局部风格，不因推荐项扩大重构范围。
- 用户明确选择合法写法时直接执行，不重复劝告。
- `std::span` 是 C++20 的非 owning 连续内存 view，不是容器；必须确认借用生命周期。
- `std::variant` 是带标签联合类型，不是容器。
- C++20 使 `std::vector` 的许多操作可参与常量求值，但非空 vector 对象通常不能作为
  一般的持久常量表达式；固定编译期数据通常优先使用 `std::array`。
- `constexpr`、结构化绑定、concepts、ranges、coroutines、modules 和 `<=>` 只在承载
  非显然语义时解释，不因使用新语法机械添加说明。

### 2.5 项目特定约定

- 既有代码只能作为识别局部风格的证据；代码频率不能覆盖现行规范，也不能自动形成
  新项目政策。不得依据统计比例建立规则。
- 新约定只有在项目明确决策或用户授权的文档修订任务中，才能写入权威文档。
- 实现任务发现总纲和专题冲突时，遵循当前最高权威并报告冲突，不得自行改写权威。

### 2.6 决策优先级

在任务范围内，按以下顺序处理冲突：

1. 用户明确的目标、文件范围和验证范围；
2. Qt/C++ 技术硬要求、目标标准和 public API/ABI 合同；
3. 已发布的项目强制门禁，包括“所有 `QObject` 子类必须 `Q_OBJECT`”；
4. 对应专题规范和文档工具链 profile；
5. 现有局部风格；
6. Qt/KDE 上游强烈建议和可选推荐。

用户选择不能覆盖技术硬要求或已发布门禁；用户明确授权修改门禁时，应先修改权威并
同步执行，不能在实现阶段隐式豁免。

## 3. AI/agent 执行流程

1. **范围先行**：确认用户目标、允许修改的文件、代码路径和验证范围。
2. **确认技术边界**：读取 C++ 标准、Qt 版本、构建配置、public API/ABI 和项目权威。
3. **判断 `Q_OBJECT`**：门禁启用时，任务范围内所有适用的 `QObject` 子类都必须添加
   宏；门禁未启用时，只按元对象技术需求或上游建议处理。
4. **确认 moc/build**：添加宏不能只停留在源码文本；必须确认 moc/AUTOMOC、生成源文件、
   编译和链接链路都覆盖该类型。
5. **确认生命周期**：先确认 parent、owner、线程亲和性、事件状态、在途使用和事件循环，
   再选择 parent ownership、RAII、直接析构或 `deleteLater()`。
6. **识别文档 profile**：读取 `.qdocconf`、`Doxyfile`、KApiDox、Doxyqml、ECM 或项目
   入口；没有可确认配置时按无生成器处理，不新增工具链。
7. **限制注释增量**：遵循 `CPP_Code_Comment_Guidelines.md`，只为新增或语义变化的
   public/protected/QML-facing 契约和非显然内部约束补注释。
8. **事实不可臆造**：版本、日期、bug ID、owner、替代 API、默认值、线程模型和技术例外
   必须有项目事实来源；`@since`、`@deprecated` 和 TODO 元数据同样不得猜测或编造。
9. **验证并停止**：只运行与修改范围匹配的格式、构建、moc、测试或文档验证；未配置
   工具链时报告“未配置”，不得声称文档零警告。

QCH 必须沿用已确认的 QDoc 或 Doxygen 基础引擎；只有项目已依赖 ECM 时才使用
ECMAddQch。不得仅为 `.qch` 自动切换基础引擎或引入完整 KApiDox。

## 4. 实际场景示例

### 场景 1：新函数选择现代返回类型

目标标准支持 C++17，且空值确实表达“没有有效颜色”时，可以采用 `std::optional`；这
不是无条件的“新代码必须现代化”。

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

示例只保留参与返回逻辑的变量，不添加重复代码语义的注释。

### 场景 2：维护既有 API

维护 `bool + 输出参数` 合同时，不因可选推荐改成 `std::optional`；只修改任务要求的
验证逻辑和当前触及的格式。

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

### 场景 3：用户明确选择合法传统写法

用户要求 `QPair` 时，先确认它不违反目标标准、public ABI 或 meta-object 合同；确认合法后
直接执行，不机械建议改用结构化绑定。所有函数示例遵守项目 clang-format 基线。

### 场景 4：项目门禁下的 `Q_OBJECT`

项目门禁启用时，即使类型没有自身 signal、slot 或 property，项目范围内的 `QObject` 子类
也必须添加 `Q_OBJECT`：

```cpp
class InternalWorker : public QObject
{
    Q_OBJECT

public:
    explicit InternalWorker(QObject *parent = nullptr);
};
```

AI/agent 必须确认该类型属于门禁覆盖范围，并验证 moc/AUTOMOC、编译和链接链路。技术上
无法由 moc/build 处理的类型必须引用项目正式例外；没有正式例外时不得静默省略。

### 场景 5：QObject 生命周期

“所有 `QObject` 都禁止直接 `delete`”仍然不是正确表述。动态子对象优先 parent ownership；
事件处理、跨线程销毁请求或明确异步 worker 生命周期时使用 `deleteLater()`，并确认所属
线程仍有可处理 deferred delete 的事件循环。

```cpp
QWidget *createWidget(QWidget *parent)
{
    auto *widget = new QWidget(parent);
    widget->setObjectName(QStringLiteral("editorPane"));
    return widget;
}

void bindWorkerLifetime(Worker *worker)
{
    // 前提：worker 所在线程仍能处理 deferred delete。
    QObject::connect(worker, &Worker::finished, worker, &QObject::deleteLater);
}
```

跨线程直接删除或在对象处理所接收事件时同步删除必须阻止；其他场景按可证明的 owner、
线程和事件状态决定，不批量改写既有生命周期。

## 5. AI/agent 应该做与不应该做的事

### 应该做

1. 先锁定范围，再执行技术硬要求、项目门禁和专题规范。
2. 门禁启用时，对任务范围内所有适用 `QObject` 子类添加 `Q_OBJECT`，并验证 moc/build。
3. 区分 Qt 技术硬要求、Qt/KDE 上游强烈建议和项目强制政策。
4. 保护 public API/ABI、QML/meta-object 合同和已验证生命周期。
5. 只使用真实版本、追踪信息、线程模型和文档 profile。
6. 使用真实执行过的验证结果完成交付。

### 不应该做

1. 依据代码占比自动建立规范。
2. 未经项目授权把建议升级为门禁，或在门禁未启用时全树补 `Q_OBJECT`。
3. 门禁已启用时把 private/internal 类自行排除。
4. 把所有直接销毁视为违规，或把所有异步代码机械改成 `deleteLater()`。
5. 编造版本、日期、bug ID、owner、替代 API 或技术例外。
6. 自动修改权威文档、引入生成器、同步分发版或扩大重构范围。

## 6. 相关文档、验证与版本联动

相关权威文档：

- [`Qt6_CPP17_Coding_Style.md`](./Qt6_CPP17_Coding_Style.md)
- [`Qt6_QML_Coding_Style.md`](./Qt6_QML_Coding_Style.md)
- [`Qt_Macro_Layout_Coding_Style.md`](./Qt_Macro_Layout_Coding_Style.md)
- [`Qt6_KDE_API_Parameter_Style.md`](./Qt6_KDE_API_Parameter_Style.md)
- [`CPP_Code_Comment_Guidelines.md`](./CPP_Code_Comment_Guidelines.md)
- [`Qt6_CPP17_CLANG-FORMAT`](./Qt6_CPP17_CLANG-FORMAT)

提交前按任务范围检查：

- 门禁启用时，范围内的 `QObject` 子类全部具有 `Q_OBJECT`，或引用正式技术例外。
- moc/AUTOMOC、编译、链接和运行期元对象检查通过，不能只检查源码文本。
- 注释和示例遵循 `ColumnLimit: 100` 的软基线、`ReflowComments: false`，以及
  `BraceWrapping.AfterFunction: true`。
- 所有被修改的函数示例符合项目 clang-format 基线。
- 文档 profile 存在时才执行 QDoc、Doxygen、KApiDox、Doxyqml 或 QCH 验证。
- 未配置工具链时明确报告“未配置”，不得声称零警告。
- 只有明确的文档发布或同步任务才联动根目录、`cn/`、`en/`、版本、日期和链接。

**文档包版本**：v1.1.0
**最后更新**：2026-07-25
