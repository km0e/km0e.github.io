+++
date = 2025-02-25T09:08:53Z
description = "Solve the problem of keeping redirects after cloudflare proxy"
draft = false
title = "Redirects after Cloudflare Proxy"

[taxonomies]
tags = ["GH","FAQ"]
+++
`Github Pages` 启用了`Enforce HTTPS`后，`Cloudflare`代理后尝试使用`HTTP`访问`Github Pages`时，会触发重定向，然后又去访问`Cloudflare`，导致一直重定向。
解决办法：将`Cloudflare`的`SSL/TLS`设置模式改为`Full`。

{{ img(src="/images/image-3.png") }}
