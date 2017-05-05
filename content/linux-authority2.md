Title: Linux学习笔记——权限管理(2)
Date: 2013-11-23 23:10
Category: Technology
Tags: Linux
Slug: linux-authority2
Author: xymelon

**特殊文件权限：SUID,SGID,SBIT**

**SUID**：当s这个标志出现在**文件拥有者**权限的x权限上时，如【-rwsr-xr-x】,此时被称为Set UID,简称为SUID的特殊权限，且只对文件有效。

1. SUID权限仅对二进制程序（binary program）有效；
1. 执行者对于该程序需要具有x的可执行权限；
1. 本权限尽在执行该程序的过程中有效（run-time）；
1. 执行者将具有该拥有者（owner）的权限。
 
<!-- PELICAN_END_SUMMARY -->

简单的说就是具有x权限的执行者（groups/others）在执行过程中会具有拥有者(owner)的权限。
 
**SGID**：当s这个标志出现在**群组权限**的x权限上时，如【-rwx--s--x】,此时被称为Set GID,简称为SGID的特殊权限，可对文件和目录有效。对目录设定SGID权限后，具有如下功能：

1. 用户对于此目录具有r与x的权限时，该用户能够进入此目录；
1. 用户在此目录下的有效群组（effective group）将会变成该目录的群组；
1. 用途：若用户在此目录下具有w的权限（可新建文件），则使用者所建立的新文件的群组与此目录的群组相同。
 
**SBIT(Sticky Bit)**：当t这个标志出现在**其他权限**的x权限上时，如【drwxrwxrwt】，只对目录有效，对于目录作用如下：

1. 当用户对于此目录具有w,x的权限，亦即具有写入的权限时；
1. 当用户在该目录下建立文件和目录时，仅有自己与root才有权力删除该文件。
