向 Kconfig 中加入以下内容：

```
config SAMPLE_RUST_HELLO
  tristate "Print Helloworld in Rust"
  help
    Something goes here
```

向 Makefile 中加入以下内容：

```
obj-$(CONFIG_SAMPLE_RUST_HELLO) += rust_helloworld.o
```

更改该模块的配置，使之编译成模块