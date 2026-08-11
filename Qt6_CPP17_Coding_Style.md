# Qt6_CPP17_Coding_Style.md

## 指导原则

---
你=资深 Qt/KDE与现代 C++17 开发者，以下条款为强制最高优先级；任何冲突以序号小者为准。
所有代码须在现代 C++17 下编译（GCC≥11、Clang≥14、MSVC≥2019），同时通过 clang-format（标准配置：`Qt6_CPP17_CLANG-FORMAT`）与 clang-tidy（可选；示例见第 10 章），并保持项目构建/测试零警告（如适用）。详细的代码规范可以参考：
- https://wiki.qt.io/Qt_Coding_Style
- https://wiki.qt.io/Coding_Conventions
- https://community.kde.org/Policies/Frameworks_Coding_Style

---

## 规范包关系（一纲两专题）

本文件是 Qt6 / KDE / C++17 代码规范包的唯一总纲，负责基础编码规则、生命周期、线程、错误处理、工具配置与通用 Qt 约定。

两份专题文档作为本总纲的展开，不再与总纲重复维护全文规则：

- `Qt_Macro_Layout_Coding_Style.md`：Qt / QML / moc 宏布局的唯一权威，回答 `Q_OBJECT`、`Q_PROPERTY`、`Q_SIGNALS:`、`Q_SLOTS`、`Q_ENUM`、metatype、d-pointer、日志宏等放在哪里。
- `Qt6_KDE_API_Parameter_Style.md`：Public API 参数语义的唯一权威，回答 Borrow / Owning、view 生命周期、QML/meta-object 边界类型、重载控制与兼容性。

当专题范围内的细节与本文件摘要不一致时，以对应专题为准；本文件只保留摘要和跳转引用，避免三份文档规则漂移。

本总纲同时是 `Q_OBJECT` 存在性项目门禁的权威来源：项目内所有 `QObject` 派生类均必须
包含 `Q_OBJECT`。这是本项目把 Qt 上游强烈建议提升为可执行门禁的政策，不应表述为 Qt
的普遍技术要求；宏的位置和顺序仍由 `Qt_Macro_Layout_Coding_Style.md` 决定。

## 0 总览
- 编译器：GCC ≥ 11 | Clang ≥ 14 | MSVC ≥ 2019
- 标准：C++17 (`set(CMAKE_CXX_STANDARD 17)`)
- 警告：`-Wall -Wextra -Wpedantic` 全开，**零警告提交**
- 格式化：项目根放置 `Qt6_CPP17_CLANG-FORMAT`（必要时可复制/链接为 `.clang-format` 供工具自动发现），提交前 `git clang-format --style=file:Qt6_CPP17_CLANG-FORMAT`
- 项目级禁止：C++ 异常（`throw`/`try`/`catch`；这是本项目自行提高的约束，见 §5.1）、RTTI、dynamic_cast、裸 new（`QObject` 派生为明确例外，见第 6 章）、单语句无 braces、64-bit enum
- Qt 宏写法：公共库、toolkit header、普通 app/internal code 全部使用无关键字宏写法：`Q_SIGNALS:`、`Q_SLOTS`、`Q_EMIT`
- Use templates wisely, not just because you can（明智地使用模板，不仅仅是因为你可以）
- Avoid C casts, prefer C++ casts (static_cast, const_cast, reinterpret_cast)
- Don't use dynamic_cast, use qobject_cast for QObjects or refactor your design, for example by introducing a type() method (see QListWidgetItem)
- Use the constructor to cast simple types: int(myFloat) instead of (int)myFloat

---

## 1 文件与编码
| 规则 | 正例 | 反例 |
|---|---|---|
| UTF-8 无 BOM | 保存为 UTF-8 | UTF-8-BOM |
| include 顺序 | clang-format 分组：① `"..."` ② `<Q...>/<Qt...>` ③ 其他 `<...>` | 顺序颠倒 |
| include 语法 | `#include <QString>` | `#include <QtCore/QString>` |
| guard 写法 | `#ifndef MYWIDGET_H ...` | `#pragma once`（仅工具可用） |

### 1.1 Include Guards
- If you would include it with a leading directory, use that as part of the include
- Put them below any license text

Example for kaboutdata.h:
```cpp
#ifndef KABOUTDATA_H
#define KABOUTDATA_H
```
Example for kio/job.h:
```cpp
#ifndef KIO_JOB_H
#define KIO_JOB_H
```
---

## 2 命名
| 类型 | 风格 | 正例 | 反例 |
|---|---|---|---|
| 类 | 大驼峰 | `class MainWindow` | `class main_window` |
| 函数/局部变量/参数 | 小驼峰 | `void updateData()` | `void updatedata()` |
| 普通类 private/protected 非静态状态成员 | `m_` + 小驼峰 | `int m_count` | `int count` |
| 经明确批准的 public 非静态直接字段 | 小驼峰，无前缀 | `QUrl sourceUrl` | `QUrl m_sourceUrl` |
| 静态/全局 | `s_` 前缀 | `static QObject *s_instance` | `static QObject *instance` |
| 常量 | `k` 前缀 | `constexpr int kMaxDepth = 3` | `const int MAX_DEPTH = 3` |
| 枚举值 | 驼峰 + 尾逗号 | `enum class Direction { North, South, };` | `enum Direction { NORTH };` |
| 命名空间 | 全小写 | `namespace app::utils` | `namespace AppUtils` |

- Avoid short or meaningless names (e.g. "a", "rbarr", "nughdeget")
- Single character variable names are only okay for counters and temporaries, where the purpose of the variable is obvious
- Wait when declaring a variable until it is needed
- Variables and functions start with a lower-case letter. Each consecutive word in a variable's name starts with an upper-case letter

### 2.1 直接字段数据类型与 Qt 上游私有数据风格

成员访问模型与成员命名必须分两阶段裁决。命名规则不能反过来授权公开状态。

#### 第一阶段：批准直接字段模型

只有以下两类类型可以明确批准 public 非静态状态采用直接字段模型：

1. **record-like 数据类型**：主要职责是承载一组数据，直接字段访问本身就是类型接口；类型不依赖 setter 拦截写入来维护关键不变量。少量不改变记录本质的构造、比较或转换函数不影响该判断。
2. **内部 PIMPL / Qt shared-data 实现类型**：定义在 `.cpp`、`private/`、`_p.h` 或等价内部实现文件，不导出，也不是稳定 Public API；它要么与公共类形成明确的 `Foo` / `FooPrivate` 实现关系，要么作为 `QSharedDataPointer<T>` / `QExplicitlySharedDataPointer<T>` 持有的内部数据实现类型。

