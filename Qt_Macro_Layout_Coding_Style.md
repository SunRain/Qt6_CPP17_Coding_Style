# Qt 宏布局代码规范

本规范隶属 `Qt6_CPP17_Coding_Style.md`，是 Qt meta-object / QML / moc 宏布局的专题展开。

适用范围：Qt 6 / KDE 风格 C++ 代码，尤其是 public header、QML toolkit、可被下游复用的组件库。

权威边界：
- 宏位置、类体布局、QML 注册宏、metatype 宏、d-pointer 宏、日志宏，以本文为唯一权威。
- Public API 参数类型、Borrow / Owning、view 生命周期与 QML/meta-object 边界类型，以 `Qt6_KDE_API_Parameter_Style.md` 为唯一权威；本文只引用结论。
- 基础格式、命名、生命周期、线程、错误处理，以 `Qt6_CPP17_Coding_Style.md` 为总纲。

## 1. 总原则

- Qt 元对象宏应集中在类体顶部，先声明 Qt/QML 合同，再进入 C++ `public:` API。
- 公共库、toolkit header、普通 app/internal code 全部使用无关键字宏写法：`Q_SIGNALS:`、`public Q_SLOTS:`、`Q_EMIT`。
- `Q_PROPERTY` 是 meta-object/QML 合同，不受 C++ `public/private` 访问控制约束，推荐放在 `public:` 之前。
- `Q_DISABLE_COPY` / `Q_DISABLE_COPY_MOVE` 推荐放在 `private:`，避免把禁用复制/移动暴露成业务 public API。
- `Q_ENUM` / `Q_FLAG` 必须紧跟对应 enum / flags 声明。
- 宏的位置优先体现 Qt 语义边界；换行和缩进交给项目 `.clang-format`。

## 2. 推荐类布局模板

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

## 3. 核心宏位置规范

| 宏 | 推荐位置 | 说明 |
|---|---|---|
| `Q_OBJECT` | QObject 派生类类体第一行 | 必须早于 `public:`，作为元对象入口 |
| `Q_GADGET` / `Q_GADGET_EXPORT` | 非 QObject value type 类体第一行 | 用于 enum、property、QML value type |
| `Q_NAMESPACE` / `Q_NAMESPACE_EXPORT` | namespace 内顶部 | 用于 namespace 级 enum 元对象 |
| `QML_ELEMENT` / `QML_NAMED_ELEMENT` | 紧跟 `Q_OBJECT` / `Q_GADGET` | 表示 QML 注册合同 |
| `QML_SINGLETON` | QML 注册宏区域 | 与 `QML_NAMED_ELEMENT` 相邻 |
| `QML_UNCREATABLE` | QML 注册宏区域 | 说明 QML 可见但不可构造 |
| `QML_ATTACHED` | QML 注册宏区域 | attached property provider 放类头部 |
| `Q_CLASSINFO` | QML 注册宏之后、`Q_PROPERTY` 之前 | 常用于 DefaultProperty 等 meta 信息 |
| `Q_INTERFACES` | `Q_OBJECT` 后的类头部 | 插件/接口实现类使用 |
| `Q_PLUGIN_METADATA` | `Q_OBJECT` 后，与 `Q_INTERFACES` 相邻 | 插件实现类使用 |
| `Q_PROPERTY` | 类头部集中声明，`public:` 之前 | QML/meta-object API 合同 |
| `Q_ENUM` | enum 声明之后 | 不要与对应 enum 分离 |
| `Q_FLAG` | `Q_DECLARE_FLAGS` 之后 | 不要与对应 flags 分离 |
| `Q_ENUM_NS` / `Q_FLAG_NS` | namespace enum / flags 声明之后 | namespace 元对象枚举 |
| `Q_DECLARE_FLAGS` | flag enum 后 | 通常在类体内紧跟 enum |
| `Q_DECLARE_OPERATORS_FOR_FLAGS` | 类外，namespace 内或头文件末尾 | 不能放类体内 |
| `Q_INVOKABLE` | public/protected 方法声明前 | QML 可调用 API 通常放 `public:` |
| `Q_SIGNALS:` | public API 后、`private:` 前 | 统一替代 `signals:` |
| `public Q_SLOTS:` | public 方法后、`Q_SIGNALS:` 前 | QML/Qt meta-call 可调用的 public slot |
| `protected Q_SLOTS:` | protected API 区 | 子类可用的 slot |
| `private Q_SLOTS:` | private 区，普通 private 方法前后均可 | 仅内部 meta-object 调用需要 |
| `Q_EMIT` | 函数体内发射信号处 | 统一替代裸 `emit` |
| `Q_DISABLE_COPY` | `private:` 开头 | 仅禁用复制 |
| `Q_DISABLE_MOVE` | `private:` 开头 | 仅禁用移动 |
| `Q_DISABLE_COPY_MOVE` | `private:` 开头 | QObject 派生类优先使用 |
| `Q_DECLARE_PRIVATE` / `Q_DECLARE_PUBLIC` | `private:` | d-pointer 模式内部桥接 |
| `Q_D` / `Q_Q` | 函数体开头附近 | d-pointer 函数体内部使用 |
| `Q_DECLARE_METATYPE` | 类型完整定义之后，头文件末尾 | namespace 类型通常在 namespace 外写全限定名 |
| `Q_DECLARE_TYPEINFO` | 类型完整定义之后 | value type 性能/移动语义声明 |
| `Q_DECLARE_INTERFACE` | 接口类完整定义之后 | 插件接口头文件末尾 |
| `Q_DECLARE_LOGGING_CATEGORY` | 头文件类外，namespace 内 | 声明日志类别 |
| `Q_LOGGING_CATEGORY` | `.cpp` include 后，namespace 或匿名 namespace 内 | 定义日志类别 |
| `Q_DECLARE_TR_FUNCTIONS` | 非 QObject 类体顶部，`public:` 前 | 需要翻译上下文时使用 |
| `Q_MOC_INCLUDE` | include 区之后、类声明之前 | 只给 moc 补充类型 include |
| `Q_REVISION` | 对应属性/方法/信号旁边 | QML 版本化 API 使用 |
| `Q_PRIVATE_SLOT` | `private:` 区 | d-pointer 私有 slot 桥接 |
| `Q_UNUSED` | 函数体开头附近 | 仅用于无法删除参数名的场景 |
| `Q_ASSERT` / `Q_ASSERT_X` | 函数体内前置条件或不变量处 | 不用于用户输入错误处理 |
| `Q_FALLTHROUGH` | switch case 有意贯穿处 | 写在贯穿位置 |
| `Q_UNREACHABLE` | 理论不可达分支末尾 | 谨慎使用 |

