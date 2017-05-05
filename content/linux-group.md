Title: Linux学习笔记——有效群组（effective group）与初始群组（initial group）
Date: 2013-12-18 16:09
Category: Technology
Tags: Linux
Slug: linux-group
Author: xymelon

**有效群组（effective group）与初始群组（initial group）**

初始群组：/etc/passwd中第四栏所标志的GID；当用户登入系统，立刻拥有该群组的相关权限。

有效群组：当以某个用户身份登入后，输入`groups`命令第一出现的群组即为有效群组。它的作用是当用户建立新文件或目录时，所属群组为有效群组。

<!-- PELICAN_END_SUMMARY -->

**/etc/group档案结构**

    root:x:0:root

每一行代表一个群组，用`：`作为字段分隔符，共分为4部分。

> 1. 组名
> 1. 群组密码
> 1. GID
> 1. 该群组所支持的帐号名称。例，若cow帐号想加入root群组，即为`root:x:0:root,cow`

**切换有效群组**

使用命令`newgrp`切换，不过所切换的群组必须为该用户所加入的群组。`newgrp`是以另一个shell来提供群组切换功能的，如果想回到原本的环境中，可输入`exit`命令。