以下事实均不能单独批准直接字段模型：使用 `class` 或 `struct`、成员已经位于 `public:`、类型名以 `Private` / `Data` 结尾、类型位于内部目录、继承 `QSharedData`。普通 manager、controller、service、worker 和其他行为类默认保持封装；不得通过扩大访问权限规避 `m_` 规则。

record-like 类型若属于已发布 Public API，其 public 字段名和类型构成源码合同，字段布局还可能构成 ABI 合同。批准直接字段模型必须同时接受相应兼容性责任。

#### 第二阶段：按已批准的访问模型命名

- 经批准的 public 非静态直接字段使用无前缀小驼峰。
- private/protected 非静态状态成员使用 `m_` + 小驼峰，包括上述类型中未公开的状态。
- 静态状态继续使用 `s_`；本节不改变现有静态/全局命名规则。
- `d`、`d_ptr`、`q_ptr` 以及 `Q_D` / `Q_Q` 生成的局部变量 `d`、`q` 是 Qt 固定名称，不改写为普通成员前缀形式。

#### record-like 数据类型

```cpp
struct DocumentSnapshot
{
    QString title;
    QUrl sourceUrl;
    QDateTime modifiedAt;
    bool readOnly = false;
};
```

普通行为类不因使用 `struct` 或公开字段更方便而获得授权：

```cpp
class DocumentController : public QObject
{
    Q_OBJECT

public:
    explicit DocumentController(QObject *parent = nullptr);
    void reload();

private:
    QString m_documentTitle;
    QUrl m_sourceUrl;
    bool m_reloadPending = false;
};
```

#### 经典 PIMPL 私有类

```cpp
class DocumentSession;

class DocumentSessionPrivate
{
    Q_DECLARE_PUBLIC(DocumentSession)

public:
    explicit DocumentSessionPrivate(DocumentSession *q)
        : q_ptr(q)
    {
    }

    QString documentTitle;
    QUrl documentUrl;
    QByteArray cachedContent;
    QHash<QString, QVariant> properties;
    QPointer<QObject> lifetimeContext;
    QMetaObject::Connection contextDestroyedConnection;
    bool dirty = false;

    struct PendingChange
    {
        QString propertyName;
        QVariant value;
    };

    QList<PendingChange> pendingChanges;

private:
    DocumentSession *q_ptr = nullptr;
};

class DocumentSession : public QObject
{
    Q_OBJECT

public:
    explicit DocumentSession(QObject *parent = nullptr);
    ~DocumentSession() override;

private:
    Q_DECLARE_PRIVATE(DocumentSession)
    Q_DISABLE_COPY_MOVE(DocumentSession)
    QScopedPointer<DocumentSessionPrivate> d_ptr;
};
```

#### Qt shared-data 实现类型

`QSharedDataPointer<T>` 提供隐式写时分离：通过非 const 路径写入时自动 detach。`QExplicitlySharedDataPointer<T>` 不自动分离；需要值语义写时分离时，由持有者在写入前显式调用 `detach()`。两种 data 类型的 public 状态命名规则相同，差别只在持有者的写入策略。

下例中的 `new T` 在同一构造表达式中立即交给 shared-data 持有者接管，不形成裸 owning 指针。

```cpp
class DocumentValueData : public QSharedData
{
public:
    QString title;
    QUrl sourceUrl;
    QDateTime modifiedAt;
};

class DocumentValue
{
public:
    DocumentValue()
        : d(new DocumentValueData)
    {
    }

    void setTitle(const QString &title)
    {
        d->title = title; // 非 const 访问按需自动 detach
    }

private:
    QSharedDataPointer<DocumentValueData> d;
};

class SharedPaletteData : public QSharedData
{
public:
    QString name;
    QColor accentColor;
};

class SharedPalette
{
public:
    SharedPalette()
        : d(new SharedPaletteData)
    {
    }

    void setAccentColor(const QColor &accentColor)
    {
        d.detach();
        d->accentColor = accentColor;
    }

private:
    QExplicitlySharedDataPointer<SharedPaletteData> d;
};
```

真正可能被多个公开值对象共享的 data 默认不得保存某一个公开实例专属的 `q_ptr`，也通常不使用 `Q_DECLARE_PUBLIC`。detach 策略由持有者负责，不改变 data 字段的命名规则。

---

## 3 缩进与括号（KDE 风格）
| 规则 | 正例 | 反例 |
|---|---|---|
| 缩进 | 4 空格 | Tab |
| 单语句 if/for/while | 必须加 braces | `if (x) {\n    return;\n}` |
| 左 brace | 控制语句附着式；函数/类/struct 换行 | `if (x) {` ... |
| else 位置 | `} else {` | `}\nelse` |
| case 缩进 | case label 缩进 1 级 | `    case 0:\n        break;` |

- 对于指针或引用，类型和'*'或'&'之间始终使用单个空格，但'*'或'&'和变量名之间不加空格：
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

## 4 行长与换行
- 软限制 100 列；二元运算符放新行首（由 clang-format 决定）；构造函数初始化列表使用“行首逗号”（KDE/clang-format）
```cpp
// 正
if (longCondition1
    && longCondition2) {
}

// 误
if (longCondition1 &&
    longCondition2) {
}
```

构造函数初始化列表示例（行首逗号）：
```cpp
Foo::Foo(int a, int b)
    : m_a(a)
    , m_b(b)
{
}
```

---

## 5 可选的现代 C++17 最佳实践（已在 Qt6/KF6 使用）

> **说明**：以下特性为**可选推荐**，而非强制要求。
> - ✅ **鼓励使用**：在新代码中优先采用这些现代化写法
> - 🔄 **渐进迁移**：现有代码可保持不变，不强制重构
> - 🤔 **权衡选择**：根据团队熟悉度、性能需求、可读性综合判断

| 场景 | 推荐 | 传统写法（仍可接受） |
|---|---|---|
| 可选返回值 | `std::optional<QColor> tryColor()` | `bool getColor(QColor *out)` |
| variant 访问 | `std::visit([](auto& v){ ... }, var)` | 手写 switch(type) |
| 结构化绑定 | `auto [it, inserted] = map.insert({k, v});` | `QPair<It,bool> res = ...` |
| 编译期常量 | `constexpr int kSize = 256;` | `const int kSize = 256;` 或 `#define` |
| nodiscard | `[[nodiscard]] int calc() const;` | 无属性（编译器不强制检查） |
| maybe_unused | `[[maybe_unused]] auto idx = ...;` | `Q_UNUSED(idx);` |
| 原子操作 | `std::atomic<int> value; value.fetch_add(1)` | `QAtomicInt` 或互斥锁 |
| 二进制缓冲 | `QByteArrayView buf` | `(const char*, size_t)` |
| 路径计算 | `std::filesystem::path p = dir / "file.txt";` | `QDir::cleanPath(dir + "/file.txt")` |
| 计时 | `auto t0 = std::chrono::steady_clock::now();` | `QElapsedTimer` |
| 折叠表达式 | `(stream << ... << args);` | 手写循环拼接 |

