# DebianOnQEMUwithProxmox

Debian qcow2 multi-arch images on QEMU with Promox

# Note for compiling qemu
```
apt-get install git libssl-dev libffi-dev build-essential python3-pip zlib1g-dev pkg-config libglib2.0-dev libpixman-1-dev bridge-utils
./configure --target-list=arm-softmmu,mips-softmmu,mips64-softmmu,mips64el-softmmu,mipsel-softmmu,aarch64-softmmu,arm-linux-user,aarch64-linux-user,mips64el-linux-user,mipsel-linux-user,mips-linux-user,mips64-linux-user --prefix=/opt/qemu --python=/usr/bin/python3
```

# Quick Start

Download `vmlinuz`, `initrd` and `qcow2` image from the release page, and start your virtual machine with QEMU.

A sample for `mipsel`:

```bash
qemu-system-mipsel  -m 512 -M malta -kernel debian.mipsel/vmlinuz \
                    -initrd debian.mipsel/initrd.img -append "root=/dev/sda net.ifnames=0 biosdevname=0 nokaslr" \
                    -hda debian.mipsel/image.qcow2 -net nic -net tap,ifname=tap109,script="./qemu-brup",downscript="./qemu-brdown" \
                    -nographic
```

A sample for `armhf`:
```bash
/opt/qemu/bin/qemu-system-arm -m 2048 -M virt -cpu cortex-a15 -smp cpus=4,maxcpus=4 \
                              -kernel debian.armhf/vmlinuz -initrd debian.armhf/initrd.img -append "root=/dev/vda net.ifnames=0 biosdevname=0 nokaslr" \
                              -hda debian.armhf/image.qcow2 -net nic -net tap,ifname=tap108,script="./qemu-brup",downscript="./qemu-brdown" \
                              -nographic
```

cat qemu-brup
```bash 
#! /bin/sh
# qemu-ifup script to bring a network (tap) device for qemu up
# ifconfig / brctl version

# You must specify the preexisting bridge interface you want the tap
# connected to in the variable "bridge" set below.
bridge=vmbr0

brctl=$(which brctl)
echo $0 connecting $1 to $bridge
ifconfig "$1" 0.0.0.0 up
brctl addif $bridge "$1"
exit
```

cat qemu-brdown
```bash 
#! /bin/sh
# qemu-tap-down script to execute just before the tap interface is removed.
# Normally this script is empty.
echo $0 removing tap interface $1
exit
```
