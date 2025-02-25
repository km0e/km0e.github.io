+++
date = 2025-02-25T09:08:53Z
description = "How to remove a submodule from a git repository"
draft = false
title = "git rm submodule"

[taxonomies]
tags = ["Git"]
+++
Original source: [Stackoverflow](https://stackoverflow.com/a/16162000)

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