**使用建议**：
- 新功能/新文件：优先使用现代写法
- 维护旧代码：保持风格一致，避免混用
- 团队协作：根据团队共识选择，统一标准
- 性能敏感：实测验证，`std::optional` 等零成本抽象通常无性能损失
- `[[nodiscard]]`：对“可能失败”的 API 属于强制项（见 §5.1）；其他场景可按团队选择

### C++20+ 可选（仅当启用 C++20 及以上时）

| 场景 | 推荐 | 说明 |
|---|---|---|
| 二进制缓冲 | `std::span<const std::byte> buf` | 需要 C++20 |
| 原子视图 | `std::atomic_ref<int>(val).fetch_add(1)` | 需要 C++20 |

## 5.1 错误处理与断言（项目级无异常约束）

> Qt 和 KDE 官方规范没有把 `throw`、`try`、`catch` 作为普遍禁止的语法。本项目在 Qt/KDE 通用建议之上自行提高约束：项目 C++ 代码禁止使用这三种语法，也不把异常作为 API 错误返回或控制流机制。

### 5.1.1 总则（强制）

- **项目自定义加严约束（强制）**：项目 C++ 代码中禁止 `throw`、`try`、`catch`；不得以异常作为 API 错误返回机制或控制流。该条款是本项目自行提高的约束，不应表述为 Qt 或 KDE 的普遍技术要求。
- **必须**：任何“可能失败”的函数都要**显式表达失败**；禁止通过隐式默认值吞掉错误。
- **必须**：API 错误使用错误码、错误状态、`errorString()` 等显式接口表达；诊断文本不能替代结构化错误状态。
- **必须**：失败返回值必须带 `[[nodiscard]]`，防止调用方忽略失败（例如 `[[nodiscard]] bool ...`、`[[nodiscard]] std::optional<T> ...`）。
- **必须**：断言只用于“编程错误”（前置条件/不变量被破坏，理论上不应发生）；对用户输入、I/O、网络、插件加载等可恢复错误，必须走返回值错误路径。
- **应该**：错误信息的归属明确：底层库函数**不应**无条件 `qWarning()` 噪声刷屏；在“边界层”（UI/命令/服务入口）统一记录日志或转为用户可见信息。

### 5.1.2 失败表达选型（推荐路径）

> 目标：同一模块内尽量统一，评审时能快速判断“失败如何被处理/传播”。

**A. 不需要错误原因：`std::optional<T>`**
- **推荐**：`std::optional<T>` 表达“有/无结果”；无结果返回 `std::nullopt`。
- **必须**：在接口注释中写清 `nullopt` 的语义（未找到/不适用/解析失败等）。

**B. 需要错误原因（文本）：`bool + QString *error`**
- **推荐**：返回 `bool`，额外用 `QString *error`（允许为 `nullptr`）输出失败原因。
- **必须**：失败时若 `error != nullptr`，写入可诊断的原因；成功时不写或清空（团队统一即可）。

**C. 需要错误原因（结构化）：`bool + ErrorCode`**
- **推荐**：对可枚举的失败原因定义 `enum class ErrorCode`，并以 `ErrorCode *out` 返回（或作为成员 `lastError()`），避免大量字符串拼接。

**D. Qt 错误状态与诊断文本：`error()` / `errorString()`**
- **推荐**：对 Qt 类型提供的 `error()`、`errorString()` 等接口，先检查错误状态，再读取诊断文本；`errorString()` 只用于解释错误，不作为成功/失败判定。
- **必须**：在公共 API 注释中写明错误状态的生命周期、清除时机和 `errorString()` 的有效条件。

### 5.1.3 第三方异常依赖（强制禁止跨入）

- **禁止**：接入会把 C++ 异常传播到项目调用链的第三方 API；异常不得跨越 Qt 事件循环、信号槽、线程、插件或 QML/C++ 边界。
- **必须**：优先使用第三方库的 no-exceptions 配置或不抛异常的 API，并在模块边界转换为错误码、错误状态或 `errorString()`。
- **禁止**：以新增 `try`/`catch` 作为常规第三方适配方案；无法保证无异常传播的依赖视为不符合本项目约束。

### 5.1.4 示例（Qt6 + C++17）

**❶ optional：解析失败返回空**
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

**❷ bool + error：携带可诊断信息**
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

## 6 Qt 6 专属约定
| 规则 | 正例 | 反例 |
|---|---|---|
| `Q_OBJECT` | 项目门禁：每个 `QObject` 派生类均包含 | 省略宏或未经记录自行豁免 |
| 信号槽连接 | `connect(sender, &Sender::valueChanged, receiver, &Receiver::update);` | `SIGNAL/SLOT` 字符串 |
| 字符串字面量 | `QStringLiteral("hello")` 或 `u"hello"_qs` | `QString("hello")` |
| 线程耗时 | `QtConcurrent::run(&Worker::doWork)` | 手动 `new Thread` |
| QObject 派生语义 | 指针语义：仅用指针/引用传递与存储 | `QVector<Foo *> foos;` | `QVector<Foo> foos;` |
| 内存管理 | 值语义/RAII、parent tree、确定安全点直接析构、`deleteLater()` | 无明确 owner、重复接管、错线程销毁 |

- **规则强度**：**必须/禁止**用于内存安全、生命周期、线程正确性和单一所有权；**应该/默认**用于 Qt 6、KDE、QML library 与现代 C++17 推荐；高于 Qt/KDE 最低要求的条款必须标记为**本项目提高约束**。
- **基本原则**：必须先确定值语义、独占所有权、共享生命周期或借用观察，再选择指针类型；禁止按 Qt/std 类型家族预先选择。
- 能使用值对象、栈对象或直接成员表达生命周期时，不应进行堆分配，也不应使用智能指针。
- `T &` 表达非空借用，`T *` 表达可空借用；裸指针和引用默认不转移所有权。所有权转移必须通过类型、QObject parent、明确命名的接管接口或公共文档表达。
- 自定义 deleter、销毁线程、API/ABI 边界及既有控制块家族都是所有权合同的一部分。

### 6.1 值语义/RAII 与 QObject（团队落地版）

