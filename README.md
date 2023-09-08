# Ansible Project - Homelab

## Section 1. Authorise SSH logins on cluster nodes

Copy locally available public keys to `~/.ssh/uthorized_keys` file on remote machines

```shell
$ ssh-copy-id pi@raspberrypi-node-1.local
```

## Section 2. Create BTRFS filesystem on top of LUKS encrypted partitions

Wipe empty disks with dm-crypt

```shell
$ cryptsetup open --type plain -d /dev/urandom /dev/<block-device> to_be_wiped
```

Wipe the containers with zeros

```shell
$ dd if=/dev/zero of=/dev/mapper/to_be_wiped status=progress bs=1M
```

Close the temporary containers after wiping

```shell
$ cryptsetup close to_be_wiped
```

Generate a keyfile of 2048 random bytes

```shell
$ dd bs=512 count=4 if=/dev/random of=/etc/cryptsetup-keys.d/<key-file> iflag=fullblock
```

Deny any access for other users than `root`

```shell
$ chmod 600 /etc/cryptsetup-keys.d/<key-file>
```

Set up encrypted volumes for dm-crypt and LUKS

```shell
$ cryptsetup luksFormat /dev/<block-device> /etc/cryptsetup-keys.d/<key-file>
```

Add a *passphrase* for key slot for the encrypted volumes

```shell
$ cryptsetup luksAddKey --key-file /etc/cryptsetup-keys.d/<key-file> -y /dev/<block-device>
```

Modify `/etc/crypttab` to automate unlocking and mounting on boot

```text
<volume-name>  UUID=<UUID>  /etc/cryptsetup-keys.d/<key-file>
```

Create btrfs filesystem on encrypted volumes

```shell
$ mkfs.btrfs -d raid1 -m raid1 /dev/mapper/<volume-name> /dev/mapper/<volume-name>
```

Modify `/etc/fstab` to automate mounting on boot

```text
UUID=<UUID>  /mnt  btrfs  defaults  2
```
