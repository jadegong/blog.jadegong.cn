---
title: C++/Qt开发中遇到的问题汇总
date: 2024-11-28 11:35:02
categories: tech
tags: [Qt]
---

## 动态库开发问题
- 使用VS开发Qt动态链接库，生成dll和lib后使用时报错：动态库 staticmetaobject 不允许dllimport静态数据成员的定义
需要将mydll_global.h文件中的代码修改一下：
<!-- more -->

```c++
#pragma once
#include <QtCore/qglobal.h>

#ifndef BUILD_STATIC
# if defined(MYDLL_LIB)
#  define MYDLL_EXPORT Q_DEC_EXPORT
# else
//#  define MYDLL_EXPORT Q_DEC_IMPORT // 该句修改为下一行，不能有IMPORT相关代码
#  define MYDLL_EXPORT
# endif
#else
# define MYDLL_EXPORT
#endif
```
- 使用VS开发C++动态链接库，使用ofstream时出现错误：使用未定义的 class“std::basic_ofstream<char,std::char_traits<char>>”
需要将预编译头文件放到fsrteam头文件之前：
```c++
#include "pch.h" // 旧版为 stdafx.h
#include <fstream>
```
- chrono和filesystem头文件中的namespace filesystem重复冲突
将filesystem头文件放在chrono头文件之前引入：即可让std::filesystem命名空间识别为filesystem头文件中的定义；

## Qt Quick相关技术
- 需要在qml中格式化输出日期时间？
使用 `Qt.formatDate(Date, format)` 输出，示例：`Qt.formatDate(new Date(), "yyyy-MM-dd hh:mm:ss")`。

- 在页面中行排布(Row)需要自动换行？
使用Flow也可以实现，并且Flow也有诸如spacing之类的属性。
