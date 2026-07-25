# AI 生成 C++ 代码注释规范提示词（补充 C++17/20 专属规则）
（兼容 C++11 及以上标准，简洁无冗余）

English | 简体中文 | 原文

> 说明：本文档为当前 C++ 注释规范的中文整理版（用于阅读与分发）。若与规范基线存在差异，以规范基线为准。

## 核心目标
生成C++代码时，按以下规范添加注释，仅保留关键信息，避免冗余内容干扰代码阅读。

## 注释基础规范
1. 遵循C++常规注释习惯，优先使用 `//` 进行单行注释；
2. 类/函数的整体性说明可使用 `///`（Doxygen风格简化版），无需过度格式化；
3. 注释语言简洁精准，单条注释长度控制在80字以内，复杂说明可拆分为多条短注释。

## 必须添加注释的场景（通用+C++17/20专属）
### 通用场景（适用于所有版本）
1. 类/结构体/枚举：说明核心功能或设计用途（例：“管理线程池任务队列的结构体”）；
2. 非直观成员变量：解释变量具体含义（例：“累计未处理任务超时毫秒数”）；
3. 函数：说明功能目的、非明显含义的参数、不直观的返回值、关键副作用（例：“会修改全局配置表，非线程安全”）；
4. 复杂逻辑块：仅在算法设计意图、分支决策原因不直观时添加（例：“选快速排序，因数据量小内存开销更关键”）；
5. 宏定义/常量：解释数值或逻辑的由来（例：“最大连接数1024，为系统FD限制的80%”）；
6. 特殊处理：标注失败条件/错误处理约束、线程安全约束等（例：“需外部加锁，非线程安全”）。

### Qt/QML 与 Qt/KDE library 专属场景
（前置条件：项目使用 Qt/QML，或按 Qt/KDE library 风格组织 C++ 库）
1. **公共 API 契约**：
   对 public class/struct/enum、public/protected method、`virtual` override、factory function、parser/loader、start/stop/request/dispatch 类函数，注释 API 用途、参数合法范围、返回值含义、失败条件、关键副作用、线程安全或重入限制，以及是否会改变对象状态。
2. **QML 暴露 API 与属性契约**：
   对 `Q_INVOKABLE`、public slot、QML singleton、attached property provider、QML-facing model 等暴露给 QML 的接口，注释用途、返回值、失败条件、副作用和调用约束。`Q_PROPERTY` 只在语义、默认值、副作用、稳定性或失败行为不直观时注释；可集中在类说明、对应 getter/setter 或属性组说明中，避免逐项堆注释。
3. **QObject 生命周期与所有权**：
   当 `QObject`、`QQuickItem`、裸指针、`QPointer`、event filter、lambda 捕获或 `deleteLater()` 涉及非显然所有权时，注释对象由谁持有、何时释放、是否仅观察、是否依赖事件循环析构（例：“QObject parent ownership deletes the source with the session.”）。
4. **QML/C++ 数据边界契约**：
   对 `QVariantMap`、`QVariantList`、model role、role name、QML 依赖的字符串字段、restore key、trace payload 等跨 QML/C++ 的数据结构，注释稳定字段、无效输入、空值语义和返回约定。
5. **QAbstractItemModel / role / model 数据契约**：
   对 `QAbstractItemModel`、model row 与业务对象的关系、role name、stable key、restore path、空 model 和无效 index 的返回语义，注释 QML 或测试依赖的稳定边界。
6. **Qt 事件循环、异步与重入**：
   对 `Qt::QueuedConnection`、`QMetaObject::invokeMethod(..., Qt::QueuedConnection)`、deferred operation、transaction queue、debounce/throttle、timer 生命周期、reentrancy guard、signal 触发顺序依赖和必须等下一轮事件循环的逻辑，注释时序原因和防护意图，而非解释 Qt 语法。
