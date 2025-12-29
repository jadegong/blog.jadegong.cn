---
title: gorules规则引擎调研
date: 2025-12-29 11:14:12
categories: tech
tags: [gorules,golang,python,node]
---
# gorules规则引擎调研
## 调研技术
1. 完整项目：通过docker可部署，但源码未开放，不可进行二次开发；
2. 前端开源技术：jdm-editor（react+vite）
3. 规则引擎技术：zen-engine（rust, nodejs, go, python）
<!-- more -->
## 前端技术
1. 新建vite+react-ts项目，在项目中引入jdm-editor前端包，即可使用官方开源的前端功能，可进行规则创建/编辑/测试(需后端接口)等功能；
2. 可下载官方开源jdm-editor项目，本地二次修改源代码，然后重新打包，在本地前端项目中引用修改后的前端包；
```tsx
import { useState, } from 'react';
import { JdmConfigProvider, DecisionGraph } from '@gorules/jdm-editor';
...
const [graph, setGraph] = useState({ nodes: [], edges: [] });
...
<JdmConfigProvider>
  <DecisionGraph value={graph} onChange={setGraph} />
</JdmConfigProvider>
```
### jdm-editor功能
- Graph功能：

1. 请求Request：规则起始节点，输入数据；  
2. 表达式Expression Node：可以计算表达式结果，配置key和expression，可以根据expression配置输出key配置的字段值（key: status, expression: total.amount > 1_000 ? "green" : "red"， 意思为输出的数据status字段为输入的数据中total.amount值通过expressions中的表达式计算出来的值）；  
3. 函数Functions Node：通过编写自定义js函数处理输入数据，返回值为输出数据；  
4. 决策表格Decision Table：决策表格可以通过表格配置对应输入情况下的输出，支持多个输入条件和多个输出值；
5. 分支Switch Node：决策分支主要功能为不同条件走向不同的下一个节点；
6. 响应Response：规则结束节点，输出数据；
7. 所有中间节点支持直通功能，即不进行决策或表达式计算；


## 规则引擎技术

### 提供的功能
1. zen-engine提供规则解析/根据规则将输入生成输出等功能；
2. 支持文件系统、内存、闭包等多种加载器；
3. 支持表达式计算；

rust 示例代码：
```rust
use zen_expression::(evaluate_expression, json);

let content = json!({ "tax": { "percentage": 10 } });
let tax_amount = evalute_expression("50 * tax.percentage / 100", &context).unwrap();
assert_eq!(tax_amount, json!(5));
```

## gRPC示例代码python
1. 安装必要的python包；
```sh
pip install grpcio grpcio-tools protobuf zen-engine
```
2. 建立proto文件`proto/zen_evaluate.proto`；
```protobuf
syntax = "proto3";
package zen_evaluate;

// gRPC服务定义
service ZenEvaluater {
    rpc CalcEvaluate (EvaluateRequest) returns (EvaluateResponse) {}
}

// 请求消息体
message EvaluateRequest {
    string graph = 1;
    string input = 2;
}
// 响应消息体
message EvaluateResponse {
    bool success = 1;
    string result = 2;
}
```
3. 根据proto文件生成python代码；
```sh
python -m grpc_tools.protoc -I. --python_out=. --grpc_python_out=. proto/zen_evaluate.proto
```
执行完成后会在当前项目proto目录下生成两个py文件：`proto/zen_evaluate_pb2.py, proto/zen_evaluate_pb2_grpc.py`，后续在代码中会引用这两个文件；

4. 创建服务端和客户端代码，详情见示例代码；
5. 执行代码；
```sh
python server.py # 先启动gRPC服务端
python client.py # 启动客户端进行测试
```
6. 测试结果（CPU: R9 9900x）；

- server-python, client-python:  
--- ✅ 并发测试结果总结 ---  
并发请求数: 1000  
成功请求数: 1000  
总耗时 (包括所有等待时间): 0.5770 秒  
平均单次请求耗时: 0.0006 秒  
吞吐量 (QPS/RPS): 1733.19 请求/秒  