## 4. `Q_PROPERTY` 规范

推荐：

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

要求：

- `Q_PROPERTY` 集中放在类头部，位于 `Q_OBJECT` / `QML_*` / `Q_CLASSINFO` 之后。
- 不把 `Q_PROPERTY` 分散到 getter/setter 附近。
- 不逐项给显然属性堆注释；只有默认值、副作用、稳定性、失败行为不直观时才注释。
- QML/meta-object 边界优先使用 owning 类型，例如 `QString`、`QUrl`、`QVariant`，不要暴露 `QStringView` / `QAnyStringView` / `QByteArrayView`。
- QML-facing、public stable、明确不允许派生类覆盖的属性推荐写 `FINAL`；需要派生类扩展、测试替身覆盖或 API 仍在演进中的属性不强制 `FINAL`。
- 属性的 getter/setter 若是 QML public API，应放在 `public:`。

## 5. `Q_DISABLE_COPY` / `Q_DISABLE_COPY_MOVE` 规范

推荐：

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

要求：

- QObject 派生类默认禁止复制和移动。
- Qt6 项目优先使用 `Q_DISABLE_COPY_MOVE(Class)`。
- 若项目 Qt 版本没有 `Q_DISABLE_COPY_MOVE`，使用：

```cpp
private:
    Q_DISABLE_COPY(ClassName)
    ClassName(ClassName &&) = delete;
    ClassName &operator=(ClassName &&) = delete;
```

- 不推荐放在 `public:`，除非项目已有明确一致约定。
- 放在 `private:` 开头，位于 private 成员变量之前。

## 6. `Q_SIGNALS:` 规范

推荐：

```cpp
Q_SIGNALS:
    void titleChanged();
    void requestFailed(const QString &reason);
```

要求：

- 所有 Qt C++ 代码使用 `Q_SIGNALS:`，不要使用 `signals:`。
- 应放在 public API 和 slots 之后、`private:` 之前。
- signal 参数必须适合 Qt meta-object 边界；跨 QML/queued connection 时使用 owning 类型。
- signal 名称表达“已经发生的事实”，例如 `titleChanged()`，不要命名成命令式。
- signal 一般不写访问修饰符；`Q_SIGNALS:` 本身就是 Qt signal 区域。

## 7. `Q_SLOTS` 规范

推荐写法：

```cpp
public Q_SLOTS:
    void reset();

protected Q_SLOTS:
    void refreshFromBackend();

private Q_SLOTS:
    void handleTimeout();
```

要求：

- 所有 Qt C++ 代码使用 `Q_SLOTS`，不要使用 `slots`。
- `public Q_SLOTS:` 用于 QML、旧式 meta-call、外部对象需要调用的 slot。
- `protected Q_SLOTS:` 用于子类可扩展或可复用的 slot。
- `private Q_SLOTS:` 仅用于内部 Qt meta-object 调用。
- 如果只是现代函数指针 `connect()` 的普通接收函数，不需要声明成 slot，普通 private 方法即可。
- slot 区域应放在普通 public/protected/private 方法之后，`Q_SIGNALS:` 之前或对应访问区内。

## 8. `Q_EMIT` 规范