7. **诊断、错误码与可观测契约**：
   对 deny code、trace event kind、debug payload、日志类别、contract test 依赖的稳定字符串等，注释它们是外部可观测契约，以及被 QML、测试或工具依赖的边界。
8. **库 API public/internal/private 边界**：
   吸收 Qt/KDE library 中关于 public/private header、private implementation、导出符号、线程亲和性、信号语义、弃用说明和替代路径的注释规则；但不默认引入 API/ABI 兼容承诺。若项目明确不保证兼容，注释应说明 public/internal/private 边界和调用约束，而不是承诺稳定性。
9. **平台、安全与文件访问边界**：
   对 debug-only 行为、`file://` 读取、sandbox/portal 权限、Wayland/X11/Linux-only 分支、平台 fallback 或拒绝策略，注释限制原因、失败行为和安全边界。

### C++17专属场景
（前置条件：项目启用 C++17 或更高）
1. **结构化绑定（structured bindings）**：
   当绑定的对象结构复杂（如嵌套结构体、tuple多元素）时，注释各绑定变量的实际含义（例：“[id:用户唯一标识, score:本次考试分数]”）。
2. **if constexpr**：
   注释编译时分支的“区分逻辑”（例：“仅对整数类型执行位运算优化”），而非重复“编译时判断”的语法含义。
3. **std::optional/std::variant**：
   - 对`std::optional`：注释“为空时的默认行为”（例：“空值表示未检测到设备”）；
   - 对`std::variant`：注释“主要使用的类型分支”（例：“优先处理string类型，其余转默认值”）。
4. **折叠表达式（fold expressions）**：
   注释表达式的“聚合逻辑”（例：“对参数包求和，忽略负数”），而非语法本身。
5. **内联变量（inline variables）**：
   注释变量的“跨编译单元共享逻辑”（例：“全局配置单例，需保证初始化顺序”）。

### C++20专属场景
（前置条件：项目启用 C++20 或更高）
1. **概念（concepts）**：
   注释概念的“核心约束目的”（例：“要求类型支持随机访问迭代器”），而非重复约束条件本身。
2. **协程（coroutines）**：
   - 注释`co_await`的“等待条件”（例：“等待IO操作完成信号”）；
   - 注释`co_yield`的“产出值含义”（例：“每次产出当前帧的处理结果”）。
3. **范围（ranges）**：
   对复杂管道操作（如`views::filter | views::transform`），注释“整体数据处理目标”（例：“筛选有效用户并提取ID”）。
4. **模块（modules）**：
   注释`import`/`export`的“依赖逻辑”（例：“导入基础工具模块，无循环依赖”）。
5. **constexpr容器（如constexpr std::vector）**：
   注释“编译期初始化的特殊逻辑”（例：“编译期预计算素数表，减少运行时开销”）。
6. **三路比较运算符（<=>）**：
   注释“比较逻辑的特殊规则”（例：“按长度优先，再比较内容”），默认逐成员比较时无需注释。

## 禁止添加注释的场景
1. 简单逻辑操作（如 `i++`、`if (flag)` 等直观代码）；
2. 含义明确的变量（如 `int count`、`bool is_ready`，名称可说明意义）；
3. 常规函数（如简单getter/setter、无特殊逻辑的构造/析构，名称可说明功能）；
4. 重复代码语义的注释（如 `// 给a赋值` 注释 `a = 5;`）；
5. C++17/20新特性的“语法性说明”（如无需注释“这是结构化绑定”“这是concepts约束”，仅注释非直观逻辑）；
6. 每行代码都解释的流水账、历史聊天背景、与代码不同步的设计说明；
7. 无 owner、触发条件或后续动作的 TODO；
8. 对每个 `Q_PROPERTY`、简单 signal 或显然 role 逐项堆注释；
9. 将 KDE/Qt library 的兼容承诺写成默认要求；只有项目明确承诺兼容时才记录 API/ABI 稳定性。

---

**文档包版本**：v1.1.0
**最后更新**：2026-07-25
