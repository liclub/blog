---
layout: post
category: ORD使用
title: ORD进行SwTools配置后用户自定义的设置重置的解决办法
tagline: by 明不知昔
tags: 
  - ORD
  - BIM
published: true
---



在使用环境同步工具同步 swTools 环境后，发现自己定义的设置全都恢复默认了，每次都需要重新设置一次，简直有些让人抓狂。

<!--more-->

### 产生的原因

因为在每次同步环境的时候，需要对`C:\ProgramData\Bentley\OpenRoads Designer CE\Configuration`目录下的`ConfigurationSetup.cfg`文件进行重新配置，那么在配置的时候，将`_USTN_CONFIGURATION`也一并重置了。

`_USTN_CONFIGURATION`这个变量是用来配置用户的一些设置属性的。

当`_USTN_CONFIGURATION`被重置后，打开ORD时，就会恢复默认的用户配置。

### 解决方案

1. 手动：

   在每次更新的时候，备份`C:\ProgramData\Bentley\OpenRoads Designer CE\Configuration\ConfigurationSetup.cfg`文件，更新完成后，再恢复它。

2. 自动：

   在协同平台 v3.0.11.28 及以后的版本自动处理了该错误，并不会发生设置默认的情况。