---
layout: article
title: ubuntu1804 设置开机启动脚本
mode: immersive
tag: linux
header:
  theme: dark
  background: 'linear-gradient(135deg, rgb(34, 139, 87), rgb(139, 34, 139))'
article_header:
  type: overlay
  theme: dark
  background_color: '#203028'
  background_image:
    gradient: 'linear-gradient(135deg, rgba(34, 139, 87 , .4), rgba(139, 34, 139, .4))'
    src: /assets/images/img/xiongmao1.png
---
<!-- ![image](/assets/images/your-image.jpg) -->
## 在Ubuntu 1804 中使用/etc/rc.local

### 1、systemd默认读取/etc/systemd/system下的配置文件，该目录下的文件会链接/lib/systemd/system/下的文件。一般系统安装完/lib/systemd/system/下会有rc-local.service文件，即我们需要的配置文件。

可以复制一份 不去动/lib/systemd/system/下面的文件

cp /lib/systemd/system/rc-local.service /etc/systemd/system/

{% highlight txt linenos %}
#  SPDX-License-Identifier: LGPL-2.1+
#
#  This file is part of systemd.
#
#  systemd is free software; you can redistribute it and/or modify it
#  under the terms of the GNU Lesser General Public License as published by
#  the Free Software Foundation; either version 2.1 of the License, or
#  (at your option) any later version.
 
# This unit gets pulled automatically into multi-user.target by
# systemd-rc-local-generator if /etc/rc.local is executable.
[Unit]
Description=/etc/rc.local Compatibility
Documentation=man:systemd-rc-local-generator(8)
ConditionFileIsExecutable=/etc/rc.local
After=network.target
 
[Service]
Type=forking
ExecStart=/etc/rc.local start
TimeoutSec=0
RemainAfterExit=yes
GuessMainPID=no
 
[Install]
WantedBy=multi-user.target
Alias=rc-local.service

{% endhighlight %}
默认是没有Install 部分的，在文件中添加

### 2、创建/etc/rc.local文件，赋予权限，在文件中添加要开机执行的命令.

touch /etc/rc.local

chmod 755 /etc/rc.local

在rc.local文件中添加想要开机执行的命令重启系统查看是否设置成功。



<a href="javascript:scroll(0,0)">-- 返回顶部 --</a>
