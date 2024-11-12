---
title: "Go Template"
description: 
date: 2024-10-20T12:32:50+08:00
image: 
math: 
license: 
hidden: false
comments: true
draft: true
---
# Go Template FAQ
这里只介绍我曾经探索过的一些用法，更多教学内容，请参考[官方文档](https://pkg.go.dev/text/template)。

由于我是在使用`chezmoi`时，才开始接触`Go Template`的，所以这里的例子都是文本模板的用法。

## 1. 去除空行
直接在`{{`和`}}`之间加上`-`，或者写在一行，如下：
```go
{{- if .Name -}}
{{- .Name -}}
{{- end -}}
```
```
{{ if .Name }}{{ .Name }}{{ end }}
```

## 2. 函数输入参数
```go
{{- $name := "world" -}}
{{- $name := .Name -}}
{{- $name := printf "%s %s" .FirstName .LastName -}}
```