# coding-style

---

## 项目简介

本仓库用于沉淀 **Qt 6 + C++17** 的编码规范、注释规范与 AI 助手协作规范，并提供 clang-format 配置，降低代码风格分歧与审查成本。

English docs: [en/](en/)

---

## 快速开始

1. 阅读编码规范：[`Qt6_CPP17_Coding_Style.md`](Qt6_CPP17_Coding_Style.md)
2. 使用 clang-format：
   - 将本仓库的 [`Qt6_CPP17_CLANG-FORMAT`](Qt6_CPP17_CLANG-FORMAT) 复制/链接到你的项目根目录并命名为 `.clang-format`
   - 对代码执行格式化，例如：`clang-format -i path/to/file.cpp`
3. 了解注释规范与 AI 协作约定：
   - [`CPP_Code_Comment_Guidelines.md`](CPP_Code_Comment_Guidelines.md)
   - [`AI_CODING_BEHAVIOR.md`](AI_CODING_BEHAVIOR.md)

## 核心命名规则

- 普通类 private/protected 非静态成员使用 `m_`。
- 合法内部 PIMPL `FooPrivate` 的 public 状态成员使用无前缀小驼峰。
- 数据型 struct 的 public 字段使用无前缀小驼峰。
- 静态成员使用 `s_`，常量使用 `k` 前缀。
- `q_ptr`、`d_ptr`、`d`、`q` 保留 Qt d-pointer 固定名称。

PIMPL 例外的完整适用条件与通用 Qt 6 示例见 [`Qt6_CPP17_Coding_Style.md`](Qt6_CPP17_Coding_Style.md#21-qt-上游风格-pimpl-私有类)。

---

## 目录结构

```text
.
├── Qt6_CPP17_Coding_Style.md            # Qt6/C++17 编码规范（中文）
├── Qt6_CPP17_CLANG-FORMAT               # clang-format 配置（可复制/链接为 .clang-format）
├── CPP_Code_Comment_Guidelines.md       # C++ 注释规范（中文）
├── AI_CODING_BEHAVIOR.md                # AI 协作与强制规范说明（中文）
├── cn/                                  # 中文分发版文档
└── en/                                  # English translations
```

---

## 文档导航

- 编码规范（中文）：[`Qt6_CPP17_Coding_Style.md`](Qt6_CPP17_Coding_Style.md)
- clang-format 配置：[`Qt6_CPP17_CLANG-FORMAT`](Qt6_CPP17_CLANG-FORMAT)
- 注释规范（中文）：[`CPP_Code_Comment_Guidelines.md`](CPP_Code_Comment_Guidelines.md)
- AI 协作约定（中文）：[`AI_CODING_BEHAVIOR.md`](AI_CODING_BEHAVIOR.md)
- Qt6 / KDE Public API 参数规范（中文）：[`Qt6_KDE_API_Parameter_Style.md`](Qt6_KDE_API_Parameter_Style.md)
- Qt6 / KDE API System Prompt（中文）：[`Qt6_KDE_API_Parameter_Style.system-prompt.md`](Qt6_KDE_API_Parameter_Style.system-prompt.md)
- English：
  - [`en/Qt6_CPP17_Coding_Style.md`](en/Qt6_CPP17_Coding_Style.md)
  - [`en/CPP_Code_Comment_Guidelines.md`](en/CPP_Code_Comment_Guidelines.md)
  - [`en/AI_CODING_BEHAVIOR.md`](en/AI_CODING_BEHAVIOR.md)
  - [`en/Qt6_KDE_API_Parameter_Style.md`](en/Qt6_KDE_API_Parameter_Style.md)
  - [`en/Qt6_KDE_API_Parameter_Style.system-prompt.md`](en/Qt6_KDE_API_Parameter_Style.system-prompt.md)