#### 结论（先看这个）
- `QObject` 及其派生类**不应、也无法**“强制遵守值语义（可拷贝/可移动）”。它们是典型的 **identity type（身份对象）**：语义绑定到对象身份（地址）、对象树、元对象系统、信号槽连接与线程亲和性。
- RAII 原则对 `QObject` **依然适用**，但落地方式应是：**把非 Qt 资源用 RAII 封装在成员里**；`QObject` 自身释放遵循 **parent ownership / 确定安全点直接析构 / `deleteLater()`**。把整个 `QObject` 当成可拷贝/可移动的“值对象”是误区。
- 团队执行建议：把“值语义/RAII”规则拆成两套并明确适用范围：① 一般 C++ 类型（非 `QObject`）——优先值语义 + RAII；② `QObject` 派生类——指针语义 + Qt 生命周期模型。

#### 为什么 `QObject` 不适用值语义（按维度解释）
- **可拷贝/可移动语义**：`QObject` 禁用拷贝；“移动一个 QObject”在语义上会改变地址与身份，并破坏内部状态（连接、事件、属性等），Qt 也不提供可用的 move 语义。
- **对象身份（identity）**：`QObject` 常表示“某个具体对象”（`objectName`、dynamic properties、`eventFilter`、`findChild` 等）；等价关系不是“值相等”。
- **父子对象树（ownership tree）**：parent 负责析构子对象；拷贝/移动无法定义“子树如何复制/迁移”，且容易引入双重释放/悬挂引用；并且 parent/child 必须在同一线程。
- **元对象系统（meta-object）**：`Q_OBJECT`、反射、属性系统与动态属性都与对象实例绑定；拷贝/移动并不会“复制运行期元信息/注册状态”。
- **信号槽连接**：连接以 sender/receiver 的对象身份（地址）为核心；拷贝/移动无法正确“复制连接关系”，移动还会导致既有连接语义不成立。
- **线程亲和性**：`QObject` 有 thread affinity；对象应在所属线程中处理事件/调用多数成员；跨线程销毁必须走事件循环与队列连接。
- **事件循环/延迟删除**：`deleteLater()` 通过事件循环安全析构，避免待处理事件/queued slot 造成 UAF；这与“作用域结束立即析构”的传统 RAII 时序不同。

#### 规则条款（可直接执行的规范文本）

**A. 对一般 C++ 类型（非 `QObject`）适用的值语义/RAII**
- **必须**：资源由对象生命周期管理（RAII）；避免裸 `new`/`delete`（见本规范第 0/6 章）。
- **应该**：优先值语义（可拷贝或可移动）与按值容器；独占资源类型应“可移动但不可拷贝”。只有多个 owner 确实需要独立延长生命周期时，才允许 shared ownership。
- **默认**：新通用 C++17 代码的独占所有权使用 `std::unique_ptr`，创建时优先 `std::make_unique`。
- **可保留**：Qt/KDE PIMPL、Qt cleanup policy、稳定 ABI 或既有正确代码可以保留 `QScopedPointer`；不得仅为统一风格迁移。
- **必须**：PIMPL 指向不完整类型时，持有类析构函数在实现文件中、私有类型完整的位置定义。
- **必须**：callback 只有在合同上确实是 co-owner，并明确取消、释放和循环引用处理时才可持有 shared pointer。
- **默认**：纯 C++ 或标准库边界使用配套的 `std::shared_ptr`/`std::weak_ptr`；封闭 Qt 局部实现可以使用配套的 `QSharedPointer`/`QWeakPointer`。
- **应该**：默认 deleter 优先使用 `std::make_shared` 或 `QSharedPointer::create`；需要自定义 deleter 时不得使用这两个创建接口。
- **必须**：通过基类智能指针拥有派生对象时，基类必须有可用虚析构函数，或使用能正确销毁实际类型的 deleter。
- **必须**：`QSharedDataPointer` 提供隐式写时分离；`QExplicitlySharedDataPointer` 仅在持有者显式调用 `detach()` 后写时分离。二者都表达 shared-data 值语义，不表达对象生命周期 shared ownership。
- **禁止**：用裸指针表达所有权；API 通过“返回裸指针 + 口头约定”隐式转移所有权。

**B. `QObject` 派生类（特例：指针语义 + Qt 生命周期模型）**
- **必须**：`QObject` 派生类采用身份语义，禁止拷贝和移动。
- **禁止**：按值传参/按值返回 `QObject` 派生类型；禁止将 `QObject` 派生类型按值存入 `std::vector`/`QVector`/`QList` 等需要移动/拷贝的容器。
- **本项目提高约束**：每个 `QObject` 派生类在类内显式使用 `Q_DISABLE_COPY_MOVE(Class)`，以获得类级编译期诊断并明确该合同；这是本项目门禁，不是 Qt/KDE 项目的普遍硬性要求。
- **必须**：需要 `moveToThread()` 的 `QObject` **禁止设置 parent**；禁止构造跨线程 parent/child 关系（parent/child 必须在同一线程）。
- **必须**：所有权策略唯一且可追溯：
  - **parent ownership**：`new T(parent)`，由 parent 析构释放；
  - **显式所有权**：明确唯一 owner，再按下文选择确定安全点直接析构或 `deleteLater()`。
- **本项目提高约束**：禁止同一 `QObject` 同时设置 parent 并由 owning 智能指针管理。
- **允许**：直接 QObject 成员使用包含对象作为 parent，例如 `QTimer m_timer{this}`；这不属于智能指针重复拥有。
- **应该**：当对象可能被异步销毁（parent 析构、`deleteLater`、跨线程）时，持有方用 `QPointer<T>` 等弱引用方式避免悬挂指针；跨异步边界避免缓存裸指针。
- **应该**：当子 `QObject` 生命周期与 owner 完全一致时，优先使用“直接成员 + parent”（如 `QTimer m_timer{this};`）而不是堆分配；这不是值语义，仍禁止 copy/move/按值容器。
- **应该**：创建型 API 用 `QObject *parent` 参数表达所有权；返回值通常用裸指针表达“非 owning”（由 parent/owner 管理），并在函数注释中写清生命周期。
- **应该**：默认约定：裸指针/引用参数与返回值表达“借用（non-owning）”；一旦发生所有权转移，必须通过设置 parent、明确命名（如 `takeOwnership...`）、或在注释中显式说明。
- `QPointer` 仅用于成员和延迟任务捕获；普通 API 参数仍使用 `T *` 或 `T &`。

#### 6.1.1 控制块、弱引用与 QObject 销毁路径

**控制块、weak pointer 与并发**

- 同一对象只能存在一种有效 owning 策略；同一共享对象只能属于一个 shared control block，Qt/std 混用也不得例外。
- 禁止从已经被管理对象的裸指针、`get()`、`data()`、借用 API 返回值或 `this` 重新构造 owner。
- 对象需要获取自身强引用时，只能使用匹配家族的 `std::enable_shared_from_this` 或 `QEnableSharedFromThis`。
- 后续强引用只能通过复制/移动既有 owner、保留控制块的 cast/aliasing 接口，或从匹配 weak pointer 提升。
- 无异常 C++17 代码从 weak pointer 提升时使用 `std::weak_ptr::lock()` 或 `QWeakPointer::toStrongRef()`。
- `release()`、`take()` 等解除所有权操作只能立即交给明确接管方；shared ownership cycle 必须由 weak pointer 打断。
- 引用计数安全不代表 pointee 线程安全；同一个 shared-pointer 变量的并发读写也必须同步。

