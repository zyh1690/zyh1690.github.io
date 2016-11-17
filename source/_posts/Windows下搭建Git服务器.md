---
title: Windows下搭建Git服务器
date: 2016-11-16 22:38:25
tags:
---

最近实在忍受不了SVN的各种限制（集中式版本控制，一旦服务器不在线，就无法commit），所以建议项目组长换成Git服务器，也就有了这篇Git服务器搭建文章。

需要的软件：

1. JRE
2. Gitblit

# JRE

安装Java运行环境，很简单，网上一搜一大堆，在此不再赘述。

# Gitblit

## What is Gitblit?

> Gitblit is an open-source, pure Java stack for managing, viewing, and serving Git repositories.
> It's designed primarily as a tool for small workgroups who want to host centralized repositories.

## Why use it?

因为它**免费**。

## 下载

[GitBlit 官网](http://gitblit.com/)

## 安装

- 下载下来后，是个zip包，解压到你想放的文件夹即可
  <a href="http://www.zyh1690.org/wp-content/uploads/2016/11/2016111613322445.png"><img src="http://www.zyh1690.org/wp-content/uploads/2016/11/2016111613322445.png" alt="gitblit" class="aligncenter size-full wp-image-2332" /></a>
- 创建用于放置 repositories 的文件夹，博主是创建的是<code>E:\GitServer</code>
- 配置gitblit目录中data文件夹下的gitblit.properties文件
   <a href="http://www.zyh1690.org/wp-content/uploads/2016/11/201611161338337.png"><img src="http://www.zyh1690.org/wp-content/uploads/2016/11/201611161338337.png" alt="gitblitconfig" class="aligncenter size-full wp-image-2334" /></a>
- 打开defaults.properties和gitblit.properties两个文件
   <a href="http://www.zyh1690.org/wp-content/uploads/2016/11/2016111614005429.png"><img src="http://www.zyh1690.org/wp-content/uploads/2016/11/2016111614005429.png" alt="configfilenew" class="aligncenter size-full wp-image-2337" /></a>
   我们可以在gitblit.properties中自定义自己的设置，它会覆盖defaults.properties中的默认配置。图中红圈部分是我自定义的一些设置总共4项，

```lua
git.repositoriesFolder = E:/GitServer  --设置仓库所在位置，注意windows下要换成/，否则/会被解释成转义字符，
server.httpPort = 8080  --设置访问端口
server.httpBindInterface = zyh1690-pc  --"zyh1690-pc"是我在局域网中的电脑名，如此设置，是为了避免因为自动分配ip地址，导致之前的地址失效，当然你也可以设置为你的IPV4地址
server.httpsBindInterface = localhost
```

大家可以在defaults.properties查看每项的具体信息。

- 运行gitblit.cmd批处理文件
  <a href="http://www.zyh1690.org/wp-content/uploads/2016/11/2016111614050898.png"><img src="http://www.zyh1690.org/wp-content/uploads/2016/11/2016111614050898.png" alt="gitblitcmd" class="aligncenter size-full wp-image-2338" /></a>
  运行结果如下表示成功：
  <a href="http://www.zyh1690.org/wp-content/uploads/2016/11/2016111614073420.png"><img src="http://www.zyh1690.org/wp-content/uploads/2016/11/2016111614073420.png" alt="result" class="aligncenter size-full wp-image-2339" /></a>
  此时我们就可以在浏览器打开GitBlit并使用了，**默认用户名密码都是admin**
  <a href="http://www.zyh1690.org/wp-content/uploads/2016/11/2016111614092729.png"><img src="http://www.zyh1690.org/wp-content/uploads/2016/11/2016111614092729.png" alt="blitinbrow" class="aligncenter size-full wp-image-2340" /></a>
- 最后一步，设置以Windows Service方式启动GitBlit，在GitBlit目录下找到installService.cmd批处理文件，

> 修改ARCH，x86： SET ARCH=X86；x64：SET ARCH=amd64
> 添加CD：SET CD=D:/Program Files/gitblit-1.8.0
> 删除StartParams里的启动参数
> 修改后的版本应如下：

```bash
@REM Install Gitblit as a Windows service.

@REM gitblitw.exe (prunmgr.exe) is a GUI application for monitoring 
@REM and configuring the Gitblit procrun service.
@REM
@REM By default this tool launches the service properties dialog
@REM but it also has some other very useful functionality.
@REM
@REM http://commons.apache.org/daemon/procrun.html

@REM arch = x86, amd64, or ia32
SET ARCH=amd64
SET CD=D:/Program Files/gitblit-1.8.0

@REM Be careful not to introduce trailing whitespace after the ^ characters.
@REM Use ; or # to separate values in the --StartParams parameter.
"%CD%\%ARCH%\gitblit.exe"  //IS//gitblit ^
		 --DisplayName="gitblit" ^
		 --Description="a pure Java Git solution" ^
		 --Startup=auto ^
		 --LogPath="%CD%\logs" ^
		 --LogLevel=INFO ^
		 --LogPrefix=gitblit ^
		 --StdOutput=auto ^
		 --StdError=auto ^
		 --StartPath="%CD%" ^
		 --StartClass=org.moxie.MxLauncher ^
		 --StartMethod=main ^
		 --StartParams="" ^
		 --StartMode=jvm ^
		 --StopPath="%CD%" ^
		 --StopClass=org.moxie.MxLauncher ^
		 --StopMethod=main ^
		 --StopParams="--stop;--baseFolder;%CD%\data" ^
		 --StopMode=jvm ^
		 --Classpath="%CD%\gitblit.jar" ^
		 --Jvm=auto ^
		 --JvmMx=1024
```

保存并以管理员身份运行。此时打开“服务”，就可以在列表中看到GitBlit了：
<a href="http://www.zyh1690.org/wp-content/uploads/2016/11/2016111614174569.png"><img src="http://www.zyh1690.org/wp-content/uploads/2016/11/2016111614174569.png" alt="blitinserver" class="aligncenter size-full wp-image-2341" /></a>

# 使用

创建一个版本库
<a href="http://www.zyh1690.org/wp-content/uploads/2016/11/2016111614193557.png"><img src="http://www.zyh1690.org/wp-content/uploads/2016/11/2016111614193557.png" alt="createrepo" class="aligncenter size-full wp-image-2342" /></a>
创建完成后显示如下：
<a href="http://www.zyh1690.org/wp-content/uploads/2016/11/2016111614203426.png"><img src="http://www.zyh1690.org/wp-content/uploads/2016/11/2016111614203426.png" alt="createrepocomp" class="aligncenter size-full wp-image-2343" /></a>
可以看到我们可以使用各种客户端来clone此仓库，博主使用的SourceTree，复制版本库地址，到SourceTree中Clone到本地：
<a href="http://www.zyh1690.org/wp-content/uploads/2016/11/2016111614240698.png"><img src="http://www.zyh1690.org/wp-content/uploads/2016/11/2016111614240698.png" alt="insourcetree" class="aligncenter size-full wp-image-2344" /></a>
团队其他成员使用，可以使用SourceTree自带的Putty，生成公钥私钥。
Tips：你可以使用admin账号添加用户，并编辑他们的权限，不同的用户有对应的clone地址-只需将admin改为相应的username即可。

如：<code>ssh://**admin**@zyh1690-pc:29418/HuiDemo.git --> ssh://**username**@zyh1690-pc:29418/HuiDemo.git</code>

本文完。