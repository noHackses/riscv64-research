# Fedora installation on RISC-V

This guide details how to setup a fedora distribution that runs on RISC-V.

## Prerequisites

We will use Fedora Workstation. I installed it on Virtualbox

## Installation

We need to install the following packets :
-  Sphinx (Python)
-  Ninja
-  Autoconf
-  Automake
-  Libtool
-  pkg-config
-  zlib
-  glib
-  pixman

```
sudo dnf install python3-sphinx
sudo dnf install python3-sphinx_rtd_theme
sudo dnf install ninja-build
sudo dnf -y install autoconf
sudo dnf install automake
sudo dnf -y install libtool
sudo dnf -y install pkg-config
sudo dnf -y install zlib-devel
sudo dnf install glib2-devel
sudo dnf install pixman-devel
```

Downloading QEMU

```
wget https://download.qemu.org/qemu-8.2.0.tar.xz
tar xvJf qemu-8.2.0.tar.xz
cd qemu-8.2.0
```

QEMU Configuration

```
./configure --target-list=riscv64-softmmu && make
```

Downloading pre-built fedora images for RISC-V

```
wget ‘https://dl.fedoraproject.org/pub/alt/risc-v/repo/virt-builder-images/images/Fedora-Minimal-Rawhide-20200108.n.0-fw_payload-uboot-qemu-virt-smode.elf’
wget ‘https://dl.fedoraproject.org/pub/alt/risc-v/repo/virt-builder-images/images/Fedora-Minimal-Rawhide-20200108.n.0-sda.raw.xz’
unxz Fedora-Minimal-Rawhide-*-sda.raw.xz
```

Starting up RISC-V Fedora with QEMU

```
./build/qemu-system-riscv64 -nographic -machine virt -smp 4 -m 2G -kernel Fedora-Minimal-Rawhide-*-fw_payload-uboot-qemu-virt-smode.elf -bios none -object rng-random,filename=/dev/urandom,id=rng0 -device virtio-rng-device,rng=rng0 -device virtio-blk-device,drive=hd0 -drive file=Fedora-Minimal-Rawhide-20200108.n.0-sda.raw,format=raw,id=hd0 -netdev tap,id=usernet,ifname=tap0,script=no,downscript=no -device virtio-net-device,netdev=usernet
```
