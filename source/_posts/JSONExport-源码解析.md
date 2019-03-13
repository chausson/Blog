---
title: JSONExport 源码解析
date: 2019-03-14 00:30:27
tags:
---

## 代码生成代码——JSONExport解析

因为最近想开发一款网路的大礼包，类似Mock API 到Model 的生成，通过一个按钮，或者一行命令去完成，来提高开发效率，节省重复劳动力，毕竟我们开发所有轮子的目的就在于此。

其中不免需要用到代码生成文件之类的功能，那如何生成呢？

答案有很多，不过大抵都是通过代码来生成代码，也有一些优秀的开源库和工具，接下来就介绍我们今天的主角**JSONExport**(https://github.com/Ahmed-Ali/JSONExport)，由作者[Ahmed-Ali](https://github.com/Ahmed-Ali)开发提供的优秀开源库。

它是一款Mac的App,通过Swift语言来编写，需要在网站上下载工程用Xcode编译运行。

运行后，大致是长这样:

![](jsonexport.png)

左边输入JSON，右边生成文件预览，右下角根据选择的语言，通过方法loadSupportedLanguages()来读取本地模版。

作者在工程中定义了一系列的数据模版来将JSON来映射解析，类似如这样:

``` json
{
  "langName": "名称", // 模版名称
  "modelStart": "",
  "basicTypesWithSpecialFetchingNeedsReplacements": [], // 类型转换映射
  "basicTypesWithSpecialStoringNeeds": [], // 基础类型映射
  "importForEachCustomType": "", 
  "reservedKeywords": [], //关键字转换
  "briefDescription": "", //生成的描述
  "utilityMethods": [],
  "dataTypes": {}, // 基础类型映射
  "wordsToRemoveToGetArrayElementsType": [],
  "constructors": [],
  "constVarDefinition": "",
  "modelDefinition": "",
  "genericType": "",
  "headerFileData": {
    "modelDefinitionWithParent": "",
    "modelEnd": "",
    "instanceVarDefinition": "",
    "utilityMethodSignatures": [],
    "constructorSignatures": [],
    "typesNeedSpecialDefinition": [],
    "modelStart": "",
    "importParentHeaderFile": "",
    "headerFileExtension": "",
    "modelDefinition": "",
    "importForEachCustomType": "",
    "instanceVarWithSpeicalDefinition": "",
    "staticImports": "#import <UIKit/UIKit.h>"
  }, // OC头文件
  "fileExtension": "", // 文件后缀，导出文件拼接用
  "arrayType": "NSArray",
  "basicTypesWithSpecialFetchingNeeds": [],
  "displayLangName": "", //客户端显示的语言名称,如:Swift,Objective-C
  "instanceVarDefinition": "",
  "supportsFirstLineStatement": "",
  "modelEnd": "",
  "staticImports": "", // 需要导入的静态包
  "importHeaderFile": ""
}
```



流程大致是这样:

![](progress.png)

模版中有需要一些动态插入的部分，如属性名，类名等之类的，用了<!ModelName!>,<!VarName!>作为关键次，在插入字符串过程中，会映射替换JSON的内容。可以看下**SharedConstants.swift**文件中：

```swif
let elementType = "<!ElementType!>"
let modelName = "<!ModelName!>"
let modelWithParentClassName = "<!ParentClass!>"
let varName = "<!VarName!>"
let capitalizedVarName = "<!CapitalizedVarName!>"
let varType = "<!VarType!>"
let varTypeReplacement = "<!VarBasicTypeReplacement!>"
let varTypeCast = "<!VarBasicTypeCast!>"
let capitalizedVarType = "<!CapitalizedVarType!>"
let lowerCaseVarType = "<!LowerCaseVarType!>"
let lowerCaseModelName = "<!LowerCaseModelName!>"
let jsonKeyName = "<!JsonKeyName!>"
let constKeyName = "<!ConstKeyName!>"
let additionalCustomTypeProperty = "<!AdditionalForCustomTypeProperty!>"
```

在上面的文件中基本是定义了大量关键词，从这里不难看出，其中的属性与JSON的模版文件来进行映射关系的，使插入的内容能够动态去生成。

在生成字符串的过程中，是逐行逐行来写入文件，就像我们的打印机一样一行一行来输出，用换行符\n和\t来进行格式排版的控制,最后替换关键词，这样的思路确实是能够胜任代码生成代码的部分。

可以知道的是在其中作者用一套解析方式，应用于多个模版，如果生成的文件需要增加的同时，也需要增加一个模版。下面具体来解析一下源码中的代码设计，整个思路大致是这样，但是作者使用的设计模式还是值得学习的。

是不是现在有点疑惑，各种模型生成的方式不一样Swift,Objective-C,Java，怎么用一套规则去解析生成呢。那么我们就了解下源码中的一些代码设计方式。首先看一下一些类的结构，大致分为三组，也可以定义为三层:

- Supported Languages(模版文件) => Data层
- Lang Data Models (模版文件模型)  => Model层
- File content generators （文件生成转化类) => Service层

##### Supported Languages(模版文件) => Data层

##### Lang Data Models (模版文件模型)  => Model层

##### File content generators （文件生成转化类) => Service层

当然还有其他文件生成的方式，如**Sourcery** 框架，但只针对于Swift文件语言写入。利用Swift中xxxxx。


