---
title: 造一个Iwara-Telegram-Bot
author: 夏同光
date: 2022-08-01 17:37:08 +0800
categories: [Tutorial]
tags: [python-telegram-bot, chatbot, Telegram, python3, requests, sqlite3, opencv-python, beautifulsoup4, spider, Iwara]
img_path: /assets/img/Iwara-Telegram-Bot
pin: true
# math: true
# mermaid: true
# image:
#   path: /commons/devices-mockup.png
#   width: 800
#   height: 500
#   alt: Responsive rendering of Chirpy theme on multiple devices.
draft: false
---

本篇文章记录了我开发[Iwara-Telegram-Bot](https://github.com/watanabexia/Iwara-Telegram-Bot)中途遇到的一些问题和解决方案。

# 从[iwara.tv](https://iwara.tv)爬取视频

## 登录

> 所需工具： `requests`, `beautifulsoup4` 
{: .prompt-info }

如果想要获取自己订阅的用户发布的视频，就必须登录。Iwara有一点点反机器人机制，登录Iwara时，除了用户名和密码以外，post请求还必须携带其他的表单信息：
```python
{
    'name':<user name>,
    'pass':<password>,
    'form_build_id':'form-dummy',
    'form_id':'user_login',
    'antibot_key': get_login_key(loginPageHtml),
    'op': "ログイン",
}
```
其中`antibot_key`需要从登录页面自己获取，具体位置在表单里面被隐藏的地方

![Window shadow](Iwara_antibot.png){: .shadow }
_`antibot_key`的位置_

## [iwara.tv/videos](https://iwara.tv/videos) 无内容
有时候，Iwara的最新视频页面会什么都不显示，但网站其实其他部分都运转正常。解决方法是使用另一个更稳定的最新视频页面：`https://iwara.tv/videos-3` 。

## 下载视频
并不需要专门去从视频详细页面爬取下载链接，Iwara似乎有一个API，只要向 `https://iwara.tv/api/video/\<videoID\> `发送一个get请求，就会收到三种清晰度各自的下载链接。之后便可以使用`requests`来下载视频。

# 发送视频到Telegram

> 所需工具： `python-telegram-bot`, `opencv-python` 
{: .prompt-info }

## 使用 Local Bot API Server
其实在所有Telegram Bot被创建的时候，默认是由Telegram官方来提供API的，也就是通过`https://api.telegram.org/<bot API Token>/`来和Bot交互。然而，使用这种方法来和Bot交互有一个`50MB`的上传文件大小限制，这对于一般的Iwara视频来说肯定是不够用的。所以我们需要有一个自己部署的**Local Bot API Server**。这里的Local并不代表是在本地跑，而是相对于Telegram官方服务器来说是Local的，也就是用户自己部署的。

### 获取`api_id`和`api_hash`
详见 https://core.telegram.org/api/obtaining_api_id 。注意在提交申请的时候，使用某些IP会收到没有任何额外信息的ERROR提示，此时应该换用别的IP。笔者当时使用了一个俄罗斯🇷🇺的IP成功提交申请。

### 安装 Local Bot API Server
需要在设备上编译。如果Linux机器的内存在`1G`以下，直接编译会导致系统卡死。如果机器有足够的硬盘空间的话，可以考虑增加swap的容量。笔者的机器就是只有`1G`内存，一开始编译到50%时就会卡死，后来增加了`7G`swap容量后，成功编译并安装。