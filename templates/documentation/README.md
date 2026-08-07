# Documentation profile templates

> 状态：模板、非启用配置。此目录不表示当前项目已经安装或接入任何文档生成器。

## 目录

- [`qdoc.conf.in`](./qdoc.conf.in)：Qt QDoc C++ 和 QML 的共用模板。
- [`Doxyfile.in`](./Doxyfile.in)：普通 Doxygen 的 C++ API 模板。
- [`kapidox/README.md`](./kapidox/README.md)：KDE KApiDox 的组合式集成说明。
- [`doxyqml/README.md`](./doxyqml/README.md)：Doxyqml 的组合式集成说明。

## 选择规则

| 项目情况 | 推荐模板 | 说明 |
|---|---|---|
| 一般第三方 Qt6/QML，已有 Doxygen 风格注释，发布 C++ API | `Doxyfile.in` | 默认 C++ 路线 |
| 上述项目还要发布 QML 公共 API | `Doxyfile.in` + `doxyqml/` | Doxyqml 只扩展 QML 文档 |
| 明确采用 Qt 原生文档或需要 QCH | `qdoc.conf.in` | QDoc C++/QML 共用配置 |
| 已有 Doxygen 配置 | `Doxyfile.in` | 已有配置优先，模板只作参考 |
| KDE/ECM 公共库 | `Doxyfile.in` + `kapidox/` | KApiDox 是组合式集成 |
| 不生成 API 文档 | 不使用配置文件 | 保持无生成器模式 |

## 使用边界

1. 所有 `@...@` 内容都是项目占位符，不得未经替换直接作为生产配置。
2. 模板不会新增依赖、构建目标、文档发布步骤或 QCH 输出。
3. 已有项目配置优先；模板只能在明确选择对应 profile 后使用。
4. 启用前必须填入真实路径、版本和项目元数据，并完成一次匹配的文档生成验证。
5. QCH 是选定 QDoc 或 Doxygen/KDE 工具链的输出或构建集成，不是独立 profile。

## 必须提供的项目事实

- 项目名称、版本、描述和主页。
- C++ 源码、public include 和 QML 源码目录。
- QML module URI、import path 和文档输出目录。
- 生成 HTML、XML、QHP/QCH 的需求。
- Doxygen、QDoc、KDE/ECM 和 Doxyqml 的实际版本或安装入口。
