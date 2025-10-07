+++
date = 2025-10-07T16:20:45Z
update-date = 2025-10-07T16:20:45Z
description = "ACME 从 Google Domains 获取通配符域名证书"
draft = false
title = "通配符域名证书"

[extra]
toc = true

[taxonomies]
tags = ["Note"]
+++


## 获取Google云服务的b64MacKey和KeyID

### 获取Google云服务的b64MacKey和KeyID
打开google云服务，选择API和服务

{{ img(src="/images/image-5.png") }}

点击右上角的Shell

{{ img(src="/images/image-6.png") }}

成功后输入以下命令
```
gcloud publicca external-account-keys create
```
得到b64MacKey和KeyID
### 使用acme.sh申请证书
安装acme.sh到～/.acme.sh
```
curl https://get.acme.sh | sh -s
```
注册账号
```
acme.sh --register-account \
        --server https://dv.acme-v02.api.pki.goog/directory \
        --server google -m [Your Mail]] \
        --eab-kid [Your keyId] \
        --eab-hmac-key [Your b64MacKey]
```
由于域名托管在Cloudflare，所以需要添加环境变量
首先在Cloudflare上拿到Global API Key

{{ img(src="/images/image-7.png") }}

在account.conf中添加
```
CF_Key="Global API Key"
CF_Email="Your Mail"
```
申请证书
```
acme.sh --server google --issue --dns dns_cf -d [Your Domain]
```

可以设置默认证书的CA
```
acme.sh --set-default-ca  --server google
```

### 更新证书

```bash
acme.sh --renew -d [Your Domain] --force
```


