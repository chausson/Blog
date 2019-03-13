---
title: 自动化构建工具Jenkins
date: 2016-10-31 18:05:50
tags:
categories: 环境搭建 #分类
---

### 安装
操作系统：Windows
注意：以下所有的安装和配置目录都尽量不要出现中文，以免有错误

##### 1.0 安装JDK环境

##### 1.1 下载网址：[https://jenkins.io/index.html](https://jenkins.io/index.html)

##### 1.2 Jenkins安装和配置

＊ 直接安装：直接解压压缩包，双击.exe文件进行安装。

＊ 命令行安装：在cmd中输入：java -jar jenkins.war

    java -jar jenkins.war
          
##### 1.3 **Jenkins** 安装验证

在浏览器中输入：*http://localhost:8080* 如果能正常跳转，说明安装成功

    http://localhost:8080

此处的端口 **8080** 可以根据自己的需要进行修改，找到安装主目录下的 **jenkins.xml** 文件中的这段代码，找到其中的 **8080** 端口进行修改，然后保存文件，重启浏览器。

    <arguments>-Xrs -Xmx256m -Dhudson.lifecycle=hudson.lifecycle.WindowsServiceLifecycle -jar "%BASE%\jenkins.war" --httpPort=8080 --webroot="%BASE%\war"</arguments>

---
### 第二部分 Jenkins的使用
上面已经完成Jenkins的安装，下面主要介绍Jenkins的使用。


#### 1：用户注册
**Jenkins** 成功安装之后，会进入到下面的锁定界面。根据页面上的提示找到安装目录下的 **initialAdminPassword** 文件打开，复制里面的内容，输入到下面的方框内，

![](http://i.imgur.com/ER5U0is.png)

点击下一步会进入到插件的安装界面，主要有：默认安装和选择安装。请自己进行选择，我选择的是默认安装，会比较慢。

![](http://i.imgur.com/aVemMoL.png)

插件安装之后，会进入到用户注册界面。此处可能插件不一定能够全部安装成功，会卡主安装界面。

![](http://i.imgur.com/UnW2IqV.png)

不用担心。我的解决方式是，关闭浏览器，重新打开Jenkins。会进入到下面界面

![](http://i.imgur.com/myb4x4F.png)

注意一定要点击 **continue** ，才会进入到用户注册界面，点击Retry又会回到插件下载界面

![](http://i.imgur.com/hdaGxc9.png)

注册信息填写好之后，选择**Save and Finish** 就会进入到Jenkins的主界面。

#### 2：插件下载
进入到jenkins首先要进行相应的插件下载，不然后期工作无法展开。Android需要的插件主要有：**Git Plugin**、**Gradle Plugin**、**SSH Credentials Plugin**。 进入到Jenkins主界面选择 **系统管理** 

![](http://i.imgur.com/NsUi08u.png)

进入之后选择 **管理插件**，进入到插件下载界面，进行相应插件下载。

![](http://i.imgur.com/37JT32x.png)

#### 3：系统设置（重点）
在插件下载完成之后，进入到 **系统设置** 界面，这里可以设置 **JDK**、**Gradle**、**Android SDK**的环境变量，如果在安装这些软件的时候已经配置好了系统变量，那么这里就不需要进行设置了。

![](http://i.imgur.com/tPPMXfJ.png)

