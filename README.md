## 简介

Linux Runtime 来源于 Yongbao 构建系统中 linux_runtime 组件，因其相对独立的构建内容而将其独立出来，作为用于创建指定架构 Linux 运行时 或 sysroot 的构建工具。

本构建工具可创建 Linux 运行时（Linux Runtime），用来为二进制翻译（如 QEMU 、LATX User 等）提供目标架构二进制所需的基础库；
创建的内容也可作为 Linux sysroot 使用，用于提供在对目标架构进行交叉编译时所需的库文件及二进制链接的支持。


## 构建环境准备

本构建工具可在多种 Linux 发行版进行使用，以下提供部份发行版在构建时需准备的环境步骤。

#### Fedora

安装依赖：

```
sudo dnf install @c-development glibc-langpack-zh gawk gettext vim wget curl autoconf \
		texinfo flex bison rsync pkg-config cmake ninja gperf patch openssl git kmod \
		zlib-devel python3-devel libffi-devel glib2-devel gettext-devel expat-devel \
		gmp-devel mpfr-devel libmpc-devel libxml2-devel pcre2-devel libyaml-devel \
		fuse-overlayfs xmlto groff doxygen patchelf squashfs-tools
```


#### Ubuntu/Debian

安装依赖：

```sh
sudo apt-get install ca-certificates locales gettext git curl wget rsync \
		build-essential autoconf gawk flex bison cmake python3 pkg-config \
		zlib1g-dev libgmp-dev libmpfr-dev libmpc-dev libexpat1-dev \
		libssl-dev libpcre2-dev libffi-dev libxml2-dev libyaml-dev libglib2.0-dev \
		fuse-overlayfs autopoint kmod texinfo gperf doxygen patchelf \
		libxml2-utils xmlto groff squashfs-tools
```

设置语言环境：
```sh
sudo locale-gen zh_CN.UTF-8
```

## 构建存储空间需求

请保留不少于 10G 的存储空间用于编译和构建过程。

## 下载和构建

使用以下步骤进行下载和构建：

#### 下载：

```sh
git clone https://github.com/YongbaoOS/Linux_Runtime.git
```

#### 构建：
可以使用默认方式进行构建，构建命令如下：

```sh
pushd linux_runtime/main
	./build.sh
popd
```

默认方式的构建将生成 x86_64 架构的 Linux Runtime，并存放在 linux_runtime 目录中。

构建过程时间取决于当前运行构建环境的硬件性能。

#### 指定架构：
若想生成其它架构的 Linux Runtime，例如 LoongArch 64位的 loongarch64 ，可使用如下命令：

```sh
pushd linux_runtime/main
	./build.sh -e target=loongarch64 -S loongarch64_runtime
popd
```

这里提供了两个参数：

-e target=loongarch64 ，该参数表示为 loongarch64 架构创建 Linux Runtime ；

-S loongarch64_runtime, 该参数表示创建 Linux Runtime 存放在 loongarch64_runtime 目录中，后续打包时也会使用该名称。


## 支持的架构列表

目前支持的架构名称有：aarch64 i686 loongarch64 mips64el riscv64 x86_64 x86_64-v2 x86_64-v3


## 打包和产出

构建完成后为方便使用，需要对其进行整理和打包

#### 整理
整理是为了让生成的成品减少多余的内容，整理命令：

```sh
pushd linux_runtime/main
	./strip_os.sh -f
popd
```
上述步骤会对所有已生成的 Linux Runtime 目录进行整理，若只想处理某一个架构，可指定构建时设置的名称，例如：

```sh
pushd linux_runtime/main
	./strip_os.sh -f loongarch64_runtime
popd
```
这将仅处理 loongarch64_runtime 目录中的文件。

#### 打包
整理完成后可进行打包，使用如下命令进行打包：

```sh
pushd linux_runtime/main
	./pack_os.sh -s -f
popd
```
此步骤也是将对所有已整理的架构目录进行打包，如需要只处理某一个架构，可以指定该架构创建时的目录名。

打包格式默认为 squashfs ，如想使用 tar 的压缩包格式，可使用如下的参数
```sh
./pack_os.sh -s -f -m tar
```
#### 产出物
打包完成后可在 main/workbase/dist/os/tar/ 查寻已打包文件，若是 squashfs 格式则在 main/workbase/dist/os/squashfs 目录中查寻。

查找到的文件命名以指定目录名且以tar.xz格式的（如 loongarch64_runtime.tar.xz）则为 Linux Runtime，解压后可用于二进制翻译等用途；

若需要作为sysroot进行交叉编译的，则需要将同名且带 devel.tar.xz 的文件（如 loongarch64_runtime.devel.tar.gz）一起解压使用。



