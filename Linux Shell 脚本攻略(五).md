---
title: Linux Shell 脚本攻略(五)
tags: Shell,脚本,网站
grammar_cjkRuby: true
---

[TOC]

##  网站下载
`wget` 是一个文件下载的命令行工具，选项多且用法活
```bash?linenums
# wget 下载网页或远程文件
$ wget URL
```

##  格式化纯文本形式下载网页
```bash?linenums
# 用lynx命令 -dump选项将网页ASII字符形式下载
$ lynx -dump URL > web.txt
```

##  `cURL` 入门
`cURL` 支持包括 `HTTP`、`HTTPS`、`FTP` 在内的众多协议，还支持 `POST`、`cookie`、认证、限速、文件大小限制、进度条等特性。
`cURL` 通常下载文件输出到 `stdout`，将进度信息输出到 `stderr`。`--silent` 选项不显示进度的信息。
```bash?linenums
$ curl URL --silent

# -O用来下载数据写入文件，而非写入标准输出
$ curl URL --silent -o new_filename
```

**断点续传**
```bash?linenums
$ curl URL/file -c offset
```
**用 `cURL` 设置 `cookie`**
```bash?linenums
$ curl URL --cookie "user=slynux;pass=hack"
```




