- server-python, client-nodejs:  
--- ✅ 并发测试结果总结 ---  
并发请求数: 1000  
成功请求数: 1000  
总耗时 (包括所有等待时间): 5.466387314 秒  
平均单次请求耗时: 0.0055 秒  
吞吐量 (QPS/RPS): 182.94 请求/秒  
- server-python, client-go:  
--- ✅ 并发测试结果总结 ---  
并发请求数: 1000  
成功请求数: 1000  
总耗时 (包括所有等待时间): 601.318751ms  
平均单次请求耗时: 372.216699ms  
吞吐量 (QPS/RPS): 1663.01 请求/秒  

## gRPC示例代码nodejs
1. 安装必要的npm包；
```sh
mkdir jdm-grpc-nodejs
cd jdm-grpc-nodejs
npm init -y
npm i @gorules/zen-engine @grpc/grpc-js @grpc/proto-loader
```
2. 建立proto文件`proto/zen_evaluate.proto`；
```protobuf
syntax = "proto3";
package zen_evaluate;

// gRPC服务定义
service ZenEvaluater {
    rpc CalcEvaluate (EvaluateRequest) returns (EvaluateResponse) {}
}

// 请求消息体
message EvaluateRequest {
    string graph = 1;
    string input = 2;
}
// 响应消息体
message EvaluateResponse {
    bool success = 1;
    string result = 2;
}
```
3. 创建服务端和客户端代码，详情见示例代码；
4. 执行代码；
```sh
node server.js # 先启动gRPC服务端
node client.js # 启动客户端进行测试
```
5. 测试结果（CPU: R9 9900x）；

- server-nodejs, client-python:  
--- ✅ 并发测试结果总结 ---  
并发请求数: 1000  
成功请求数: 1000  
总耗时 (包括所有等待时间): 0.2201 秒  
平均单次请求耗时: 0.0002 秒  
吞吐量 (QPS/RPS): 4543.62 请求/秒  

- server-nodejs, client-nodejs:  
--- ✅ 并发测试结果总结 ---  
并发请求数: 1000  
成功请求数: 1000  
总耗时 (包括所有等待时间): 5.30989529 秒  
平均单次请求耗时: 0.0053 秒  
吞吐量 (QPS/RPS): 188.33 请求/秒  

- server-nodejs, client-go:  
--- ✅ 并发测试结果总结 ---  
并发请求数: 1000  
成功请求数: 1000  
总耗时 (包括所有等待时间): 233.055916ms  
平均单次请求耗时: 149.514489ms  
吞吐量 (QPS/RPS): 4290.82 请求/秒  

## gRPC示例代码golang(1.25)
1. 安装必要的golang包；
```sh
mkdir jdm-grpc-go
cd jdm-grpc-go
go get github.com/gorules/zen-go
# go get google.golang.org/grpc
go install google.golang.org/protobuf/cmd/protoc-gen-go@latest
go install google.golang.org/grpc/cmd/protoc-gen-go-grpc@latest
```
2. 建立proto文件`proto/zen_evaluate.proto`；
```protobuf
syntax = "proto3";
package zen_evaluate;

// Golang核心编译选项(可选)
// 格式：option go_package = "模块路径/包名;自定义包别名";
option go_package = "./;proto";

// gRPC服务定义
service ZenEvaluater {
    rpc CalcEvaluate (EvaluateRequest) returns (EvaluateResponse) {}
}

// 请求消息体
message EvaluateRequest {
    string graph = 1;
    string input = 2;
}
// 响应消息体
message EvaluateResponse {
    bool success = 1;
    string result = 2;
}
```
3. 根据proto文件生成golang代码；
```sh
protoc --proto_path=. --go_out=proto --go-grpc_out=proto proto/zen_evaluate.proto
```
执行完成后会在当前项目proto目录下生成两个py文件：`proto/zen_evaluate_pb.go, proto/zen_evaluate_pb_grpc.go`，后续在代码中会引用这两个文件所在目录(即包名jdm-grpc-go/proto)；
4. 创建服务端和客户端代码，详情见示例代码；
5. 执行代码；
```sh
go run server/server.go # 启动gRPC服务端
```
6. 测试结果（CPU: R9 9900x）；

