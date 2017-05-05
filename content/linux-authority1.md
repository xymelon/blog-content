Title: Linux学习笔记——权限管理(1)
Date: 2013-11-22 20:01
Category: Technology
Tags: Linux
Slug: linux-authority1
Author: xymelon

**权限对于文件**

1. r(read):可读，如读取文本内容
1. w(write):可编辑、新增、修改该文件的内容（只针对内容，不能删除文件）
1. x(execute):可被系统执行

<!-- PELICAN_END_SUMMARY -->

**权限对于目录**

1. r(read contents in directory):可查询该目录下包含哪些文件
1. w(modify contents of directory):可对该目录下的子目录和文件进行增、删、更名、移位
1. x(access directory):可进入该目录
 
**经典范例**

假设有个帐号名为cow，它对目录/home/cow/具有rwx权限。若在此目录下有个test的文件，该文件权限如下：
	 
	rwx------ 1 root  root  4365 Sep 19 23:20 test

请问cow对此文件权限为什么？可否删除此文件？

解答：
>如题，cow对此文件来说是[others]身份，因此对于该文件它没有任何权限。但是该文件位于其home目录下，它对于home目录具有rwx完整权限，因此对于test这个文件来说，它是可以删除的。
 
**默认权限**

- 文件：-rw-rw-rw-（666）
- 目录：drwxrwxrwx(777)

*umask指令数字显示时代表【该默认值需要减掉的权限】*

>例：umask数值为022

>建立文件时默认权限：（-rw-rw-rw-）-（-----w--w-）= (-rw-r--r--)

>建立目录时默认权限：（drwxrwxrwx）(d----w--w-) = (drwxr-xr-x)
