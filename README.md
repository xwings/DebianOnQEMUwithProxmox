# DebianOnQEMUwithProxmox

Debian qcow2 multi-arch images on QEMU with Promox

# Quick Start

Download `vmlinuz`, `initrd` and `qcow2` image from the release page, and start your virtual machine with QEMU.

A sample for `mipsel`:

```bash
qemu-system-mipsel  -m 512 -M malta -kernel debian.mipsel/vmlinuz \
                    -initrd debian.mipsel/initrd.img -append "root=/dev/sda net.ifnames=0 biosdevname=0 nokaslr" \
                    -hda debian.mipsel/image.qcow2 -net nic -net tap,ifname=tap109,script="./qemu-brup",downscript="./qemu-brdown" \
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
