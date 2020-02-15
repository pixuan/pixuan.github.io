---
layout: post
title:  "Vmware ubuntu share folder on mac"
date:   2020-02-16 01:00:00 +0800
categories: vmware ubuntu shareFolder
---

### mac上的vmware，ubuntu设置共享文件夹
* vmware设置中添加共享文件夹
* /mnt中有hgfs但没共享文件的方法  
<https://www.cnblogs.com/jiu0821/p/5946062.html>  
`注意：此处不能添加随机启动后自动挂载，否则会无法启动，进入emergency mode`
`所以：只能每次进行挂载，并检测是否已经挂载`
* fix: Welcome to emergency mode  
<https://blog.csdn.net/weixin_38705903/article/details/87890393>
<https://blog.csdn.net/xiangwanpeng/article/details/78083075>
* 检查是否已经挂载/mnt/hgfs目录  
<https://www.cnblogs.com/cheyunhua/p/11377599.html>
* LINUX nautilus 命令行命令打开指定文件夹  
<https://blog.csdn.net/weixin_41123761/article/details/78789332>
* 双击运行.sh文件  
<https://www.cnblogs.com/shuo1208/p/6744876.html>  
<https://www.cnblogs.com/EasonJim/p/7467457.html>
* Called "net usershare info" but it failed: Failed to execute child process “net”  
<https://blog.csdn.net/discoverer100/article/details/103052692>
* nonempty问题  
<https://blog.csdn.net/u011770788/article/details/45823793?utm_source=itdadao&utm_medium=referral>


shell脚本:
```sh
#!/bin/bash
if grep -qs '/mnt/hgfs' /proc/mounts; then
echo "/mnt/hgfs is mounted and open";
echo "${password}"|sudo -S nautilus /mnt/hgfs
else
echo "/mnt/hgfs is not mounted. to mount it then open"
echo "${password}"|sudo -S vmhgfs-fuse .host:/ /mnt/hgfs -o nonempty
echo "${password}"|sudo -S nautilus /mnt/hgfs
fi
```
