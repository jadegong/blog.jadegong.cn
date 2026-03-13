---
title: coding编程问题汇总
date: 2026-03-12 14:54:12
categories: tech
tags: [code]
---

## 一、前端问题
### React
- jdm-editor 问题，与react和antd版本问题：[issue222](https://github.com/gorules/jdm-editor/issues/222)，[issue162](https://github.com/gorules/jdm-editor/issues/162)，Modal.confirm不弹出；
Error: Warning: [antd: message] Static function can not consume context like dynamic theme. Please use 'App' component instead.
A: 通过将react与react-dom降级到18.x版本，有报错，但功能可使用。
<!-- more -->

## 二、系统问题
- clash verge服务安装问题：打开 clash-verge-rev 无法加载配置文件，一般在升级后出现；
解决方法：在命令行执行：`sudo clash-verge-service-uninstall` 和 `sudo clash-verge-service-install` 手动重装服务。

## 三、rust问题

## 四、实用技术/库
### 4.1 前端
- jsdiff: 类似 git diff 的对比算法库；
- papaparse: 解析 csv 的前端库；