**QObject 的直接析构与 shared ownership**

- 删除“所有 QObject 一律禁止直接删除”的绝对规则。QObject 优先使用 parent tree；无法使用 parent 时，必须有明确且唯一的 owner 和销毁路径。
- 只有同时满足以下条件，才允许直接析构或使用默认 deleter：对象无 parent；当前线程是 affinity thread；对象未正在处理事件或 callback；异步工作、计时器、I/O 和外部访问已经停止；销毁顺序确定。
- 事件循环、信号槽或计时器的历史使用本身不构成永久禁令；决定因素是实际销毁点是否满足同步安全条件。
- `std::unique_ptr`/`QScopedPointer` 只允许管理满足同步销毁条件的 QObject。不能证明条件时，必须使用 `deleteLater()` 或调用 `deleteLater()` 的自定义 deleter。
- `deleteLater()` 可以从其他线程安全调用，但实际删除由对象所属线程的事件循环处理。
- QPointer 的判空与解引用必须在对象所属线程完成；两者之间不得跨越可能触发重入、信号、事件处理或未知 callback 的调用。跨线程投递命令后，必须在目标线程重新检查 QPointer。
- 普通 API 参数不使用 QPointer；不得使用 QWeakPointer 观察普通 parent-owned QObject。

**shared-owned QObject（本项目提高约束）**

- QObject 默认禁止 shared ownership；优先使用 parent tree、明确单一 owner 或 `deleteLater()` 生命周期。
- 共享所有权仅作为受控例外：对象无 parent、确有多个独立 owner、只使用一个控制块家族、所有 owner/observer 来源可追溯。
- **本项目提高约束**：所有 shared-owned QObject 一律使用调用 `deleteLater()` 的自定义 deleter，不允许默认 deleter。
- 最后一个强 owner 必须在对象所属线程的事件循环停止前释放，并保证 `DeferredDelete` 有机会被处理。
- 主事件循环停止后调用 `deleteLater()`，对象不会再被该事件循环删除，可能形成永久泄漏。
- worker QObject 若依赖线程结束完成 deferred delete，最后一个 owner 必须在线程结束前释放，并通过 `destroyed()` 或等价关闭证据确认实际析构。
- 无法保证事件循环存活、最终释放时机和实际析构完成时，禁止对该 QObject 使用 shared ownership。
- shared-owned QObject 需要临时延长生命周期时使用匹配 weak pointer 提升；只检测外部 QObject 是否仍存在时使用 QPointer。
- shared pointer、weak pointer 和 QPointer 都不提供 QObject 本身的线程安全。

#### 6.1.2 QIODevice 特别说明（Qt I/O 设备对象）

> 适用范围：任何继承自 `QIODevice` 的类型（包含你们自定义派生类）。`QIODevice` 本身就是 `QObject` 派生，因此继续适用上面的 `QObject` 特例；本小节仅补充“为什么更要谨慎”与“工程落地规则”。

- **语义定位**：`QIODevice` 不是“数据容器”，而是**带状态与回调的 I/O 端点**（打开/关闭、缓冲区、异步通知信号等），强依赖对象身份与事件循环；因此**禁止值语义（copy/move/按值容器）**。
- **生命周期策略（比一般 QObject 更偏向 `deleteLater()`）**：
  - **应该**：若销毁时仍有异步通知（例如 `readyRead`/`bytesWritten`/错误信号、socket notifier、queued connection、定时器驱动读写），或无法证明待处理事件与 callback 已清空，优先 `deleteLater()`，避免 UAF。
  - **应该**：在调度销毁前先 `close()`（如适用）以尽快停止 I/O 与回调；并避免在析构顺序不清晰时把 device 指针泄露到长生命周期对象。
- **栈对象边界**：
  - **可选**：`QFile` 等在实际销毁点满足同步安全条件、指针不外泄的函数内可用栈对象。历史上连接过信号本身不构成永久禁令；若指针跨异步/作用域边界，或销毁时仍有未完成回调，应改为 parent/显式 owner，并按实际销毁点选择 `deleteLater()`。
- **owning 智能指针（强化版）**：
  - **禁止**：除满足“纯同步场景”外，禁止使用 owning 智能指针（`std::unique_ptr`/`QScopedPointer`/`std::shared_ptr`/`QSharedPointer`）以**默认 deleter**管理 `QIODevice` 及派生类。
  - **纯同步场景（必须全部满足）**：
    1. 单线程同步使用，不 `moveToThread()`
    2. 销毁时没有活动的 `readyRead`/`bytesWritten` 等异步通知、待处理 callback 或长生命周期连接
    3. 不将 `QIODevice *`/引用外泄（不保存、不跨异步边界、不交给异步 API）
    4. 不调用 `deleteLater()`，并且期望离开作用域立即析构
  - **本项目提高约束**：任何 shared-owned `QIODevice` 必须使用调用 `deleteLater()` 的自定义 deleter，禁止使用默认 deleter；并确保最后一个 owner 在目标事件循环停止前释放。
- **线程与亲和性**：
  - **必须**：设备对象只在其 thread affinity 所属线程使用；跨线程只用信号/queued connection 交互。
  - **必须**：需要 `moveToThread()` 的设备对象**不得设置 parent**。非 affinity thread 发起销毁时必须调用线程安全的 `deleteLater()`；已在 affinity thread 时，只有满足上文同步安全条件才允许立即析构。
- **API 所有权表达（建议统一成团队习惯）**：
  - **应该**：把“使用某个设备”作为依赖注入：形参用 `QIODevice *device`/`QIODevice &device` 表达借用；调用方负责确保生命周期覆盖使用期。
  - **必须**：如函数/对象需要“接管设备生命周期”，必须通过 `QObject *parent`（或明确 owner）表达所有权归属；owning 智能指针策略见上文“owning 智能指针（强化版）”。
- **常用 `QIODevice` 体系类（非穷尽）**：
  - QtCore：`QBuffer`、`QFile`、`QSaveFile`、`QTemporaryFile`、`QProcess`
  - QtNetwork：`QAbstractSocket`（`QTcpSocket`、`QUdpSocket`）、`QSslSocket`、`QLocalSocket`、`QNetworkReply`
  - QtSerialPort：`QSerialPort`
  - 备注：`QTextStream`/`QDataStream` 等**不是** `QIODevice` 子类，而是面向 `QIODevice` 的读写封装。

