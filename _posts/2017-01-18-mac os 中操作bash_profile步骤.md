---
layout: post
title: mac os 中操作bash_profile步骤
categories: [Mac, ]
description: mac os 中操作bash_profile步骤
keywords: Mac, 
---

在 Windows 系统中，配置开发的相关环境变量时，需要去操作 其 PATH 进行环境变量的配置，那么，在 Mac 系统中，如何配置环境变量呢？这里就需要在 bash_profile 文件中进行操作。
mac os 中操作bash_profile步骤

#### 第一步：进入当前用户的home目录
在 Mac 上打开 Terminal 终端，并输入 `cd ~`
#### 第二步： 判断 .bash_profile 是否存在
使用 `touch .bash_profile` 创建  `.bash_profile` 文件， touch 命令会判断文件是否存在， 不存在则新建文件。
#### 第三步： 打开 .bash_profile 文件
使用 `open .bash_profile` 命令打开  .bash_profile 文件 ，并在弹出的 文件进行编辑

#### 更新 .bash_profile 文件
当 .bash_profile 文件更改完成后，保存退出。 使用 `source .bash_profile` 进行更新


![](/images/posts/tools/2017-01-18-01.png)


