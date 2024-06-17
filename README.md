# Ansible Project - Homelab

## Section 1. Flash Raspberry Pi OS

Using `rpi-imager`, flash Raspberry Pi OS Lite (64bit) to the SD cards

## Section 2. Configure and build a Linux kernel with Target Core Mod (TCM) support

Install Git and the build dependencies

```shell
sudo apt install git bc bison flex libssl-dev libncurses5-dev make
```

Get the Linux kernel sources

```shell
git clone --depth=1 https://github.com/raspberrypi/linux
```

Prepare the default configuration for Raspberry Pi Compute Module 4

```shell
cd linux
KERNEL=kernel8
make bcm2711_defconfig
```

Configuring the kernel using `menuconfig`

```shell
make menuconfig

# Enabel CONFIG_TARGET_CORE
```

Build and install the kernel, modules, and Device Tree blobs

```shell
make -j4 Image.gz modules dtbs
sudo make modules_install
sudo cp arch/arm64/boot/dts/broadcom/*.dtb /boot/firmware/
sudo cp arch/arm64/boot/dts/overlays/*.dtb* /boot/firmware/overlays/
sudo cp arch/arm64/boot/dts/overlays/README /boot/firmware/overlays/
sudo cp arch/arm64/boot/Image.gz /boot/firmware/$KERNEL.img
```

## Section 3. Runs the Ansible playbooks

Set up the system packages

```shell
ansible-playbook playbooks/site.yml
```

Create a K3s cluster

```shell
ansible-playbook --ask-vault-pass playbooks/k3s/install.yml
```

## Section 4. Create a ZFS pool

Create a mirrored ZFS pool mounted at `/data`

```shell
sudo zpool create -f -o ashift=12 -m /data tank mirror <id> <id>
```

Configure the default checksum algorithm to blake3

```shell
sudo zfs set checksum=blake3 tank
```

Create datasets under the zpool to accommodate Kubernetes persistent data

```shell
sudo zfs create tank/k8s
sudo zfs create tank/k8s/main
sudo zfs create tank/k8s/main-snapshots
```
