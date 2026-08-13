# Gamepad UI Toolkit / Linux 输入与手柄库注释增量

[English translation](../en/Gamepad_Input_Comment_Guidelines.md) | 简体中文（领域增量） |
[根目录兼容入口](../Gamepad_Input_Comment_Guidelines.md)

> 权威关系：本文是 `CPP_Code_Comment_Guidelines.md` 的中文领域增量，只列出输入与
> 手柄代码中容易遗漏的隐藏契约，不独立决定注释形式、覆盖范围或文档生成器。发生
> 冲突时以同目录的注释专题权威为准。
>
> 注释语法必须服从项目已选的 QDoc、Doxygen、KApiDox 或无生成器 profile。项目未配置生成器时，不得仅因本文引入 Doxygen/KApiDox 工具链；示例中的 `///` 只表示“稳定 API 合同注释”，实际标记按所选 profile 调整。

## 核心原则

注释只解释命名、类型和局部代码无法稳定表达的信息：输入语义、状态机边界、平台差异、设备生命周期、映射契约、错误恢复和诊断契约。

不要用注释复述代码字面行为。能从代码直接看懂的内容，不加注释。

## 注释形式

公共类型、公共函数和稳定契约按 `CPP_Code_Comment_Guidelines.md` 及项目文档 profile 书写。以下示例采用 Doxygen 风格：

```cpp
/// @brief Packs key and relevant modifiers into a stable chord code.
static quint64 makeChordCode(int key, Qt::KeyboardModifiers modifiers);
```

`.cpp` 内关键逻辑使用短注释：

```cpp
// Bump epoch before replacing mappings so stale releases can be rejected.
setMappingEpoch(m_mappingEpoch + 1);
```

单条注释保持简短；复杂含义拆成多条短句。注释解释意图，不解释 C++、Qt 或 Linux API 基本语法。

## 必须添加注释的内容

### 1. 输入事件语义

必须注释：

- press / repeat / release 状态机；
- action event 生命周期；
- consumed 语义；
- synthetic event 语义；
- duplicate press/release 抑制；
- release 丢失时的恢复策略；
- 设备断开时 held state 的清理策略。

示例：

```cpp
// Suppress duplicate press/release pairs from repeated or paired chords.
return;
```

### 2. 手柄映射与 action id

必须注释：

- physical button 到 action id 的映射规则；
- axis / trigger 到 action 的映射规则；
- action id 是否 canonical；
- action id 是否属于稳定契约；
- mapping preset 的默认语义；
- keyboard-to-gamepad mirror 规则；
- player index / device id 分配规则。

示例：

```cpp
/// @brief Builds a named built-in mapping normalized for players/devices.
static KeyboardActionMapping preset(const QString &presetName,
                                    int playerCount,
                                    int deviceIdBase);
```

### 3. Mapping epoch 与 hot-switch

必须注释：

- mapping epoch / generation 的作用；
- mapping reload 何时允许发生；
- held action 未释放时如何处理；
- stale release 如何识别；
- force release 的合成事件语义；
- mapping switch 是否等待 safe point。

示例：

```cpp
// Apply mapping changes only after the final release event is delivered.
maybeSchedulePendingKeyboardMappingReload();
```

### 4. Axis / trigger / deadzone

需要注释：

- deadzone 默认值来源或策略；
- hysteresis 的目的；
- trigger threshold 的业务含义；
- axis range 归一化规则；
- diagonal policy；
- ramp / interpolation 策略；
- 反向轴同时按下时的处理规则。

示例：

```cpp
// Ramps interpolate from the current value to avoid direction jumps.
state->startValue = state->value;
```

### 5. 真实手柄与键盘模拟优先级

必须注释：

- real gamepad 与 keyboard emulation 同时存在时谁优先；
- auto / keyboard-only / real-only / both 的行为；
- text entry 时是否暂停 keyboard mirror；
- keyboard event 是否 pass-through；
- exclusive capture 条件；
- event source 过滤范围。

示例：

```cpp
// Auto mode gives real devices priority and disables the mirror keyboard.
syncKeyboardActionSourceRunningState();
```

### 6. Linux 输入后端

涉及 evdev、libinput、udev、hidraw、SDL、Steam Input 等后端时，需要注释：

- 设备发现和过滤规则；
- hotplug 处理策略；
- 设备身份稳定规则；
- 权限失败和降级路径；
- exclusive grab 行为；
- kernel repeat 是否忽略；
- axis / hat / trigger 归一化；
- vendor-specific quirk；
- 手柄断连清理策略。

注释重点是平台行为和边界，不解释 Linux API 字面用法。

### 7. 多设备、多玩家与路由

需要注释：

- device id 生成策略；
- player index 分配规则；
- 多手柄排序策略；
- active device 选择规则；
- route target 选择规则；
- overlay / page / focus 对输入路由的影响；
- modal overlay 是否吞输入。

示例：

```cpp
// Overlay routes stay modal; macro routing only applies to base content.
```

### 8. Gamepad UI 导航与焦点恢复

必须注释：

- focus restore 策略；
- stable key / index restore 差异；
- focus lock；
- queued move direction；
- spatial fallback；
- view delegate recycling 特殊处理；
- manual focus edge；
- route boundary / macro router。

示例：

```cpp
// Preserve the stable key while delegates are being recycled by the view.
storeKey->insert(QStringLiteral("itemKey"), existingItemKey);
```

### 9. 输入诊断与可观测性

需要注释：

- input trace 字段含义；
- drop reason；
- deny reason；
- macro router degrade reason；
- debug state 中稳定字段；
- raw device event 与 normalized action event 的对应关系。

示例：

```cpp
/// Macro-router trace codes used by diagnostics and static contract gates.
enum class MacroRouterCode
```

### 10. 安全与用户体验边界

需要注释：

- 为什么忽略某些输入；
- 为什么阻止某些重复事件；
- 为什么某些输入不穿透；
- 为什么 text entry 下关闭键盘映射；
- 为什么断连时合成 release；
- 为什么 fallback 优先级固定。

这类注释用于防止未来维护者误删“看似多余”的保护逻辑。

## 禁止添加注释的内容

不要添加：

- 复述代码字面行为的注释；
- 解释 C++ / Qt / Linux API 基本语法的注释；
- 简单 getter / setter 注释；
- 明确变量名已经表达含义的注释；
- 每行代码都解释的流水账；
- 无 owner / 条件 / 动作的 TODO；
- 历史聊天背景；
- 与代码不同步的设计说明；
- “这里创建对象”“这里发送信号”“循环列表”这类注释。

错误示例：

```cpp
// Set enabled.
m_enabled = enabled;

// Emit changed signal.
emit enabledChanged();
```

## 判断标准

添加注释前先问：

1. 调用者是否需要知道隐藏契约？
2. 维护者是否可能误删关键保护逻辑？
3. 是否涉及输入状态、平台差异、设备生命周期、错误恢复或稳定诊断契约？

任一为“是”，添加简短注释。全为“否”，不加注释。
