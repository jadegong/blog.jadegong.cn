---
title: 原生javascript发送POST请求
date: 2017-01-25 09:06:13
categories: tech
tags: [javascript,ajax,前端]
---
## 原生javascript发送ajax请求
前段时间发现自己对javascript基础知识掌握不到位，对于原生的js发送请求不是十分了解，便去学习了以下。
## 开发背景
最近在开发一个展示数据图表的网站，由于网站没有太多的DOM操作，于是并没有使用jQuery库；在网站开发过程中，需要进行数据请求，然而在使用js进行数据请求时，发生了一些错误——由于未将发送的数据进行编码，所以导致服务器接收错误。
<!-- more -->
## 解决方案
正确的请求方法如下：
``` javascript
function sendData() {//默认参数为：url, data
	var args = Array.prototype.slice.call(arguments);
	var urlEncodedData = '';
	for (name in args[1]) { //将对象数据转换成URL字符串，并进行编码(UTF-8)
		urlEncodedData += name + '=' + encodeURIComponent(args[1][name]) + '&';
	}
	//删掉多余的 "&" 字符
	urlEncodedData = String.prototype.slice.call(urlEncodedData,0,-1);
	var xmlhttp = new XMLHttpRequest();
	xmlhttp.onreadystatechange = function() {
		if (xmlhttp.readyState == 4 && xmlhttp.status == 200) {
			ret = xmlhttp.responseText;
			console.log(ret);
			//处理返回的数据
		}
	};
	xmlhttp.open('POST', args[0]);
	//设置请求header  !important
	xmlhttp.setRequestHeader('Content-Type', 'application/x-www-form-urlencoded');
	xmlhttp.send(urlEncodedData);
}
```
更多详细的使用JavaScript发送数据的方式请访问[此处](https://developer.mozilla.org/zh-CN/docs/Web/Guide/HTML/Forms/Sending_forms_through_JavaScript%22%E6%AD%A4%E5%A4%84%22)  
文章来源：https://developer.mozilla.org/zh-CN/docs/Web/Guide/HTML/Forms/Sending_forms_through_JavaScript
