编译 Linux 内核，在完成相关设置后执行 make 即可

[Linux Kernel Build System](https://www.kernel.org/doc/html/next/kbuild/index.html)

```shell
make x86_64_defconfig
make LLVM=1 menuconfig
# Press “y” to set
# General setup
#         ---> [*] Rust support
make LLVM=1 -j$(nproc)
```
