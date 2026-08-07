# Doxyqml integration template

> 状态：组合式模板说明，不是可独立运行的 QML 文档生成器。

Doxyqml 是 Doxygen 的 QML 扩展。使用时：

1. 以 [`../Doxyfile.in`](../Doxyfile.in) 为基础配置。
2. 将真实 QML 源码目录加入 Doxygen 输入范围。
3. 按项目安装的 Doxyqml 版本配置 QML 过滤器或构建入口。
4. 验证 QML 类型、属性、信号和方法都出现在生成结果中。

过滤器命令和配置键可能随 Doxyqml 集成方式变化，本模板不编造一个跨版本的固定命令。
Doxyqml 必须与基础 Doxygen 一起使用；没有 Doxygen 配置时，按“无生成器”处理。
