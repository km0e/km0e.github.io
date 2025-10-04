+++
date = 2025-09-22T13:00:50Z
description = "安装和配置Niri"
draft = false
title = "Niri"

[extra]
toc = true

[taxonomies]
tags = ["Note"]
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

安装中文字体。

```bash
paru -S noto-fonts-cjk
```

然后运行以下命令更新字体缓存：

```bash
fc-cache -fv
```

退出`Niri`重启后，字体即可生效。

### 输入法

安装`fcitx5-im`和`fcitx5-chinese-addons`：

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

修改`Niri`的配置文件，添加以下内容以启用`Wayland`与`IME`支持：

```kdl
environment {
    CHROME_FLAGS "--enable-features=UseOzonePlatform --ozone-platform=wayland --enable-wayland-ime --wayland-text-input-version=3"
}
```

## 进阶

使用一些已经配置好的`Niri`环境，这里选择`DankMaterialShell`：

安装软件包：

```bash
paru -S dms-shell-git matugen-bin
```

安装字体

```bash
mkdir -p ~/.local/share/fonts &&
curl -L "https://github.com/google/material-design-icons/raw/master/variablefont/MaterialSymbolsRounded%5BFILL%2CGRAD%2Copsz%2Cwght%5D.ttf" -o ~/.local/share/fonts/MaterialSymbolsRounded.ttf
curl -L "https://github.com/rsms/inter/raw/refs/tags/v4.1/docs/font-files/InterVariable.ttf" -o ~/.local/share/fonts/InterVariable.ttf
curl -L "https://github.com/tonsky/FiraCode/releases/latest/download/FiraCode-Regular.ttf" -o ~/.local/share/fonts/FiraCode-Regular.ttf
fc-cache -fv
```

克隆配置

```bash
mkdir ~/.config/quickshell && git clone --depth 1 https://github.com/AvengeMedia/DankMaterialShell.git ~/.config/quickshell/dms
```

安装最新`dms`：

```bash
sudo sh -c "curl -L https://github.com/AvengeMedia/danklinux/releases/latest/download/dms-$(uname -m | sed 's/x86_64/amd64/;s/aarch64/arm64/').gz | gunzip | tee /usr/local/bin/dms > /dev/null && chmod +x /usr/local/bin/dms"
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
