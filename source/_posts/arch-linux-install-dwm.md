---
title: Arch Linux 下安装 dwm
date: 2024-02-29 16:28:33
---

`dwm` 官网：[https://dwm.suckless.org/](https://dwm.suckless.org/)

## 安装 xorg + xorg-xinit + git + base-devel

----------

```bash
sudo pacman -S xorg xorg-xinit base-devel git
```

## 安装 yay

----------

`yay` 是可以安装 `AUR` 中软件包的工具。

```bash
git clone https://aur.archlinux.org/yay.git

cd yay

makepkg -si
```

如果在安装 `yay` 的过程中报以下错误，是因为 `GOPROXY` 不支持所在的地区。

```bash
go build -trimpath -mod=readonly -modcacherw  -ldflags '-X "main.yayVersion=10.2.2" -X "main.localePath=/usr/share/locale/" -linkmode=external' -buildmode=pie -o yay
go: github.com/Jguer/go-alpm/v2@v2.0.5: Get "https://proxy.golang.org/github.com/%21jguer/go-alpm/v2/@v/v2.0.5.mod": dial tcp 216.58.200.241:443: i/o timeout
make: *** [Makefile:112: yay] Error 1
==> ERROR: A failure occurred in build().
    Aborting...
```

设置 `GOPROXY` 以解决以上问题。

```bash
# 设置 GOPROXY
echo "export GOPROXY=https://goproxy.cn" >> ~/.profile

# 重新加载 .profile文件
source ~/.profile
```

## 安装字体

----------

```bash
sudo pacman -S ttf-firacode-nerd wqy-microhei
```

也可以安装自己喜欢的字体，但是需要 `nerd font` 字体。

## 安装终端模拟器

----------

### wezterm

`wezterm` 有 `GPU` 加速。

```bash
# 安装 wezterm
sudo pacman -S wezterm
```

`wezterm` 的配置文件为：`~/.config/wezterm/wezterm.lua`。

## 克隆 dwm 源码

----------

使用源码安装，每次 `修改源码` 都需要 `重新编译安装`。

```bash
git clone https://git.suckless.org/dwm
```

如果使用的终端模拟器不是 `st`，需要修改 `config.def.h` 文件中的以下代码：

```c
/* 将其中的 st 修改为安装的终端模拟器 */
static const char *termcmd[] = { "st", NULL};
```

修改 `config.mk` 文件。

```bash
# 修改 config.mk 文件
X11INC = /usr/X11R6/include               X11INC = /usr/include/X11
                                 ->
X11LIB = /usr/X11R6/lib                   X11LIB = /usr/lib/X11
```

## 编译安装 dwm

----------

```bash
# 进入 dwm 目录
cd dwm

# 编译安装
sudo make clean install
```

## 使用 startx 启动 dwm

----------

```bash
# ~/.xinitrc 文件本来是没有的，需要手动创建
vim ~/.xinitrc

# 在 .xinitrc 文件中添加以下代码
exec dwm
```

重启后，使用 `startx` 命令启动 `dwm`。

```bash
startx
```

启动后，可以看到 `dwm` 最原始的界面。

默认的 `MODKEY` 是 `Alt` 键。

`MODKEY` 键是可改变的，`Mod1Mask` 表示 `Alt` 键，`Mod4Mask` 表示 `window` 键。

按下 `MODKEY` + `bashift` + `Enter` 启动终端。

![dwm](https://github.com/gin-18/pictures/blob/master/blog/arch-linux-install-dwm/dwm.png?raw=true)

将以下代码添加到 `bashell` 的配置文件中，可以在登录后自动启动 `startx`。

```bash
# auto startx
if [ -z "${DISPLAY}" ] && [ "${XDG_VTNR}" -eq 1 ]; then
  exec startx
fi
```

## dwm 配置

----------

每次编译安装都会根据 `config.def.h` 文件产生 `config.h` 文件。

所以可以通过 `config.def.h` 文件配置 `dwm`。

每次修改源码都需要重新编译安装：

```bash
rm -rf config.h && sudo make clean install
```

### config.def.h 文件源码说明

#### 外观配置

修改以下代码以配置 `dwm` 的 `字体`，`颜色` 等。

![dwm-appearance](https://github.com/gin-18/pictures/blob/master/blog/arch-linux-install-dwm/dwm-appearance.png?raw=true)

例：

将默认的 `蓝色配色` 修改为 `紫色配置`。

![dwm-appearance-color-purple](https://github.com/gin-18/pictures/blob/master/blog/arch-linux-install-dwm/dwm-appearance-color-purple.png?raw=true)

![dwm-appearance-purple](https://github.com/gin-18/pictures/blob/master/blog/arch-linux-install-dwm/dwm-appearance-purple.png?raw=true)

#### 标签栏的显示 + 软件的窗口规则

修改以下代码以配置 `dwm` 标签栏的显示效果和软件的窗口规则。

窗口规则：`软件打开的标签`，`窗口是否浮动` 等。

![dwm-tagging-win-rule](https://github.com/gin-18/pictures/blob/master/blog/arch-linux-install-dwm/dwm-tagging-win-rule.png?raw=true)

#### 添加命令 + 定义快捷键

修改以下代码以配置 `dwm` 的程序命令和快捷键。

![dwm-commands-keys](https://github.com/gin-18/pictures/blob/master/blog/arch-linux-install-dwm/dwm-commands-keys.png?raw=true)

例：

添加 `flameshot` 截图命令和快捷键。

```c
/* 添加命令 */
static const char *flameshot[] = { "flameshot", "gui", NULL};

/* 添加快捷键：mod+s */
{ MODKEY,       XK_s,      spawn,        {.v = flameshot } },
```

## dwm 补丁

----------

通过 `补丁` 为 `dwm` 添加额外的功能。

补丁下载地址：[https://dwm.suckless.org/patches/](https://dwm.suckless.org/patches/)

### 打补丁

一般情况下，补丁文件都会说明是基于哪个版本制作出来的补丁。

例：

`hide vacant tags` 最新的补丁是基于 `dwm-6.2` 制作出来的补丁。

![hide-vacant-tags](https://github.com/gin-18/pictures/blob/master/blog/arch-linux-install-dwm/hide-vacant-tags.png?raw=true)

#### 使用 git 管理打补丁后的 dwm

在打补丁之前，可以创建一个指向该补丁版本的git分支。

```bash
# 创建版本指向6.2，分支命为dwm-hide_vacant_tags-6.2的git分支
git switch -c dwm-hide_vacant_tags-6.2 6.2
```

将补丁文件放在 `~/patches` 目录下，通过 `patch` 命令打补丁。

```bash
# 在 dwm-hide_vacant_tags-6.2 分支上打 dwm-hide_vacant_tags-6.2.diff 补丁
patch < ~/patches/dwm-hide_vacant_tags-6.2.diff

# git 提交修改
git commit -am "patch: dwm-hide_vacant_tags-6.2"
```

将 `dwm-hide_vacant_tags-6.2` 分支合并到 `master` 分支。

分支合并如果有冲突就解决冲突。

```bash
# 切换到 master 分支
git switch master

# 合并分支
git merge dwm-hide_vacant_tags-6.2
```

重新编译安装。

```bash
rm -rf config.h && sudo make clean install
```

安装完成后，发现只有 `标签1` 下有程序运行，所以只显示 `标签1`，说明补丁成功了。

![dwm-hide-vacant-tags](https://github.com/gin-18/pictures/blob/master/blog/arch-linux-install-dwm/dwm-patch-hide-vacant-tags.png?raw=true)

### 代码托管和官方代码同步

查看远程仓库地址

```bash
git remote -v
```

![origin-to-suckless](https://github.com/gin-18/pictures/blob/master/blog/arch-linux-install-dwm/origin-to-suckless.png?raw=true)

发现 `origin` 指向官方的仓库。

修改：将 `upstream` 指向官方的仓库，`origin` 指向自己的仓库。

```bash
# 将 upstream 指向官方的仓库
git remote rename origin upstream

# 添加自己的仓库地址
git add remote origin <自己的仓库地址>
```

将配置好的 `dwm` 推送到自己的远程仓库。

```bash
# 推送所有的分支和 tags
git push origin --all && git push origin --tags
```

官方代码同步。

```bash
git pull upstream master
```

如果 `git pull` 有冲突就解决冲突。

### 推荐补丁

```
透明补丁：alphasystray.diff

临时小窗口：dwm-scratchpad-6.2.diff

隐藏空标签：dwm-hide_vacant_tags-6.2.diff

窗口间距：dwm-vanitygaps-20190508-6.2.diff

自动启动脚本：dwm-autostart-20161205-bb3bd6f.diff

窗口全屏：dwm-actualfullscreen-20191112-cb3f58a.diff

状态栏显示多个窗口信息：dwm-awesomebar-20191003-80e2a76.diff
```

## 状态栏右侧显示信息

----------

使用 `xsetroot` 命令可以在 `dwm` 状态栏的右侧显示需要的信息。

```bash
# 安装 xsetroot
sudo pacman -S xsetroot

# 使用 xsetroot 命令在状态栏右侧显示 "hello dwm"
xsetroot -name "hello dwm"
```

![dwm-xsetroot](https://github.com/gin-18/pictures/blob/master/blog/arch-linux-install-dwm/dwm-xsetroot.png?raw=true)

配合 `dwm-autostart-20161205-bb3bd6f.diff` 补丁和 `自定义脚本` 可以在系统启动时显示需要的系统信息。

例：

系统启动时，在状态栏右侧显示系统时间。

打上 `dwm-autostart-20161205-bb3bd6f.diff` 后 ，在 `dwm.c` 文件中找到以下代码：

```c
void
runAutostart(void) {
    system("cd ~/scripts; ./autostart_blocking.sh")
    system("cd ~/scripts; ./autostart.sh &")
}
```

可以修改为 `自定义脚本` 目录。

```c
void
runAutostart(void) {
    system("cd ~/dwm/scripts; ./autostart.sh &")
}
```

`autostart.sh` 脚本的内容如下：

```bash
#!/bin/sh

dwm_date () {
    date '+%Y年%m月%d日 %a %H:%M'
}

while true
do
    xsetroot -name "$(dwm_date)"
    sleep 1
done
```

![dwm-date](https://github.com/gin-18/pictures/blob/master/blog/arch-linux-install-dwm/dwm-date-script.png?raw=true)
