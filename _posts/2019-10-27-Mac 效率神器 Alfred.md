---
title: Mac 效率神器 Alfred
description: 
categories:
 - App
tags: Mac
---


# 简介

- [Alfred 官网](https://www.alfredapp.com/)

> Alfred is an award-winning app for macOS which boosts your efficiency with hotkeys, keywords, text expansion and more. Search your Mac and the web, and be more productive with custom actions to control your Mac.

在 Mac 系统中不管是自带的还是第三方的都有很多可以提高效率的软件，比如自带的 Spotlight，第三方的 Better And Better 都是很优秀的效率软件。但是 Alfred 才是最出名的效率神器，它跟 Spotlight 相似，但它的功能远比 Spotlight 强大，我们完全可以用它来代替 Spotlight。
<!-- more -->
Alfred 的基本功能是免费的，但部分强大的扩展功能和 Workflows 功能是需要购买 Powepack 的，[价格](https://www.alfredapp.com/shop/)如下。当然需要收费的软件就会有破解版，可以自行 Google。

![](/assets/post_imgs/alfred/price.png)

## 为什么要用 Alfred

![](/assets/post_imgs/alfred/alfred.png)

因为使用 Alfred，你的所有操作几乎都是以下三个步骤：

1. 快捷键唤起 Alfred，比如 `command + space`
1. 功能关键字 + 搜索内容，比如 `gg alfred` 用谷歌搜索 alfred
1. 回车走人 

# General

![](/assets/post_imgs/alfred/general.png)

## startup

配置是否开机自启动

## Alfred Hotkey

配置唤醒 Alfred 的快捷键，可以用`control、option、command、shift、空格`这几个键随意组合，也可以配置为双击某个按键

## Where are you

设置你所在的国家，在进行快捷搜索的时候 Alfred 会根据这个配置自动使用网站的目标国家版本，比如 Google/ebay 等等（好像没什么用对我们来说）

## Permission

Alfred 需要某些系统权限才能完成一些自动化的工作，在这里可以提前授权。不授权的话，在使用过程中 Alfred 会弹出响应的提示框申请权限，到时再授权也可以。

# Features

![](/assets/post_imgs/alfred/feature.png)

Features 这个选项里面的功能基本上就是上面列出来的高效操作了。下面对一些比较有用的操作做一下说明。

## 快速启动 APP

唤醒 Alfred 窗口后，输入你想要启动的 APP 名称或包含的关键字（中英文都支持），然后在下方的候选列表中 up/down 选择，enter 启动

![](/assets/post_imgs/alfred/launch_wechat.png)

## 文件搜索

- 文件搜索相关操作的配置在 Features-&gt;File Search  选项卡中

![](/assets/post_imgs/alfred/file_search.png)

- open + 文件名 或者 空格+文件名 或者直接输入文件名 即可快速定位到该文件，enter 直接用默认程序打开，comand+enter 则是打开该文件所在文件夹

![](/assets/post_imgs/alfred/open_file.png)

- in + 关键字 快速查找包含此关键字的文件，例如一个文本文件中有一个单词 Alfred，那么输入 in + Alfred 时改文件就会被查找出来

![](/assets/post_imgs/alfred/in_file.png)

## 快速使用某网站的搜索功能

- Web Search 选项默认提供了一些热门的网站的快速搜索功能，比如 Google、Twitter、YouTube 等等，输入其配置的关键就可以触发该网站的搜索功能（如果觉得关键字不合适也可以进行修改，在配置页面双击改关键字即可，Google 我就改成了 gg），例如输入 gg Alfred  然后 enter，就会自动打开默认浏览器然后访问 Google 搜索 Alfred。

![](/assets/post_imgs/alfred/gg_search.png)

- 除了使用默认提供的网站搜索之外，Alfred 也支持自定义 web search 配置，下面以添加百度搜索为例
    - 打开百度搜索搜索 Alfred，复制结果页 URL，大概长这样 `https://www.baidu.com/s?wd=Alfred&rsv_spt=1&rsv_iqid=0xff559d690039e457&issp=1&f=8&rsv_bp=1&rsv_idx=2&ie=utf-8&rqlang=&tn=baiduhome_pg&ch=&rsv_enter=1&rsv_dl=ib&inputT=4589` 可以注意到重点在 wd=Alfred，后面的参数都是多余的。因为 Alfred 是我们的关键字，所以我们只需要将 `wd=Alfred` 中的 Alfred 替换成占位符 `{query}`即可。配置方式如下图

![](/assets/post_imgs/alfred/add_custom_search.png)

![](/assets/post_imgs/alfred/bd_search.png)

## 快速打开书签

快速打开书签的关键字是 bm(bookmarks)

![](/assets/post_imgs/alfred/bm.png)

## 强大的剪贴板历史

clipboard history 功能可以保存你最近的复制的文本、图片、文件，你可以分别设置他们保留的时间，也可以在 `advanced` 选项下设置是否自动粘贴到当前激活的应用上。

![](/assets/post_imgs/alfred/clip_history.png)

![](/assets/post_imgs/alfred/clip_advanced.png)

当你输入 clipboard 或者使用快捷键 `command+option+c`之后就可以进入剪贴板历史，选择某个历史记录回车即可再次复制。

![](/assets/post_imgs/alfred/clip_history_panel.png)

## 快速使用文本片段

Snippets 是一个保存文本片段的地方，你可以将一些常用的文本模板或者代码片段保存到这里，然后通过快捷键或者设定的关键字快速使用该文本。Snippets 还提供了一些占位符功能，比如在片段中使用`{date}`	Snippets 会自动帮你替换为当前日期。当然你不需要记住这些占位符，因为你可以在 Snippets 编辑界面中通过菜单选择相应的占位符。

![](/assets/post_imgs/alfred/snippets.png)

![](/assets/post_imgs/alfred/add_snippet.png)

保存完成后，在 Alfred 窗口输入 snip + ‘片段关键字或名称’ 即可快速搜索到该片段，然后 enter 键复制选中的 snippet。或者在前面的剪贴板历史中，第一项 `All Snippets`	中选择某个片段。

![](/assets/post_imgs/alfred/snip_search.png)

## 计算器

在 Alfred 的窗口中可以直接输入算式计算，如果要进行复杂运算的话需要先输一个`=`	。对于计算结果，按 enter 复制结果，按 = 将结果赋值到 Alfred 窗口，方便继续计算。

![](/assets/post_imgs/alfred/calculator.png)

![](/assets/post_imgs/alfred/sin_cal.png)

## 字典

这个功能比较少用，可以用`有道词典 workflow` 代替

![](/assets/post_imgs/alfred/spell.png)

## 系统操作

通过 Alfred 可以输入指定关键字执行一些系统操作，比如锁定屏幕、显示屏保、加减音量、清空废纸篓等等。具体可用的操作查看 System 选项提供的配置项。

![](/assets/post_imgs/alfred/system.png)

## 快速执行一条命令行指令

Terminal 默认使用前缀`>` 识别指令，默认用系统自带的终端执行。如果想切换为 iterm2 的话，在 application 选项中选择 custom，然后粘贴一段脚本，脚本内容查看 [stuartcryan/custom-iterm-applescripts-for-alfred](https://github.com/stuartcryan/custom-iterm-applescripts-for-alfred)

例如我命令行已经配置了 alias jump 执行自动登录跳板机，那么我需要在 Alfred 中输入 `> jump`	就可以自动打开终端登录跳板机了。

# Workflows 

Alfred 最强大的功能就是 workflow，使用现有的 workflow 很简单，操作跟前面自带的那些功能几乎一致，但是如果想自己写 workflow 的话就需要一定的学习成本了，因为需要通过写代码来完成。对于普通用户来说，我们一般只需要用 workflow，因为已经有很多现成的 workflow，几乎可以满足我们的使用需求了。

## 安装

非常简单，首先下载 workflow 文件（`.alfredworkflow`文件），双击打开，导入即可。例如导入一个[沙雕游戏打乒乓球](https://github.com/vitorgalvao/alfred-workflows/tree/master/PingPong)。

![](/assets/post_imgs/alfred/install_workflow.png)

## 推荐

- [有道翻译](https://github.com/whyliam/whyliam.workflows.youdao)
    - 快速翻译，复制翻译结果，具体功能使用可以查看官方文档。

![](/assets/post_imgs/alfred/yd.png)

- [New File](https://github.com/vitorgalvao/alfred-workflows/tree/master/NewFile)
    - 快速在当前文件夹创建一个新文件，Mac 的右键是没有新建文件的功能，有时候真令人头秃。所以这时候 New File  就出现了。只需要在 Alfred 窗口中输入 `nf bald` 就可以在当前文件夹创建一个 `bald.txt`  如果输入`nfo` 则是创建并打开。

![](/assets/post_imgs/alfred/nf.png)

- [CoffeeCoffee](https://github.com/vitorgalvao/alfred-workflows/blob/master/CoffeeCoffee/README.md)
    - 防止电脑进入睡眠状态
- [IP Address](https://github.com/alexchantastic/alfred-ip-address-workflow)
    - 快速获取本机 IP 地址

![](/assets/post_imgs/alfred/ip.png)

- [Kill Process](https://github.com/zenorocha/alfred-workflows)  
    - 快速杀掉某个进程，例如：我杀我自己

![](/assets/post_imgs/alfred/kill.png)

- [V2EX](https://github.com/hzlzh/Alfred-Workflows/blob/master/Downloads/V2EX.alfredworkflow)
    - 摸鱼？？？？

## 获取 workflow

你可以从以下链接获取到 workflow

- [https://github.com/vitorgalvao/alfred-workflows](https://github.com/vitorgalvao/alfred-workflows)
- [https://github.com/hzlzh/Alfred-Workflows](https://github.com/hzlzh/Alfred-Workflows)
- [http://alfredworkflow.com/](http://alfredworkflow.com/)
- [https://github.com/zenorocha/alfred-workflows](https://github.com/zenorocha/alfred-workflows)
- [https://www.alfredapp.com/workflows/](https://www.alfredapp.com/workflows/)

或者

![](/assets/post_imgs/alfred/gg_alfred.png)

![](/assets/post_imgs/alfred/bd_alfred.png)

# Windows？

推荐 [Wox](http://www.wox.one/) ，完全开源免费。基本功能相似，在 Wox 中 [Plugin](http://www.wox.one/plugin) 就相当于 Alfred 中的 Workflow

> Launcher for Windows, an alternative to Alfred and Launchy. 
