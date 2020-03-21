---
layout: post
title: 打开腾讯通RTX管理器提示选择数据源
date: 2015-01-17 12:30:18
categories: [ODBC]
---

# 问题：双击 RTX管理器，弹出窗口提示选择数据源。
查看 配置 D:\Program Files (x86)\Tencent\RTXServer\Config\rtxserver.xml，
内有：

     <DB_Setting AutoTransPwd="true" DBConnstring="Driver={Microsoft access Driver (*.mdb)};DBQ=../db/rtxdb.mdb" Ver="0">

发现用的是Access数据库。

# 配置数据源

## 方法一：控制面板 - 系统和安全 - 管理工具数据源（ODBC）- 系统DNS（用户DNS也可）- 添加 - Microsoft Access Driver  - 输入数据源名称 - 找到你的数据库

只发现有SQL Server可建。

## 方法二：打开目录：“C:\Windows\SysWOW64”，双击该目录下的“odbcad32.exe”文件，进去ODBC数据源管理界面

按照提示操作，最后选择的数据文件为 D:\Program Files (x86)\Tencent\RTXServer\db\rtxdb.mdb

参见：

[Microsoft Access driver求助_百度知道](http://zhidao.baidu.com/link?url=L6fEvYdJZPVhLloahbtg3uxlMd83NXIiSE6-XdsbrwmM8ynmi5CfD_dTVP5vlh75RNI1yzQF45MPQwqDruMNSq)
