Linux 内核在其 v6.1 版本中将对 Rust 的支持合并到主线，目的是帮助确定 Rust 语言是否适合内核。目前，Rust 支持主要面向对 Rust 支持感兴趣的内核开发者和维护者，以便他们可以开始着手抽象和驱动程序的开发，同时帮助基础设施和工具的开发。详细信息请见：[the rust experiment](https://www.kernel.org/doc/html/latest/rust/index.html#the-rust-experiment)。

[Rust for Linux](https://rust-for-linux.com/) 为在 Linux 内核中用 Rust 写内核模块提供支持，简称 R4L。

关于为 Linux 开发驱动程序的相关基本概念，参考书籍：
- 《嵌入式 Linux 设备驱动程序开发指南（原书第2版）》

# 其它
在 VSCode 中，需要将包含 rust-project.json 配置文件的目录打开或添加到工作区中，rust 项目才能被 rust-analyzer 解析

在 [Rust for Linux 官方文档](https://rust-for-linux.github.io/docs/alloc/index.html) 中的 kernel crate 下找不到 pci 模块的相关说明，本项目中的 pci 模块为非官方版本