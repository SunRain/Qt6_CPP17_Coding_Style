# coding-style

---

## 项目简介

本仓库用于沉淀 **Qt 6 + C++17** 的编码规范、注释规范与 AI 助手协作规范，并提供 clang-format 配置，降低代码风格分歧与审查成本。

English docs: [en/](en/)

---

## 快速开始

1. 阅读中文权威编码规范：[`cn/Qt6_CPP17_Coding_Style.md`](cn/Qt6_CPP17_Coding_Style.md)
   （根目录 [`Qt6_CPP17_Coding_Style.md`](Qt6_CPP17_Coding_Style.md) 仅为兼容入口）
2. 使用 clang-format：
   - 将本仓库的 [`Qt6_CPP17_CLANG-FORMAT`](Qt6_CPP17_CLANG-FORMAT) 复制/链接到你的项目根目录并命名为 `.clang-format`
   - 对代码执行格式化，例如：`clang-format -i path/to/file.cpp`
3. 了解注释规范与 AI 协作约定：
   - [`cn/CPP_Code_Comment_Guidelines.md`](cn/CPP_Code_Comment_Guidelines.md)
   - [`cn/AI_CODING_BEHAVIOR.md`](cn/AI_CODING_BEHAVIOR.md)

## 核心命名规则

- 先明确批准类型采用直接字段模型，再决定字段命名；命名规则本身不授权公开状态。
- 只有 record-like 数据类型，以及内部 PIMPL / Qt shared-data 实现类型可以获得直接字段授权。
- 获准的 public 非静态直接字段使用无前缀小驼峰；private/protected 非静态状态使用 `m_`。
- `class/struct`、`Private` / `Data` 后缀、内部路径和 `QSharedData` 继承均不自动授权公开状态。
- 静态成员使用 `s_`，常量使用 `k` 前缀。
- `q_ptr`、`d_ptr`、`d`、`q` 保留 Qt d-pointer 固定名称。

两阶段裁决、record-like、普通行为类、经典 PIMPL，以及隐式/显式 shared-data 示例见 [`cn/Qt6_CPP17_Coding_Style.md`](cn/Qt6_CPP17_Coding_Style.md#21-直接字段数据类型与-qt-上游私有数据风格)。

---

## 目录结构

```text
.
├── Qt6_CPP17_Coding_Style.md            # 根目录兼容入口
├── Qt6_CPP17_CLANG-FORMAT               # clang-format 配置（可复制/链接为 .clang-format）
├── CPP_Code_Comment_Guidelines.md       # 注释专题的根目录兼容入口
├── AI_CODING_BEHAVIOR.md                # AI 派生文档的根目录兼容入口
├── cn/                                  # 中文权威文档
└── en/                                  # English translations
```

---

## 文档导航

- 编码规范（中文权威）：[`cn/Qt6_CPP17_Coding_Style.md`](cn/Qt6_CPP17_Coding_Style.md)
- 根目录兼容入口：[`Qt6_CPP17_Coding_Style.md`](Qt6_CPP17_Coding_Style.md)
- clang-format 配置：[`Qt6_CPP17_CLANG-FORMAT`](Qt6_CPP17_CLANG-FORMAT)
- 注释规范（中文权威）：[`cn/CPP_Code_Comment_Guidelines.md`](cn/CPP_Code_Comment_Guidelines.md)
- 手柄输入注释增量（中文）：[`cn/Gamepad_Input_Comment_Guidelines.md`](cn/Gamepad_Input_Comment_Guidelines.md)
- QML 规范（中文权威）：[`cn/Qt6_QML_Coding_Style.md`](cn/Qt6_QML_Coding_Style.md)
- Qt 宏布局规范（中文权威）：[`cn/Qt_Macro_Layout_Coding_Style.md`](cn/Qt_Macro_Layout_Coding_Style.md)
- AI 协作约定（中文派生）：[`cn/AI_CODING_BEHAVIOR.md`](cn/AI_CODING_BEHAVIOR.md)
- C++ 注释规范与 AI 编码行为完整审查结论（中文）：[`doc/CPP_Comment_and_AI_Behavior_Review_Conclusion.md`](doc/CPP_Comment_and_AI_Behavior_Review_Conclusion.md)
- Qt6 / KDE Public API 参数规范（中文权威）：[`cn/Qt6_KDE_API_Parameter_Style.md`](cn/Qt6_KDE_API_Parameter_Style.md)
- Qt6 / KDE API System Prompt（中文派生）：[`cn/Qt6_KDE_API_Parameter_Style.system-prompt.md`](cn/Qt6_KDE_API_Parameter_Style.system-prompt.md)
- English：
  - [`en/Qt6_CPP17_Coding_Style.md`](en/Qt6_CPP17_Coding_Style.md)
  - [`en/CPP_Code_Comment_Guidelines.md`](en/CPP_Code_Comment_Guidelines.md)
  - [`en/Gamepad_Input_Comment_Guidelines.md`](en/Gamepad_Input_Comment_Guidelines.md)
  - [`en/AI_CODING_BEHAVIOR.md`](en/AI_CODING_BEHAVIOR.md)
  - [`en/Qt6_QML_Coding_Style.md`](en/Qt6_QML_Coding_Style.md)
  - [`en/Qt_Macro_Layout_Coding_Style.md`](en/Qt_Macro_Layout_Coding_Style.md)
  - [`en/Qt6_KDE_API_Parameter_Style.md`](en/Qt6_KDE_API_Parameter_Style.md)
  - [`en/Qt6_KDE_API_Parameter_Style.system-prompt.md`](en/Qt6_KDE_API_Parameter_Style.system-prompt.md)
