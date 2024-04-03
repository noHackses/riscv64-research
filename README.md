# riscv64-research

Guide and summary of a university research project with aim to deploy a RISC-V64 virtual machine with Linux. This repository will include multiple guides that summarise our work on networking and K3S/Kubernetes on a Linux distrib (Debian) running with a QEMU VM on a riscv64 architecture.

# Introduction

RISC-V is an open standard instruction set architecture based on established reduced instruction set computer (RISC) principles : what is it, how can we install it?

Its noteworthy feature lies in its departure from the norm from major Instruction Set Architectures (ISAs) such as x86, ARM, and AVR, as it is offered under a [permissive license](https://riscv.org/about/faq/). While this characteristic is not entirely unprecedented, RISC-V holds an additional advantage in the realm of technical enthusiasts: it has been entirely conceived from the ground up.

Currently, RISC-V boards are available for purchase, with SiFive offering several options. But why spend money when you can avoid it altogether? Let's explore how to configure a RISC-V virtual machine without incurring any costs.

# Debian installation on RISC-V64

The installation I made was on Ubuntu Desktop 23.10.1 installed in Virtual box in Windows 11 on an AMD Machine (Ryzen 5 5600) (ubuntu-23.10.1-desktop-amd64).
No Linux distribution has an official port for RISC-V yet (as of March 26th 2024), so we need to find a port. Obviously it will not be optimal but it gets us a place to start. 

## Prerequisites

First of all, we need to install a few packets :
-	QEMU (Quick EMUlator) is an open source virtual machine for Linux, Mac, Windows... It’s very useful since it supports a wide range of guest architectures – x86, the various flavors of ARM, m68k, S390x, and even RISC-V.
-	OpenSBI is a standardized piece of glue for RISC-V platforms to switch from firmware mode (M-Mode) up to the kernel layer (S-Mode). Its source code is available here, and a high-ish level overview of the RISC-V boot process is available here. U-Boot is an open source bootloader which supports a wide variety of architectures, including, but not limited to, RISC-V. For Ubuntu 20.10 and Debian unstable.

```
sudo apt install qemu-system-riscv64
sudo apt install u-boot-qemu opensbi
```

## Installation

First we need to create a directory where we will install all the elements needed for the virtual machine. I chose to work in `/home/riscv`. Simply open the terminal and type :

```
mkdir riscv
cd riscv
```

Then, let's download a pre-built ported image of Debian for RISC-V64 :

```
wget "https://gitlab.com/api/v4/projects/giomasce%2Fdqib/jobs/artifacts/master/download?job=convert_riscv64-virt" -O debian-rv64.zip
mkdir debian-rv64
cd debian-rv64
unzip ../debian-rv64.zip
cd dqib_riscv64-virt
```

This is not required but let’s make a backup image so that, in case something is broken inside the virtual machine, this will allow to go back to a clean image. 

```
sudo apt-install qemu-utils
qemu-img create -o backing_file=image.qcow2,backing_fmt=qcow2 -f qcow2 overlay.qcow2¨
```

If something ever breaks, simply delete `overlay.qcow2` and run the command again.

## Booting the virtual machine

Then, everything should be ready. You can boot the virtual machine :
```
qemu-system-riscv64 \
    -machine virt \
    -cpu rv64 \
    -m 1G \
    -device virtio-blk-device,drive=hd \
    -drive file=overlay.qcow2,if=none,id=hd \
    -device virtio-net-device,netdev=net \
    -netdev user,id=net,hostfwd=tcp::2222-:22 \
    -bios /usr/lib/riscv64-linux-gnu/opensbi/generic/fw_jump.elf \
    -kernel /usr/lib/u-boot/qemu-riscv64_smode/uboot.elf \
    -object rng-random,filename=/dev/urandom,id=rng \
    -device virtio-rng-device,rng=rng \
    -append "root=LABEL=rootfs console=ttyS0" \
    -nographic
```
Let see what everything does :
-	cpu rv64: Emulate RISC-V 64-bit.
-	m 1G: 1 GB of RAM. Tweak as needed.
-	netdev user,id=net,hostfwd=tcp::2222-:22: Make port 22 accessible as localhost:2222. This lets us forward SSH connections.
-	bios /usr/share/qemu/opensbi-riscv64-generic-fw_dynamic.elf: If needed, replace with the location of your OpenBSI. But make sure it’s the same configuration.
-	kernel ./u-boot-qemu/uboot.elf: Replace with the path to your U-Boot image.
-	nographic: Run in serial TTY mode. This should give you decently-accurate terminal emulation.

Log in with :
```
Admin
login: root
password: root

User
login: debian
password: debian
```

You’re logged in on Debian running on a RISC-V64 Architecture !

To boot it in the future, start a terminal, type `cd /riscv64/debian-rv64/dqib_riscv64-virt/` and then type the qemu command just above.

## SSH access

Replacing -nographic with -daemonize makes qemu run in the background. You can log in using either username/password :
```
ssh root@localhost -p 2222
ssh debian@localhost -p 2222
```

## Installing k3s on the system

To begin the installation of k3s on Debian running on RISC-V64, we need to install some packages as out Debian installation is relatively barebones. Be aware that I used the root account of the debian installation, if you are using the user account you will need to run in with sudo.
'''
apt-get update
apt-get upgrade #not necessary but recommended

apt-get install wget
apt-get install curl
'''

In summary, as we are using a pre-compiled image, the following commands should get you up and running with K3s directly :
```
# Download
wget https://github.com/CARV-ICS-FORTH/k3s/releases/download/20230721/k3s-riscv64.gz.aa
wget https://github.com/CARV-ICS-FORTH/k3s/releases/download/20230721/k3s-riscv64.gz.ab
wget https://github.com/CARV-ICS-FORTH/k3s/releases/download/20230721/k3s-riscv64.gz.ac
cat k3s-riscv64.gz.* | gunzip > /usr/local/bin/k3s
chmod +x /usr/local/bin/k3s

# Install
curl -sfL https://get.k3s.io > k3s-install.sh
INSTALL_K3S_SKIP_DOWNLOAD="true" bash -x k3s-install.sh
```