#### 典型示例（Qt6 + C++17）

**❶ 错误示例：把 `QObject` 当值类型 / 在默认 deleter owner 上调用 `deleteLater()`**
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
        m_widget->deleteLater(); // ❌ 对象可能先被 deleteLater 删除，随后 unique_ptr 默认 deleter 再次删除
    }

private:
    std::unique_ptr<QWidget> m_widget; // ❌ owning 智能指针与 Qt 异步销毁语义冲突
};

void badExamples()
{
    QVector<Bad> list; // ❌ QObject 派生不可拷贝/不可移动：容器按值存放不成立
}
```

**❷ 推荐示例：parent 机制 + RAII 成员协同（资源放在成员，QObject 用 Qt 生命周期）**
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
        , m_timer(this) // ✅ 子 QObject：同寿命，直接成员 + parent
        , m_backend(std::make_unique<Backend>()) // ✅ 非 QObject 资源：RAII
    {
        m_timer.setInterval(1000);
        connect(&m_timer, &QTimer::timeout, this, &Controller::poll);
        m_timer.start();
    }

    Q_DISABLE_COPY_MOVE(Controller)

private Q_SLOTS:
    void poll();

private:
    struct Backend {
        void poll();
    };

    QTimer m_timer;
    std::unique_ptr<Backend> m_backend;
    QPointer<QObject> m_maybeGone; // ✅ 弱引用：防悬挂（例如由外部 owner/parent 管理的对象）
};
```

**❸ 跨线程示例：线程亲和性与销毁策略（不跨线程直接析构，使用 queued + deleteLater）**
```cpp
#include <QObject>
#include <QThread>

class Worker : public QObject
{
    Q_OBJECT
public Q_SLOTS:
    void doWork();
Q_SIGNALS:
    void finished();
};

void startWorker(QObject *owner)
{
    auto *thread = new QThread(owner); // ✅ thread 由 owner 管理
    auto *worker = new Worker();       // ✅ 无 parent，后续在其线程 deleteLater
    worker->moveToThread(thread);

    QObject::connect(thread, &QThread::started, worker, &Worker::doWork);
    QObject::connect(worker, &Worker::finished, thread, &QThread::quit);
    QObject::connect(thread, &QThread::finished, worker, &QObject::deleteLater);
    QObject::connect(thread, &QThread::finished, thread, &QObject::deleteLater);

    thread->start();
}
```

### 6.2 信号槽连接与线程语义（强制 + 推荐）

> 目标：避免 lambda 悬挂引用、跨线程 UI 访问、queued 参数类型不匹配等高频崩溃问题；并让评审可快速审计“slot 在哪个线程执行、连接生命周期如何结束”。

#### 6.2.1 connect 语法与生命周期（强制）

- **必须**：仅使用新式 connect（函数指针/成员函数指针/带 context 的 functor）；禁止 `SIGNAL()/SLOT()` 字符串。
- **必须**：所有 Qt C++ 代码统一使用无关键字宏写法：`Q_SIGNALS:`、`Q_SLOTS`、`Q_EMIT`；禁止新增 `signals:`、`slots:`、裸 `emit`。
- **必须**：使用 lambda/functor 连接时必须提供 **context**（通常为接收者/owner），禁止使用“无 context”的 functor connect 重载。
- **必须**：lambda/functor 不得捕获可能先于连接断开而析构的裸指针；若捕获 `this`，context 必须是 `this`（或更长生命周期 owner），并优先捕获 `QPointer<T>` 做弱引用保护。
- **推荐**：重复连接点（可能被调用多次）使用 `Qt::UniqueConnection` 防止重复连接；需要手动断开时保存 `QMetaObject::Connection` 并在合适时机 `disconnect()`。

示例（推荐/禁止）：
```cpp
// ✅ 推荐：带 context；this 析构后自动断开
connect(sender, &Sender::valueChanged, this, [this](int v) { onValue(v); });

// ❌ 禁止：无 context；若捕获 this，极易 UAF
connect(sender, &Sender::valueChanged, [this](int v) { onValue(v); });
```

#### 6.2.2 线程语义（强制）

- **必须**：不得在非 GUI 线程访问 GUI 对象；GUI 更新必须回到 GUI 线程（queued connection 或 `QMetaObject::invokeMethod`）。
- **必须**：当 sender/receiver 明确跨线程（或 receiver 可能 `moveToThread()`）时，连接必须显式指定 `Qt::QueuedConnection`。
- **禁止**：跨线程使用 `Qt::DirectConnection`（除非 slot 完全线程安全且不触及 GUI/Qt 对象线程亲和性，且在评审中说明理由）。
- **禁止（默认）**：`Qt::BlockingQueuedConnection`，除非有明确同步需求并证明不会死锁（需要评审说明）。

#### 6.2.3 queued 参数类型（强制）

- **必须**：queued connection 传递的自定义类型必须可被 Qt 元类型系统识别：至少 `Q_DECLARE_METATYPE(T)`，并保证在使用前完成注册（推荐在 `main()` 或模块初始化中 `qRegisterMetaType<T>()`）。

示例（自定义类型作为 queued 参数）：
```cpp
struct Payload { int id = 0; };
Q_DECLARE_METATYPE(Payload)

// 在程序启动或模块初始化处（一次即可）
// qRegisterMetaType<Payload>("Payload");
```

### 6.3 元对象系统与属性（Q_OBJECT / Q_PROPERTY / 元类型）（项目门禁 + 强制 + 推荐）

#### 6.3.1 `Q_OBJECT` 项目强制门禁与宏选择

Qt 的最低技术要求是：使用自身信号、属性、元对象可调用方法、枚举、QML/插件元数据或
其他元对象服务的 `QObject` 派生类必须包含 `Q_OBJECT`；Qt 同时强烈建议其他
`QObject` 子类也使用该宏。本项目将后者提升为项目强制门禁：

- **必须**：项目内所有 `QObject` 派生类都包含 `Q_OBJECT`，覆盖 public、protected、
  private/internal、嵌套和辅助类型，不因当前没有信号或属性而自行缩小范围。
- **必须**：新增或修改该类时，确认项目采用的 qmake 或 CMake 配置、moc/AUTOMOC、编译和
  最终链接链路都能处理新增的元对象代码；适用时还应检查运行期元对象行为。
- **技术例外**：模板、header-only 或其他 moc/build 受限类型不得静默跳过。确实无法满足
  门禁时，必须记录技术原因、责任人和移除条件，或先调整类型/构建设计使其可处理。
- **布局边界**：`Q_OBJECT` 的位置、宏顺序和类体布局遵循
  `Qt_Macro_Layout_Coding_Style.md`；本节只决定宏是否必须存在。

