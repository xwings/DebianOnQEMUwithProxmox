# DebianOnQEMUwithProxmox

Debian qcow2 multi-arch images on QEMU with Promox

# Note for compiling qemu
```
apt-get install git libssl-dev libffi-dev build-essential python3-pip zlib1g-dev pkg-config libglib2.0-dev libpixman-1-dev bridge-utils ninja-build
./configure --target-list=arm-softmmu,mips-softmmu,mips64-softmmu,mips64el-softmmu,mipsel-softmmu,aarch64-softmmu,arm-linux-user,aarch64-linux-user,mips64el-linux-user,mipsel-linux-user,mips-linux-user,mips64-linux-user --prefix=/opt/qemu --python=/usr/bin/python3
```

# Quick Start

Download `vmlinuz`, `initrd` and `qcow2` image from [wtdcode repo](https://github.com/wtdcode/DebianOnQEMU/releases)
`mipsel`:

```bash
qemu-system-mipsel  -m 512 -M malta -kernel debian.mipsel/vmlinuz \
                    -initrd debian.mipsel/initrd.img \
                    -append "root=/dev/sda net.ifnames=0 biosdevname=0 nokaslr" \
                    -hda debian.mipsel/image.qcow2 \
                    -net nic -net tap,ifname=tap109,script="./qemu-brup",downscript="./qemu-brdown" \
                    -nographic
```

`armhf`:
```bash
qemu-system-arm -m 2048 -M virt -cpu cortex-a15 -smp cpus=4,maxcpus=4 \
                              -kernel debian.armhf/vmlinuz -initrd debian.armhf/initrd.img \
                              -append "root=/dev/vda net.ifnames=0 biosdevname=0 nokaslr" \
                              -hda debian.armhf/image.qcow2 -net nic \
                              -net tap,ifname=tap108,script="./qemu-brup",downscript="./qemu-brdown" \
                              -nographic
```

`aarch64`:
```
qemu-system-aarch64 -m 512 -M virt -cpu cortex-a57 -kernel  debian.aarch64/vmlinuz \
                              -initrd  debian.aarch64/initrd.img \
                              -append "console=ttyAMA0 debug root=/dev/vda net.ifnames=0" \
                              -hda debian.aarch64/image.qcow2 -nographic \
                              -net nic -net tap,ifname=tap108,script="./qemu-brup",downscript="./qemu-brdown"
```

cat qemu-brup
```bash 
#! /bin/sh
# qemu-ifup script to bring a network (tap) device for qemu up

MAINIF=eth0
BR_IF=br0vm
BRIF_IP="10.192.192.254"

ip link add name $BR_IF type bridge
ip addr add $BRIF_IP/24 dev $BR_IF
ip link set $BR_IF up

tunctl -t $1
ip link set $1 up

brctl addif $BR_IF $1

iptables -t nat -A POSTROUTING -o $MAINIF -j MASQUERADE
iptables -A FORWARD -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT
iptables -A FORWARD -o $MAINIF -i br0vm -j ACCEPT
```

cat qemu-brdown
```bash 
#! /bin/sh
# qemu-tap-down script to execute just before the tap interface is removed.
# Normally this script is empty.
echo $0 removing tap interface $1
exit
```
