---
title: 制作Pod类库
categories: 环境搭建 #分类
date: 2016-09-22 15:42:33
#tags: [博客] #PodSpec
tags:
---
	
## 注册trunk(首先)
 在注册trunk之前，我们需要确认当前的CocoaPods版本是否足够新。trunk需要pod在0.33及以上版本，如果你不满足要求，打开Terminal使用ruby的gem命令更新pod:

``` 
$ sudo gem install cocoapods
```

注册trunk

```
$ pod trunk register chaussonHi@gmail.com 'chausson'  --verbose
```

邮箱以及用户名请对号入座。用户名我使用的是Github上的用户名。
--verbose参数是为了便于输出注册过程中的调试信息。执行上面的语句后，
你的邮箱将会受到一封带有验证链接的邮件，如果没有请去垃圾箱找找，有可能被屏蔽了。
点击邮件的链接就完成了trunk注册流程。
使用下面命令向trunk服务器查询注册信息验证
```
$ pod trunk me
```

## 创建自己的github仓库
 CocoaPods都托管在github上(官方链接为：https://github.com/CocoaPods)，所有的Pods依赖库也都依赖github，因 此第一步我们需要创建一个属于自己的github仓库。仓库创建界面如下图：
 
<img src="/img/pod_image1.png"  title="one">

<img src="/img/pod_image2.png"  title="two">

上图中标了序号的共5处，对应的说明如下：

* Repository name 仓库名称，这里写成CHAlertView，必填.(相应的库名)

* Description仓库的描述信息，可选的.

* 仓库的公开性,这里只能选Public，一个是因为Private是要money的，再一个Private别人看不到还共享个毛；

* 是否创建一个默认的README文件一个完整地仓库，都需要README说明文档，建议选上。当然不嫌麻烦的话你也可以后面再手动创建一个；

* 是否添加.gitignore文件.gitignore文件里面记录了若干中文件类型，凡是该文件包含的文件类型，git都不会将其纳入到版本管理中。是否选择看个人需要；

* 与license类型
正规的仓库都应该有一个license文件，Pods依赖库对这个文件的要求更严，是必须要有的。因此最好在这里让github创建一个，也可以自己后续再创建。我使用的license类型是MIT。

## 二、clone仓库到本地
为了便于向仓库中删减内容，需要先将仓库clone到本地，操作方式有多种，推荐使用命令行：

```
$ git clone https://github.com/***.git
```

操作完成后，github上对应的文件都会拷贝到本地，目录结构为：

<img src="/img/pod_image3.png"  title="three">

github上仓库中的.gitignore文件是以.开头的隐藏文件，因此这里只能看到两个。
后续我们的所有文件增、删、改都在这个目录下进行。
## 三、向本地git仓库中添加创建Pods依赖库所需文件
注意：以下描述的文件都要放在步骤二clone到本地的git仓库的根目录下面。

1、后缀为.podspec文件
该文件为Pods依赖库的描述文件，每个Pods依赖库必须有且仅有那么一个描述文件。文件名称要和我们想创建的依赖库名称保持一致，我的CHAlertView依赖库对应的文件名CHAlertView.podspec。

1.1 如何创建podspec文件
大家创建自己的podspec文件可以有两个途径：

①copy我的podspec文件然后修改对应的参数，推荐使用这种方式。

②执行以下创建命令：

```
$ pod spec create CHAlertView 
```
1.2 将文件导入克隆的文件夹

1.3podspec文件内容
CHAlertView.podspec的保存内容为:

```
         
 Pod::Spec.new do |s|
s.name = 'CHEnvirSwitcherDemo'
s.version = '0.0.1'
s.license = 'MIT'
s.summary = 'EnvironmentSwitcher for  iOS.'
s.homepage = 'https://github.com/ChaussonORG/CHEnvirSwitcherDemo'
s.authors = { 'Daxiongzyj' => '13761057550@163.com' }
s.source = { :git => "https://github.com/ChaussonORG/CHEnvirSwitcherDemo.git", :tag => "0.0.1"}
s.requires_arc = true
s.ios.deployment_target = '8.0'
s.source_files = "CHEnvirSwitcherService", "*.{h,m}"
end

  
```
<!-- <img src="/img/pod_image4.png"  title="four"> -->

该文件是ruby文件，里面的条目都很容易知道含义。
其中需要说明的又几个参数：

1.通过在终端里输入命令 $ vim CHAlertView.podspec (相应的podspec文件)打开podspec文件,直接复制黏贴以上内容,修改相应的信息,千万别改动符号!很关键!!!

2.s.license Pods依赖库使用的license类型，大家填上自己对应的选择即可。

3.s.source_files 表示源文件的路径，注意这个路径是相对podspec文件而言的。这个地方是一个难点 "CHEnvirSwitcherService"是文件夹, "*.{h,m}"代表的是该文件夹内的.m文件与.h文件


## 四、提交修改文件到github
经过步骤三，向本地的git仓库中添加了不少文件，现在需要将它们提交到github仓库中去。提交过程分以下几步：

1、上传步骤三中添加的文件
执行以下命令：

```
$ git add .

$ git commit -am "Release 0.0.1"

$ git push origin master
```

1、pod验证
执行以下命令：

```
$ set the new version to 0.0.1

$ set the new tag to 0.0.1 
```

这两条命令是为pod添加版本号并打上tag。然后执行pod验证命令：

```
$ pod lib lint
```

如果一切正常，这条命令执行完后会出现下面的输出：

```
-> CHAlertView (0.0.1)

CHAlertView passed validation.
```

到此，pod验证就结束了。
需要说明的是，在执行pod验证命令的时候，打印出了任何warning或者error信息，验证都会失败！如果验证出现异常，打印的信息会很详细，大家可以根据对应提示做出修改。

2、本地git仓库修改内容上传到github仓库
依次执行以下命令：

```
$ git add . 

$ git tag ‘0.0.1

$ git push --tags 

$ git push origin master
```

上述命令均属git的范畴，这里不多述。如果一切正常，github上就应该能看到自己刚添加的内容了。如下图所示：

<img src="/img/pod_image5.png"  title="five">

## 提交你的Podspec树干

```
$  pod trunk push CHAlertView.podspec
```

## 六、查看我们自己创建的Pods依赖库
查看是否成功可以通过cocoapods官网搜索<https://cocoapods.org>,如果显示下图表示成功

<img src="/img/pod_image6.png"  title="six">