本节的“必须”是本项目政策，不能直接推导为其他一般 Qt 库的通用规则。仅需要反射
（枚举/属性）但不需要 `QObject` 生命周期、信号槽的值类型，推荐使用 `Q_GADGET`；需要
在命名空间暴露枚举时使用 `Q_NAMESPACE`（配合 `Q_ENUM_NS`/`Q_FLAG_NS`）。

#### 6.3.2 Q_PROPERTY（强制）

- **必须**：可变属性必须提供 `NOTIFY`；setter 必须在值未变化时早返回，避免无意义信号与绑定抖动。
- **必须**：属性的线程亲和性要明确：若属性可能被跨线程更新，必须通过 queued 方式切回对象所属线程再修改并 `Q_EMIT`。
- **推荐**：QML-facing、public stable、明确不允许派生类覆盖的属性写 `FINAL`；需要派生类扩展、测试替身覆盖或 API 仍在演进中的属性不强制 `FINAL`。

示例（推荐写法）：
```cpp
class Person : public QObject
{
    Q_OBJECT
    Q_PROPERTY(QString name READ name WRITE setName NOTIFY nameChanged FINAL)
public:
    QString name() const { return m_name; }
    void setName(const QString &name)
    {
        if (m_name == name) {
            return;
        }
        m_name = name;
        Q_EMIT nameChanged();
    }
Q_SIGNALS:
    void nameChanged();
private:
    QString m_name;
};
```

#### 6.3.3 Q_ENUM / Q_FLAG（推荐）

- **推荐**：枚举使用 `enum class`，并用 `Q_ENUM`/`Q_ENUM_NS` 暴露到元对象系统，便于 QML/调试/反射。
- **推荐**：位标志使用 `QFlags` + `Q_FLAG`/`Q_FLAG_NS`，并通过 `Q_DECLARE_FLAGS`/`Q_DECLARE_OPERATORS_FOR_FLAGS` 统一运算符。

#### 6.3.4 元类型注册（强制）

- **必须**：任何用于 queued connection 参数、`QVariant` 存储、QML 边界（属性/信号参数/方法参数）的自定义类型必须注册为元类型（见 6.2.3）；跨模块时需确保注册发生在使用方可达路径上，避免“只在某个 .cpp 静态初始化注册导致偶发缺失”。

---

## 7 内存与单例
```cpp
// 正：函数静态
Thing& thing() {
    static Thing inst;
    return inst;
}

// 正：Q_GLOBAL_STATIC
Q_GLOBAL_STATIC(Thing, s_thing)

// 误：全局裸指针
static Thing* g_thing = new Thing;
```

---

## 8 lambda 与 auto
```cpp
// 正：多行格式
auto l = []() -> bool {
    doSomething();
    return true;
};

// 误：单行混多行
auto l = []() { doSomething();
    return true; };
```

---

## 9 项目模板结构
```
MyApp/
├── CMakeLists.txt
├── Qt6_CPP17_CLANG-FORMAT
├── .clang-tidy
├── src/
│   ├── main.cpp
│   ├── MainWindow.h
│   └── MainWindow.cpp
├── qml/              (可选)
├── resources/
│   └── resources.qrc
├── translations/
└── tests/
```

---

## 10 配置文件（直接复制到项目根）

### Qt6_CPP17_CLANG-FORMAT（通用 Qt/C++ 基线）

本仓库已提供完整配置，SSOT 为仓库根目录 `Qt6_CPP17_CLANG-FORMAT`（请以该文件为准，不再在本文档重复粘贴完整 YAML，避免漂移；如需 clang-format 默认自动发现，可复制/软链为 `.clang-format`）。

关键约定（摘要）：
- 4 空格缩进（`IndentWidth: 4`，`TabWidth: 4`，`UseTab: Never`）
- 指针风格：`Type *var`（`PointerAlignment: Right`）
- 行宽：`ColumnLimit: 100`
- include 分组：强制归组（`IncludeBlocks: Regroup`），并将 `<Q...>` 与 `<Qt.../...>` 统一归类为 Qt；块内按大小写敏感排序（`SortIncludes: CaseSensitive`）
- braces：控制语句附着式；函数/类/struct 换行（`BraceWrapping`）；不允许单行 blocks（`AllowShortBlocksOnASingleLine: Never`）
- switch/case：case labels 缩进（`IndentCaseLabels: true`，`IndentCaseBlocks: false`）
- 换行：二元运算符放新行首（`BreakBeforeBinaryOperators: All`）；构造函数初始化列表行首逗号（`BreakConstructorInitializers: BeforeComma`）
- 注释：不自动折行（`ReflowComments: false`），行尾注释对齐（`AlignTrailingComments`）

关键配置对照（换行/缩进/初始化列表/括号/空格/注释/排序）（以 `Qt6_CPP17_CLANG-FORMAT` 为准）：

