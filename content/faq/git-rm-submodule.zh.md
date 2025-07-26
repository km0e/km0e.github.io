+++
date = 2025-02-25T09:14:59Z
description = "如何移除`Git`仓库中的子模块"
draft = false
title = "移除Git子模块"

[taxonomies]
tags = ["Git","FAQ"]
+++

原文: [Stackoverflow](https://stackoverflow.com/a/16162000)

```bash
mv a/submodule a/submodule_tmp

git submodule deinit -f -- a/submodule
rm -rf .git/modules/a/submodule
git rm -f a/submodule
# Note: a/submodule (no trailing slash)

# or, if you want to leave it in your working tree and have done step 0
git rm --cached a/submodule
mv a/submodule_tmp a/submodule
```
