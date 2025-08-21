+++
date = 2025-08-14T21:07:17Z
description = "开发Neovim插件"
draft = true
title = "Neovim 插件开发"

[extra]
toc = true

[taxonomies]
tags = ["Note"]
+++

## 环境

使用`LazyVim`作为Neovim的配置框架。

## 加载

在`LazyVim`中，插件的加载通常在`lua/plugins/`目录下进行。可以通过创建一个新的Lua文件来定义你的插件。
创建一个名为`music.lua`的文件，并在其中添加以下内容：

```lua
return {
  dir = "path/to/music.nvim", -- 本地仓库路径
  dev = true, -- 开发模式，支持热重载
  lazy = false, -- 是否延迟加载
  priority = 1000, -- 可选，确保先加载
  opts = {
    user = "user", -- 这里传入的表会作为 opts 给 setup
    passwd = "passwd",
  },
}
```

## 流程

插件目录结构通常如下：

```
music.nvim/
├── lua/
│   └── music/
│       ├── init.lua
│       └── other.lua
├── README.md
└── LICENSE
```

在`lua/music/init.lua`中，你可以定义插件的主要功能和配置。一般会返回一个表，有一个`setup`函数用于配置插件。

```lua
local M = {}
function M.setup(opts)
  -- 这里可以使用 opts 参数进行配置
  vim.notify("Music plugin loaded with user: " .. opts.user)
end
```
