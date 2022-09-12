---
title: AngularJS知识点
date: 2017-01-25 09:10:50
categories: tech
tags: [AngularJS,前端]
---
## AngularJS常见问题
本文是在使用AngularJS进行开发工作时发现的一系列需要注意的地方，持续更新。
## AngularJS常见错误
- 在开发中遇见controller不起作用的情况，先检查是否有controller重名的情况。
- 在Angular表单验证中，必须绑定ng-model之后，formName.inputFieldName.$dirty等值才会定义
- 在Angular-datatables中，如果renderWith中的html有angular指令，那么需要在dtOptions中添加下面两个options进行重新编译
<!-- more -->
``` javascript
.withOption('createdRow', function(row, data, dataIndex) {
   // Recompiling so we can bind Angular directive to the DT
   $compile(angular.element(row).contents())($scope);
   .withOption('headerCallback', function(header) {
   if (!vm.headerCompiled) {
      // Use this headerCompiled field to only compile header once
      vm.headerCompiled = true;
      $compile(angular.element(header).contents())($scope);
   }
})
})
```
- 在Angular-datatables中，如果使用ng-repeat，并且行内有点击事件改变行内变量的值，使用``dtInstance.rerender()``重新渲染datatables；rerender有时会请求数据多次或者改变表的界面，当有后端数据时使用``dtInstance.reloadData()``来刷新表。
- 在Angular-datatables中，如果有多个表格，不要使用同一个dtInstance进行rerender，否则会出现渲染问题（不报错）
- AngularJS有自己的循环机制，当在jQuery事件中进行某个数据的改变时，经常会出现无法在视图中改变，所以需要使用``$scope.$evalAsync(function () {/*代码*/});``进行手动消息循环。
- AngularJS中出现指令中更改数据模型时，要特别注意作用域是否独立，还有作用域的继承。
