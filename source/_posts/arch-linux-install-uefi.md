---
title: UEFI 下安装 Arch Linux
date: 2024-02-28 14:10:50
---

## 禁用安全启动

---

`Arch Linux` 安装镜像不支持 `安全启动（Secure Boot）`。要引导安装介质，需要禁用安全启动。

## 验证引导模式

---

要验证引导模式，可以使用下列命令行出 `efivars` 目录：

```sh
ls /sys/firmware/efi/efivars
```

如果命令结果显示了目录且没有报告错误，则系统以 `UEFI` 模式引导。 如果目录不存在，则系统可能以 `BIOS` 模式引导。

## 连接互联网

---

安装 `Arch Linux` 必须连通网络。

可以插入 `网线` 或使用 `wifi`。

### 连接 wifi

---

使用 `iwd` 连接 `wifi`。

```sh
# 进入iwd交互界面
iwctl

# 查看设备名
device list

# 扫描网络
station <设备名> scan

# 查看网络名称
station <设备名> get-networks

# 连接网络
station <设备名> connect <网络名称>
```

### 检查网络连接

---

```sh
ping archlinux.org
```

## 更新系统时钟

---

```sh
timedatectl status
```

## 磁盘分区

---

可以使用 `fdisk` 命令进行磁盘分区，也可以使用 `cfdisk` 命令进行磁盘分区。

`cfdisk` 命令拥有交互界面。

```sh
cfdisk
```

### 分区示列

---

| 挂载点           | 分区类型              | 建议大小        |
| ---------------- | ---------------       | --------------- |
| /mnt/boot        | EFT系统分区           | 至少300Mib      |
| [SWAP]           | Linux swap            | 大于512Mib      |
| /mnt             | Linux x86-64根目录(/) | 剩余空间        |

## 格式化分区

---

查看分区设备名。

```sh
fdisk -l
```

格式化 `boot分区`。

```sh
mkfs.fat -F 32 <boot分区设备名>
```

格式化 `根分区`。

```sh
mkfs.ext4 <根分区设备名>
```

格式化 `swap` 分区。

```sh
mkswap <swap分区设备名>
```

## 挂载分区

---

将 `根分区` 挂载到 `/mnt目录`。

```sh
mount <根分区设备名> /mnt
```

将 `boot分区` 挂载到 `/mnt/boot目录`。

```sh
mount --mkdir <boot分区设备名> /mnt/boot
```

激活 `swap` 分区。

```sh
swapon <swap分区设备名>
```

## 修改镜像列表

---

使用 `reflector` 来获取速度最快的6个镜像，并将地址保存到 `/etc/pacman.d/mirrorlist` 文件中。

```sh
reflector -c China -a 6  --sort rate --save /etc/pacman.d/mirrorlist
```

## 安装系统

---

安装系统。

```sh
pacstrap -K /mnt base linux linux-firmware
```

生成 `fstab` 文件。

```sh
genfstab -U /mnt >> /mnt/etc/fstab
```

## 进入系统

---

```sh
arch-chroot /mnt
```

## 进入系统后的设置

---

### 修改时区

---

```sh
ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
```

### 同步系统时间

---

```sh
hwclock --systohc
```

### 本地化设置

---

进入系统后没有编辑器，下载需要的编辑器。

```sh
pacman -S neovim
```

修改 `/etc/locale.gen文件` ，去掉 `en_US.UTF-8 UTF-8` 此行的注释。

```sh
nvim /etc/locale.gen
```

再执行 `locale-gen`。

```sh
locale-gen
```

创建 `/etc/locale.conf` 文件，并在 `/etc/locale.conf` 文件中添加 `LANG=en_US.UTF-8`。

```sh
# 创建并编辑/etc/locale.conf文件
nvim /etc/locale.conf

# 在此文件中添加以下内容
LANG=en_US.UTF-8
```

### 网络设置

---

创建 `/etc/hostname` 文件，在文件中添加自己的 `主机名`。

```sh
# 创建并编辑/etc/hostname文件
nvim /etc/hostname

# 在此文件中添加自己的主机名
arch-test
```

添加 `hosts`，在 `/etc/hosts` 文件中添加以下内容：

```sh
127.0.0.1     localhost

::1           localhost

127.0.1.1     主机名.localdomain 主机名
```

### 给root用户添加密码

---

```sh
passwd
```

### 安装dhcpcd和iwd

---

`dhcpcd` 用于新系统动态分配 `ip地址`。

`iwd` 用于新系统连接 `wifi`。

```sh
pacman -S dhcpcd iwd
```

## 安装grub引导

---

安装相应软件包。

```sh
pacman -S grub efibootmgr
```

`grub` 安装。

```sh
grub-install --target=x86_64-efi --efi-directory=/boot
```

生成 `grub` 的配置文件。

```sh
grub-mkconfig -o /boot/grub/grub.cfg
```

根据 `cpu` 选择安装 `intel-ucode` 或 `amd-ucode`。

```sh
pacman -S intel-ucode

或

pacman -S amd-ucode
```

`双系统` 需要额外安装 `os-prober`。

```sh
pacman -S os-prober
```

## 退出系统，取消挂载并重启系统

---

```sh
# 退出系统
exit

# 取消挂载
umount -R /mnt

# 重启系统
reboot
```

## 重启进入系统后的设置

---

启动 `dhcpcd`。

```sh
systemctl start dhcpcd

systemctl enable dhcpcd
```

启动 `iwd`。

```sh
systemctl start iwd

systemctl enable iwd
```

更新系统。

```sh
pacman -Syyu
```

安装基础软件包。

```sh
pacman -S base-devel
```

添加普通用户。

```sh
useradd -mG wheel <用户名>
```

给新用户设置密码。

```sh
passwd <用户名>
```

修改 `/etc/sudoers` 文件。

```sh
nvim /etc/sudoers
```

在 `/etc/sudoers` 文件中放开以下代码的注释就可以使用 `sudo` 命令。

```sh
# 放开此行的注释 ↓
%wheel ALL=(ALL) ALL
```
