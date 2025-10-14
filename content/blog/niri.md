+++
date = 2025-09-22T13:00:50Z
updated = 2025-10-13T23:00:39Z
description = "安装和配置Niri"
draft = false
title = "Niri"

[extra]
toc = true

[taxonomies]
tags = ["Note","DE","Wayland"]
+++

## 环境

在`Arch Linux`下使用`Niri`，并通过`paru`进行安装。

```bash
NAME="Arch Linux"
PRETTY_NAME="Arch Linux"
ID=arch
BUILD_ID=rolling
ANSI_COLOR="38;2;23;147;209"
HOME_URL="https://archlinux.org/"
DOCUMENTATION_URL="https://wiki.archlinux.org/"
SUPPORT_URL="https://bbs.archlinux.org/"
BUG_REPORT_URL="https://gitlab.archlinux.org/groups/archlinux/-/issues"
PRIVACY_POLICY_URL="https://terms.archlinux.org/docs/privacy-policy/"
LOGO=archlinux-logo
```

## 安装

首先安装`paru`，如果已经安装可以跳过这一步：

```bash
sudo pacman -S --needed rustup base-devel git
rustup default stable
git clone --depth 1 https://aur.archlinux.org/paru.git /tmp/paru
cd /tmp/paru
makepkg -si
```

然后使用`paru`安装`Niri`：

```bash
paru -S niri
```

## 配置

`Niri`的配置文件位于`~/.config/niri/config.kdl`。

### Font

安装中文与表情字体：

```bash
paru -S noto-fonts-cjk noto-fonts-emoji
```

然后运行以下命令更新字体缓存：

```bash
fc-cache -fv
```

退出`Niri`重启后，字体即可生效。

### 输入法

安装`fcitx5-im`和`fcitx5-chinese-addons`：

注意：这里会让你选音频提供者，如果担心音频问题，可以选择`pipewire`。

```bash
paru -S fcitx5-im fcitx5-chinese-addons
```

修改`Niri`的配置文件，添加以下内容：

```kdl
environment {
    XMODIFIERS "@im=fcitx"
}

spawn-at-startup "fcitx5"
```

### Chrome

安装`Chrome`浏览器：

```bash
paru -S google-chrome
```

添加以下内容到`~/.config/chrome-flags.conf`文件中：

```text
--enable-features=AcceleratedVideoDecodeLinuxGL
--enable-features=UseOzonePlatform
--ozone-platform=wayland
--enable-wayland-ime
--wayland-text-input-version=3
```

- 启用硬件加速
- 启用`Wayland`支持
- 启用`Wayland`输入法支持
- 使用`Wayland`文本输入版本3

## 进阶


### DankMaterialShell

