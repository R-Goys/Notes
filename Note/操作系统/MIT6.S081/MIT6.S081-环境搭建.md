# MIT6.S081-环境搭建

## 1. 环境搭建

我看网上的环境搭建版本大多是2020年，或者ubuntu20版本的，我也跟着很久没有搭建好，在下载工具链之前，我最开始直接用git去拉取哪个工具链，之后又配置了半天，发现一个比较友好的最新版本的环境搭建办法，在这里重新分享一下

我的环境：

- Ubuntu 24.04.1 LTS
- qemu 8.2.2
- xv6-2024

下载工具链

```bash
sudo apt install git build-essential gdb-multiarch qemu-system-misc gcc-riscv64-linux-gnu binutils-riscv64-linux-gnu libglib2.0-dev libpixman-1-dev gcc-riscv64-unknown-elf
```

---

下载qemu

```bash
wget https://download.qemu.org/qemu-8.2.2.tar.xz
tar -xf qemu-8.2.2.tar.xz
cd qemu-8.2.2
./configure --disable-kvm --disable-werror --prefix=/usr/local --target-list="riscv64-softmmu"
make
sudo make install
cd ..
```

拉取xv6源码

```bash
git clone git://g.csail.mit.edu/xv6-labs-2024
cd xv6-labs-2024
git checkout util
//util是lab1实验分支
```

然后：

```bash
make
make qemu
```

此时如果出现

```bash
xv6 kernel is booting

hart 1 starting
hart 2 starting
init: starting sh
$
```

则说明成功了。

那么还需要进一步验证

检查工具链：

```bash
riscv64-unknown-elf-gcc --version
```

检查调试工具(xv6源码目录下)：

一个终端输入：

```bash
make qemu-gdb
sed "s/:1234/:26000/" < .gdbinit.tmpl-riscv > .gdbinit
*** Now run 'gdb' in another window.
qemu-system-riscv64 -machine virt -bios none -kernel kernel/kernel -m 128M -smp 3 -nographic -drive file=fs.img,if=none,format=raw,id=x0 -device virtio-blk-device,drive=x0,bus=virtio-mmio-bus.0 -S -gdb tcp::26000
```

另一个终端：

```bash
gdb-multiarch -q kernel/kernel
```

此时如果进入了gdb，这便是没有问题了！



有问题欢迎随时提
