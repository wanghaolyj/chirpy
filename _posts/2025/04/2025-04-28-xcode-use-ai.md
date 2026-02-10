---
layout: post
title: 在Xcode中使用ai编程助手
date: 2025-04-28 16:16 +0800
categories: [Xcode]
tags: [Xcode, ai, 编程]
---

在Xcode中使用ai编程助手可以极大地提高开发效率，特别是在处理复杂逻辑和编写重复代码时。以下是一些可以在Xcode中使用的ai编程助手

### copilot

copilot是GitHub推出的一个AI编程助手，它可以自动完成代码的编写。在Xcode中使用copilot的方法如下：
官网项目地址<https://github.com/github/CopilotForXcode/releases>下载最新版本安装,之后根据提示授权即可使用。

#### 特点

使用过一段时间，copilot的体验非常好，补全速度快，准确率也高。但是1000次试用过后，需要付费才能继续使用。忙起来根不不够用。

### 腾讯AI助手

做安卓的朋友看文档，偶然发现腾讯AI助手的Xcode插件，官网首页看不到Xcode插件，但是在文档中发现的Xcode插件使用教程，文档地址是：<https://cloud.tencent.com/document/product/1749/115935>

#### 特点

登录腾讯云账号，个人免费使用，但是速度较慢，准确率还行。但是一个慢子真的太影响使用了，用了几次就放弃了。

### Comate

Comate是百度推出的一个AI编程助手，官网是https://comate.baidu.com/zh。安装之后，会发现和腾讯AI界面和tencent差不多，也不知道他俩谁抄谁的。

#### 特点

速度最快，准确率也不错。比Copliot更快。 是我目前使用的助手。一个快字，得天下。

### CopilotForXcode

CopilotForXcode是我发现的最早的Xcode插件，官网地址是：<https://github.com/intitni/CopilotForXcode>。
自由度很高的一个插件。提供了Github Copilot和的两种选择。其中codium是免费使用的，但是速度较有点慢。

#### 特点

不付费的话，无法解禁tab补全需要自定义快捷键，手动按快捷键才可补全，体验有点不好。但是他出现的早属于我最开始一直使用的助手。而且我解禁了tab补全，体验还行。


### 自定义API Key的使用

CopilotForXcode可以自定义api使用的key，这点很方便。在 <https://github.com/intitni/CustomSuggestionServiceForCopilotForXcode> 中下载启动服务。

打开软件后在设置页面

`Model`项选择`Custom Model(FIM API)`, 之后点击`Edit Model` 按钮，之后

`Format` 选择 `Mistral`

`URL` 选择 `Full URL`,下面的网址填 `https://api.deepseek.com/beta/completions`

在`Model Name`中填`deepseek-chat`

在`API Key`后的钥匙按钮中添加deepseek 的key即可。

如下图所示

![image](assets/img/2025/04/Snipaste_2025-04-29_09-03-20.png)

保存后，我们来配置`CopilotForXcode`

过程如图所示

![image](assets/img/2025/04/Snipaste_2025-04-29_09-15-58.png)

首先打开状态栏的扩展配置

![image](assets/img/2025/04/Snipaste_2025-04-29_09-16-44.png)

之后选择扩展

![image](assets/img/2025/04/Snipaste_2025-04-29_09-17-02.png)

最后激活扩展

最后在```CopilotForXcode```的设置中选择自定义的插件即可。如图所示

![image](assets/img/2025/04/Snipaste_2025-04-29_09-02-02.png)


配置之后即可使用了。 我是用的是deepseek的api key, 速度很慢，尝试了下未使用。

最新版本支持了`Qwen`的模型。具体流程未尝试，应该和deepseek类型，如果你感兴趣可以尝试一下。

### 总结

以上就只我在Xcode中使用过的AI编程助手。各有优缺点，使用流程也大致想死，可以根据自己的需求选择合适的工具。
我现在选择的是Comate。一个快，给我带来了无与伦比的体验。



