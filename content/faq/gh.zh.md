+++
date = 2025-02-25T09:08:53Z
description = "Solve some common issues when using GitHub Pages"
draft = false
title = "GitHub Pages FAQ"

[taxonomies]
tags = ["GH","FAQ"]
+++

## Pemission denied

{{ img(src="/images/image-2.png") }}

[解决办法]('https://github.com/ad-m/github-push-action/issues/96#issuecomment-889984928')
`Settings` -> `Actions` -> `General` -> `Workflows permissions` -> `Read and write permissions`

## Browser shows insecure content warning

{{ img(src="/images/image-4.png") }}

[解决办法](https://stackoverflow.com/a/46672993)
修改你使用框架的配置文件，改`url`为你自定义的域名，而不是`github.io`域名。