| 键（Qt6_CPP17_CLANG-FORMAT） | 含义 | 示例 |
|---|---|---|
| `BraceWrapping.AfterControlStatement: Never` | `if/for/while` 的 `{` 不换行，附着在同一行 | `if (ok) {` |
| `BraceWrapping.AfterFunction: true` | 函数定义的 `{` 换行 | `void f()`<br>`{` |
| `BraceWrapping.AfterClass: true` / `AfterStruct: true` | `class/struct` 的 `{` 换行 | `class Foo`<br>`{` |
| `BraceWrapping.BeforeElse: false` / `BeforeCatch: false` | `else/catch` 与右花括号同一行 | `} else {` |
| `SpaceBeforeParens: ControlStatements` | 控制语句括号前有空格；函数调用/声明名后无空格 | `if (ok)` / `foo()` |
| `SpacesInAngles: Never` | 模板尖括号内不加空格 | `std::vector<int>` |
| `SpaceBeforeRangeBasedForLoopColon: true` | 范围 `for` 的 `:` 两侧留空格 | `for (auto v : xs)` |
| `SpacesInLineCommentPrefix.Minimum: 1` | `//` 后至少 1 个空格 | `// comment` |
| `ReflowComments: false` | 不自动重排/折行注释 | `// long long long ...`（不被自动换行） |
| `AlignTrailingComments.Kind: Always` + `SpacesBeforeTrailingComments: 1` | 行尾注释对齐，且注释前至少 1 个空格 | `int a = 1;   // a`<br>`int bb = 2;  // b` |
| `IncludeBlocks: Regroup` | `#include` 按类别归组并插入空行 | `"Foo.h"`<br>`<Q...>`<br>`<...>` |
| `IncludeCategories` | include 分类优先级：① `"..."` ② `<Qt|Q...>` ③ 其他 `<...>` | `#include "Foo.h"`<br>`#include <QString>`<br>`#include <vector>` |
| `SortIncludes: CaseSensitive` | 分组内按（大小写敏感）顺序排序 | `#include <QByteArray>` 在 `#include <QWidget>` 之前 |
| `SortUsingDeclarations: LexicographicNumeric` | `using` 按“字典序+数字序”排序 | `using Foo2 = ...;` 在 `using Foo10 = ...;` 之前 |
| `IndentWidth: 4` + `TabWidth: 4` + `UseTab: Never` | 缩进使用 4 空格，不使用 Tab | `if (ok) {`<br>`    foo();`<br>`}` |
| `ContinuationIndentWidth: 4` | 换行续行缩进 4 空格 | `foo(arg1,`<br>`    arg2);` |
| `ColumnLimit: 100` | 行宽软限制 100 列（超过时更倾向断行） | `auto s = QStringLiteral("...");`（超长时将换行） |
| `BreakBeforeBinaryOperators: All` | 换行时二元运算符放行首 | `if (a`<br>`    && b) {` |
| `BreakBeforeTernaryOperators: true` | 换行时三元运算符 `?`/`:` 放行首 | `auto x = cond`<br>`    ? a`<br>`    : b;` |
| `AllowShortBlocksOnASingleLine: Never` | 不允许 `{ ... }` 单行 block | `if (ok) {`<br>`    foo();`<br>`}` |
| `AllowShortIfStatementsOnASingleLine: Never` | 不允许 `if (x) y;` 单行 if | `if (ok) {`<br>`    foo();`<br>`}` |
| `AllowShortLoopsOnASingleLine: false` | 不允许 `for (...) x;` 单行 loop | `for (...) {`<br>`    work();`<br>`}` |
| `BreakConstructorInitializers: BeforeComma` + `PackConstructorInitializers: BinPack` | 构造函数初始化列表使用“行首逗号”风格 | `Foo::Foo()`<br>`    : m_a(a)`<br>`    , m_b(b)` |
| `ConstructorInitializerIndentWidth: 4` | 初始化列表相对函数签名缩进 4 空格 | `Foo::Foo()`<br>`    : m_a(a)` |
| `IndentCaseLabels: true` + `IndentCaseBlocks: false` | `switch` 内 case label 缩进；case 语句块按层级缩进 | `switch (x) {`<br>`    case 0:`<br>`        break;`<br>`}` |
| `AlwaysBreakTemplateDeclarations: Yes` | `template <...>` 声明与后续定义分行 | `template <typename T>`<br>`class Foo` |
| `BreakInheritanceList: BeforeColon` | 继承列表换行时 `:` 放到下一行（KDE 风格） | `class Foo`<br>`    : public Bar` |

### .clang-tidy（最小零警告集）

以下为可复制到业务仓库根目录的最小示例（按需调整）：
```
Checks: >
  -*,performance-*,readability-*,-readability-magic-numbers,modernize-*,
  -modernize-use-trailing-return-type,bugprone-*,cppcoreguidelines-*,
  -cppcoreguidelines-pro-bounds-pointer-arithmetic
WarningsAsErrors: ''
HeaderFilterRegex: '.*'
```

成员命名检查不得使用无条件的 `MemberPrefix: 'm_'`。建议按访问级别配置，并对 Qt d-pointer 固定名称建立白名单：

```yaml
CheckOptions:
  readability-identifier-naming.PrivateMemberPrefix: 'm_'
  readability-identifier-naming.ProtectedMemberPrefix: 'm_'
  readability-identifier-naming.PublicMemberPrefix: ''
  readability-identifier-naming.PublicMemberCase: 'camelBack'
  readability-identifier-naming.PrivateMemberIgnoredRegexp: '^(d|d_ptr|q_ptr)$'
  readability-identifier-naming.ProtectedMemberIgnoredRegexp: '^q_ptr$'
```

clang-tidy 只负责检查词法命名。`PublicMemberPrefix: ''` 不授权任何类型公开状态。项目还必须通过 AST allowlist 或人工评审先确认类型属于获准的 record-like 数据类型或内部 PIMPL/shared-data 实现类型，并拒绝普通行为类公开状态。类型后缀和 `QSharedData` 继承均不得作为自动授权条件。

---

## 11 提交前自检清单（Copy & Paste）

```
- [ ] UTF-8 无 BOM
- [ ] include 顺序 & guard 正确
- [ ] clang-format --dry-run 无差异
- [ ] clang-tidy 零警告
- [ ] 无异常/RTTI/dynamic_cast
- [ ] 类型在命名前已完成直接字段模型授权；`class/struct`、后缀和继承关系未被当作自动授权
- [ ] 获准的 record-like 与内部 PIMPL/shared-data 类型，其 public 非静态直接字段使用无前缀 lowerCamelCase
- [ ] private/protected 非静态状态使用 m_xxx，包括获准类型中的非公开状态
- [ ] 静态成员使用 s_xxx，q_ptr/d_ptr/d/q 保留 Qt 固定名称
- [ ] 普通类未通过扩大 public 作用域规避 m_ 规则
- [ ] QSharedDataPointer 写入依赖自动 detach；QExplicitlySharedDataPointer 的写时分离在持有者写入前显式 detach
- [ ] 单语句 if/for/while 加 braces
- [ ] 枚举尾逗号
- [ ] QStringLiteral / u""_qs
- [ ] Q_OBJECT 项目门禁：范围内每个 QObject 派生类都包含 `Q_OBJECT`
- [ ] Q_OBJECT 技术例外已正式记录；moc/AUTOMOC、编译和链接验证通过
- [ ] `QObject`：禁止 copy/move/按值传递/按值容器
- [ ] 本项目门禁：每个 `QObject` 派生类显式使用 `Q_DISABLE_COPY_MOVE(Class)`
- [ ] QObject ownership、direct-delete、`deleteLater()`、shared-owned QObject、worker shutdown 和 QPointer 规则已检查
- [ ] 非 `QObject` 独占所有权默认使用 `std::unique_ptr`
- [ ] 线程耗时任务用 QtConcurrent
- [ ] 无异常：不新增 `throw`/`try`/`catch`；可能失败的 API 明确失败表达，并用 `[[nodiscard]]` 防忽略
- [ ] 信号槽：统一使用 `Q_SIGNALS:` / `Q_SLOTS` / `Q_EMIT`；lambda/functor connect 必须带 context；跨线程显式 `Qt::QueuedConnection`；queued 参数类型已做元类型注册
- [ ] 元对象：`Q_PROPERTY` 可变属性必须有 `NOTIFY`，setter 仅在变更时 `Q_EMIT`；自定义类型用于 QVariant/QML 时已注册
```

---

**文档包版本**：v1.2.0
**最后更新**：2026-08-11

---
