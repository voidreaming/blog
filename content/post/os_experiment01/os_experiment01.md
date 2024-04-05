+++
title = '[mit6.s081] 操作系统01-risc-v和xv6环境搭建'
date = 2024-04-05T12:20:01+08:00
draft = true
categories = [
    "Course"
]
tags = [
    "OS_experiment",
]
image = "f1c38810bb1e0fb68612707b8bebfd00.jpg"
+++


> 关于在Macos系统下操作系统实验环境搭建的记录。学校的OS实验仿照mit6.s081 实验，所以资料大多参考网上关于mit6.s081 lab的实验安装RISC-V编译器和工具链。

注明：本实验默认配置好了 `vscode` 、 `homegrew`、 `git` 等工具。


### 关于**RISC-V编译器和工具链的介绍。**

2022 年6月22日举行 WWDC宣布Mac 电脑的 CPU 架构将从 Intel 的 x86 架构换成 Arm 架构，自此，Mac 和 iPhone 以及 ipad 使用同样的架构，统一了操作系统。

RISC-V 是一个开源的精简指令集，其相对于主流的新x86 和 ARM 处理器架构而言更为简单。RISC-V 的基础指令只有四十多条，另外更重要的一点，也是与本实验息息相关的一点就是 RISC-V 拥有完整的工具链，开发人员通过工具连=链与 cpu 进行交互，RISC-V社区已经提供了完整的工具链，并且RISC-V基金会持续维护该工具链。当前RISC-V的支持已经合并到主要的工具中，比如编译工具链gcc, 仿真工具qemu等，本次实验也会配置这些工具链。

### 安装**RISC-V GNU Compiler Toolchain** 

根据官网介绍，**RISC-V GNU Compiler Toolchain** 是 RISC-V C 和 C++ 交叉编译器。它支持两种编译模式：通用的 ELF/Newlib 工具链和更复杂的 Linux-ELF/glibc 工具链。

下载源码：

```cpp
git clone --recursive https://github.com/riscv/riscv-gnu-toolchain
```

温馨提示：耗时过长，大约半小时，建议喝茶等待。

按照官方文档，接下来构建 Newlib 交叉编译器。注意到指定安装路径

```cpp
./configure --prefix=/usr/local/opt/riscv-gnu-toolchain    #配置产物路径
make                                                       #编译指令
```

注：`./configure`: 这是一个常用的脚本，用于配置软件包的安装选项。`--prefix=/usr/local/opt/riscv-gnu-toolchain`: 这个参数指定了安装路径。

编译过程中可能会遇到权限问题

```python
% make
rm -rf stamps/build-gdb-newlib build-gdb-newlib
rm: build-gdb-newlib/config.log: Permission denied
rm: build-gdb-newlib: Permission denied
make: *** [stamps/build-gdb-newlib] Error 1
```

此时用 `sudo make` 可以解决问题。

另外编译过程中可能会遇到如下报错：

```python
configure: error: Building GDB requires GMP 4.2+, and MPFR 3.1.0+.
Try the --with-gmp and/or --with-mpfr options to specify
their locations.  If you obtained GMP and/or MPFR from a vendor
distribution package, make sure that you have installed both the libraries
and the header files.  They may be located in separate packages.
make: *** [stamps/build-gdb-newlib] Error 1
```

手动安装GMP 和 MPFR 即可。

```python
brew install gmp mpfr
```

📎若安装上面的包之后编译`riscv-gnu-toolchain`时gdb模块依然失败，报错： `make: *** [stamps/build-gdb-newlib] Error 2` 我们可以手动下载

```python
brew install riscv64-elf-gdb
```

https://formulae.brew.sh/formula/riscv64-elf-gdb

安装完成后配置环境变量。

```python
vim ~/.zshrc                                              #打开配置文件
export PATH="$PATH:/usr/local/opt/riscv-gnu-toolchain/bin"       #末尾添加此行
source ~/.zshrc                                            #使配置生效
```

此时在命令行输入`riscv64-unknown-elf-gcc -v`，如果能显示版本信息则代表安装成功。

```python
% riscv64-unknown-elf-gcc -v
Using built-in specs.
COLLECT_GCC=riscv64-unknown-elf-gcc
COLLECT_LTO_WRAPPER=/usr/local/opt/riscv-gnu-toolchain/libexec/gcc/riscv64-unknown-elf/13.2.0/lto-wrapper
Target: riscv64-unknown-elf
Configured with: /Users/nyota/Documents/03_study/C_II/OS_experiment/riscv-gnu-toolchain/gcc/configure --target=riscv64-unknown-elf --prefix=/usr/local/opt/riscv-gnu-toolchain --disable-shared --disable-threads --enable-languages=c,c++ --with-pkgversion=gc891d8dc23e1 --with-system-zlib --enable-tls --with-newlib --with-sysroot=/usr/local/opt/riscv-gnu-toolchain/riscv64-unknown-elf --with-native-system-header-dir=/include --disable-libmudflap --disable-libssp --disable-libquadmath --disable-libgomp --disable-nls --disable-tm-clone-registry --src=.././gcc --disable-multilib --with-abi=lp64d --with-arch=rv64imafdc --with-tune=rocket --with-isa-spec=20191213 'CFLAGS_FOR_TARGET=-Os    -mcmodel=medlow' 'CXXFLAGS_FOR_TARGET=-Os    -mcmodel=medlow'
Thread model: single
Supported LTO compression algorithms: zlib
gcc version 13.2.0 (gc891d8dc23e1) 
```

### 安装 QEMU

QEMU是一个通用的开源机器模拟器和虚拟化器，用于在 ARM 架构上模拟RISC-V架构的CPU

直接通过 brew 安装即可

```python
brew install qemu 
```

### 最后下载xv6 源码

```python
git clone git@github.com:mit-pdos/xv6-riscv.git
```

注意xv6-riscv-fall19已经停止维护了，所以要从https://github.com/mit-pdos/xv6-riscv下载。

在项目目录下编译，如果能进入xv6的shell则表示实验环境已搭建成功.

```python
make
make qemu
```

到目前为止我们已经完成环境的配置，可以开始调试代码啦～👋