- server-go, client-python:  
--- ✅ 并发测试结果总结 ---  
并发请求数: 1000  
成功请求数: 1000  
总耗时 (包括所有等待时间): 0.1949 秒  
平均单次请求耗时: 0.0002 秒  
吞吐量 (QPS/RPS): 5131.08 请求/秒  

- server-go, client-nodejs:  
--- ✅ 并发测试结果总结 ---  
并发请求数: 1000  
成功请求数: 1000  
总耗时 (包括所有等待时间): 6.830945164 秒  
平均单次请求耗时: 0.0068 秒  
吞吐量 (QPS/RPS): 146.39 请求/秒  

- server-go, client-go:  
--- ✅ 并发测试结果总结 ---  
并发请求数: 1000  
成功请求数: 1000  
总耗时 (包括所有等待时间): 70.9154ms  
平均单次请求耗时: 45.180508ms  
吞吐量 (QPS/RPS): 14101.31 请求/秒  

## 测试结果（CPU：R9 9900x）
| Client\Server  | Python v3.11 | nodejs v20.19 | go v1.25 |
| :------------- | :----------- | :------------ | :------- |
| Python v3.11   | 并发请求数: 1000<br/>成功请求数: 1000<br/>总耗时 (包括所有等待时间): 0.5770 秒<br/>平均单次请求耗时: 0.0006 秒<br/>吞吐量 (QPS/RPS): 1733.19 请求/秒 | 并发请求数: 1000<br/>成功请求数: 1000<br/>总耗时 (包括所有等待时间): 0.2201 秒<br/>平均单次请求耗时: 0.0002 秒<br/>吞吐量 (QPS/RPS): 4543.62 请求/秒 | 并发请求数: 1000<br/>成功请求数: 1000<br/>总耗时 (包括所有等待时间): 0.1949 秒<br/>平均单次请求耗时: 0.0002 秒<br/>吞吐量 (QPS/RPS): 5131.08 请求/秒 |
| nodejs v20.19  | 并发请求数: 1000<br/>成功请求数: 1000<br/>总耗时 (包括所有等待时间): 5.466387314 秒<br/>平均单次请求耗时: 0.0055 秒<br/>吞吐量 (QPS/RPS): 182.94 请求/秒 | 并发请求数: 1000<br/>成功请求数: 1000<br/>总耗时 (包括所有等待时间): 5.30989529 秒<br/>平均单次请求耗时: 0.0053 秒<br/>吞吐量 (QPS/RPS): 188.33 请求/秒 | 并发请求数: 1000<br/>成功请求数: 1000<br/>总耗时 (包括所有等待时间): 6.830945164 秒<br/>平均单次请求耗时: 0.0068 秒<br/>吞吐量 (QPS/RPS): 146.39 请求/秒 |
| go v1.25       | 并发请求数: 1000<br/>成功请求数: 1000<br/>总耗时 (包括所有等待时间): 601.318751ms<br/>平均单次请求耗时: 372.216699ms<br/>吞吐量 (QPS/RPS): 1663.01 请求/秒 | 并发请求数: 1000<br/>成功请求数: 1000<br/>总耗时 (包括所有等待时间): 233.055916ms<br/>平均单次请求耗时: 149.514489ms<br/>吞吐量 (QPS/RPS): 4290.82 请求/秒 | 并发请求数: 1000<br/>成功请求数: 1000<br/>总耗时 (包括所有等待时间): 70.9154ms<br/>平均单次请求耗时: 45.180508ms<br/>吞吐量 (QPS/RPS): 14101.31 请求/秒 |

客户端为nodejs时，客户端为性能瓶颈，舍弃该测试结果；  
客户端为python时，服务端表现出的并发性能各不相同，其中server-go和server-nodejs表现基本在server-python的吞吐量的2倍以上。  
客户端为golang时，服务端表现出的并发性能差异明显，其中server-go吞吐量最大（14,101），server-nodejs其次（4,290），sever-python最差（1,663）。  
综上所述，在客户端不瓶颈的情况下，服务端为go语言的并发性能最佳。  