本节参考[GitHub - DankMaterialShell](https://github.com/AvengeMedia/DankMaterialShell?tab=readme-ov-file#dank-shell-installation)

使用一些已经配置好的`Niri`环境，这里选择`DankMaterialShell`：

安装软件包：

```bash
paru -S dms-shell-git matugen-bin cava wl-clipboard cliphist brightnessctl dms-shell-niri
```

修改`Niri`的配置文件，添加以下内容：

```kdl
// Required for clipboard history integration
spawn-at-startup "bash" "-c" "wl-paste --watch cliphist store &"

// Recommended (must install polkit-mate before hand) for elevation prompts
spawn-at-startup "/usr/lib/mate-polkit/polkit-mate-authentication-agent-1"
// This may be a different path on different distributions, the above is for the arch linux mate-polkit package

// Starts DankShell
spawn-at-startup "dms" "run"

// If using niri newer than 271534e115e5915231c99df287bbfe396185924d (~aug 17 2025)
// you can add this to disable built in config load errors since dank shell provides this
config-notification {
    disable-failed
}

// Dank keybinds
// 1. These should not be in conflict with any pre-existing keybindings
// 2. You need to merge them with your existing config if you want to use these
// 3. You can change the keys to whatever you want, if you prefer something different
// 4. For the increment/decrement ones you can change the steps to whatever you like too
binds {
   Mod+Space hotkey-overlay-title="Application Launcher" {
      spawn "dms" "ipc" "call" "spotlight" "toggle";
   }
   Mod+V hotkey-overlay-title="Clipboard Manager" {
      spawn "dms" "ipc" "call" "clipboard" "toggle";
   }
   Mod+M hotkey-overlay-title="Task Manager" {
      spawn "dms" "ipc" "call" "processlist" "toggle";
   }
   Mod+N hotkey-overlay-title="Notification Center" {
      spawn "dms" "ipc" "call" "notifications" "toggle";
   }
   Mod+Comma hotkey-overlay-title="Settings" {
      spawn "dms" "ipc" "call" "settings" "toggle";
   }
   Mod+P hotkey-overlay-title="Notepad" {
      spawn "dms" "ipc" "call" "notepad" "toggle";
   }
   Super+Alt+L hotkey-overlay-title="Lock Screen" {
      spawn "dms" "ipc" "call" "lock" "lock";
   }
   Mod+X hotkey-overlay-title="Power Menu" {
      spawn "dms" "ipc" "call" "powermenu" "toggle";
   }
   XF86AudioRaiseVolume allow-when-locked=true {
      spawn "dms" "ipc" "call" "audio" "increment" "3";
   }
   XF86AudioLowerVolume allow-when-locked=true {
      spawn "dms" "ipc" "call" "audio" "decrement" "3";
   }
   XF86AudioMute allow-when-locked=true {
      spawn "dms" "ipc" "call" "audio" "mute";
   }
   XF86AudioMicMute allow-when-locked=true {
      spawn "dms" "ipc" "call" "audio" "micmute";
   }
   XF86MonBrightnessUp allow-when-locked=true {
      spawn "dms" "ipc" "call" "brightness" "increment" "5" "";
   }
   // You can override the default device for e.g. keyboards by adding the device name to the last param
   XF86MonBrightnessDown allow-when-locked=true {
      spawn "dms" "ipc" "call" "brightness" "decrement" "5" "";
   }
   // Night mode toggle
   Mod+Shift+N allow-when-locked=true {
      spawn "dms" "ipc" "call" "night" "toggle";
   }
}
```

注意：`binds`部分需要和已有的配置合并。

### Display Manager

自动登录`Niri`，这里使用`greetd`或`lemurs`。

#### greetd

本节参考[DankMaterialShell - Greeter](https://github.com/AvengeMedia/DankMaterialShell?tab=readme-ov-file#greeter)

`dms`有对`greetd`的支持，直接执行以下命令：

```bash
dms greeter install
```

不支持的话，可以参考[Arch Wiki - Greetd](https://wiki.archlinux.org/title/Greetd)进行安装与配置。


#### lemurs

本节参考[GitHub - lemurs](https://github.com/coastalwhite/lemurs?tab=readme-ov-file#arch-linux)

##### 总览

```bash
paru -S lemurs

echo "#!/bin/sh
exec niri-session" | sudo tee /etc/lemurs/wayland/niri

# Not needed if you don't have a window manager yet
# sudo systemctl disable display-manager.service

sudo systemctl enable lemurs.service
```

#### 安装

```bash
paru -S lemurs
```

#### 配置

创建`/etc/lemurs/wayland/niri`文件：

```bash
echo "#!/bin/sh
exec niri-session" | sudo tee /etc/lemurs/wayland/niri
```

启用`lemurs`服务：

```bash
sudo systemctl enable lemurs.service
```

如果你已经有一个显示管理器在运行，可能需要先禁用它：

```bash
sudo systemctl disable display-manager.service
```


## FAQ

### Nvidia驱动

参考：[Arch Wiki - NVIDIA](https://wiki.archlinux.org/title/NVIDIA)

### 笔记本自带音频设备不能识别

安装`sof-firmware`与`pipewire-pulse`：

```bash
paru -S sof-firmware pipewire-pulse
```
