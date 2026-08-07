# KDE KApiDox integration template

> 状态：组合式模板说明，不是可独立运行的 KApiDox 配置。

KApiDox 以 Doxygen 为基础，并依赖项目实际采用的 KDE/ECM/KApiDox 入口。使用时：

1. 以 [`../Doxyfile.in`](../Doxyfile.in) 为 Doxygen 基础模板。
2. 将安装用 public header、公共 API 元数据和项目版本填入真实值。
3. 按项目使用的 KDE/ECM/KApiDox 版本接入对应的文档构建入口。
4. 仅把确实属于下游公共 API 的头文件纳入公共文档范围。
5. 先完成普通 HTML/XML 文档验证，再启用 QCH、tag file 或跨库链接集成。

本目录不规定一个跨 KDE 版本通用的 CMake 命令，也不自动启用 `ECMAddQch`。具体入口、
元数据和输出安装规则必须来自目标项目已经采用的 KDE/ECM 配置。