推荐：

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

要求：

- 所有 Qt C++ 代码使用 `Q_EMIT`，不要裸写 `emit`。
- `Q_EMIT` 只出现在函数体内。
- 状态未变化时不要发 changed signal。
- 发射信号前先完成对象状态更新，除非该 signal 明确表达“即将发生”。

## 9. enum / flags 宏规范

推荐：

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

要求：

- `Q_ENUM` 紧跟 enum。
- `Q_DECLARE_FLAGS` 紧跟 flag enum。
- `Q_FLAG` 紧跟 `Q_DECLARE_FLAGS`。
- `Q_DECLARE_OPERATORS_FOR_FLAGS` 放类外，不放类体内。
- enum 如果暴露给 QML 或测试，是稳定合同，需要中文注释说明用途。

## 10. QML 注册宏规范

推荐：

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

要求：

- `QML_ELEMENT` / `QML_NAMED_ELEMENT` 紧跟 `Q_OBJECT`。
- `QML_SINGLETON` 与 QML name 宏相邻。
- `QML_UNCREATABLE` 写在 QML 注册宏区，并给出清晰原因。
- `QML_ATTACHED` 写在 attached provider 类头部。
- `QML_DECLARE_TYPEINFO(Type, QML_HAS_ATTACHED_PROPERTIES)` 放在类型定义之后、include guard `#endif` 之前。

## 11. 插件与接口宏规范

推荐：

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

要求：

- `Q_PLUGIN_METADATA` 与 `Q_INTERFACES` 放在类头部。
- `Q_DECLARE_INTERFACE` 放接口类定义之后。
- 插件 metadata 文件路径必须稳定，不用运行时拼接。

## 12. metatype 宏规范

推荐：

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

要求：

- `Q_DECLARE_METATYPE` 必须在类型完整定义之后。
- namespace 类型通常在 namespace 外使用全限定名。
- 若类型用于 queued connection，应确保可复制、可析构，并在需要时注册 metatype。
- 不要把 view、裸借用生命周期对象注册成跨线程 payload。

## 13. 日志宏规范

头文件：

```cpp
Q_DECLARE_LOGGING_CATEGORY(hluiRuntime)
```

`.cpp`：

```cpp
Q_LOGGING_CATEGORY(hluiRuntime, "deckshell.hlui.runtime")
```

要求：

- `Q_DECLARE_LOGGING_CATEGORY` 放头文件类外。
- `Q_LOGGING_CATEGORY` 放 `.cpp` include 之后。
- category 字符串是诊断合同，命名应稳定。
- 不在头文件定义 `Q_LOGGING_CATEGORY`，避免重复定义。

## 14. d-pointer 宏规范

推荐：

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

函数体：

```cpp
void PublicApi::refresh()
{
    Q_D(PublicApi);
    d->refresh();
}
```

要求：

- `Q_DECLARE_PRIVATE` / `Q_DECLARE_PUBLIC` 放 `private:`。
- `Q_D` / `Q_Q` 放函数体开头附近。
- d-pointer 只服务 ABI/API 边界或复杂私有状态，不为小类过度引入。

## 15. 注释规范

- 类说明使用 `/// @brief` 或简洁 Doxygen 注释。
- QML-facing 类、role、稳定字符串、trace payload、错误码、生命周期边界必须注释。
- 简单 `Q_PROPERTY`、简单 signal、普通 getter/setter 不逐项堆注释。
- 注释解释“合同、边界、原因”，不要解释 Qt 语法。
- 中文注释单条尽量不超过 80 字。

## 16. 禁止项

- 禁止新增或混用 `signals:`、`slots:`、裸 `emit`；所有 Qt C++ 代码统一使用 `Q_SIGNALS:`、`Q_SLOTS`、`Q_EMIT`。
- 禁止 `Q_PROPERTY` 分散在类体多个访问区。
- 禁止把 `Q_DISABLE_COPY` 放在类体顶部或夹在 property/meta-object 区。
- 禁止 `Q_DECLARE_OPERATORS_FOR_FLAGS` 放类体内。
- 禁止 signal/slot/QML invokable 暴露 `QStringView`、`QAnyStringView`、`QByteArrayView` 等借用视图类型。
- 禁止为了通过 moc 随意移动宏，导致 C++ API 分区混乱。

## 17. 最终推荐顺序

```text
class / struct 声明
  Q_OBJECT / Q_GADGET
  QML_* 注册宏
  Q_CLASSINFO / Q_INTERFACES / Q_PLUGIN_METADATA
  Q_PROPERTY 集中声明
  public:
    enum / Q_ENUM / Q_DECLARE_FLAGS / Q_FLAG
    构造析构
    普通 public API
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
类外：
  Q_DECLARE_OPERATORS_FOR_FLAGS
  Q_DECLARE_METATYPE / Q_DECLARE_TYPEINFO / Q_DECLARE_INTERFACE
  QML_DECLARE_TYPEINFO
#endif
```
