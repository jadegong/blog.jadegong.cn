---
title: gorules规则引擎调研
date: 2025-12-29 11:14:12
categories: tech
tags: [gorules,golang,python,node]
---

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
server.py
```python
import grpc
import time
import json
from zen import ZenEngine
from concurrent import futures

# 导入生成的代码
from proto import zen_evaluate_pb2 as pb2
from proto import zen_evaluate_pb2_grpc as pb2_grpc

_ONE_DAY_IN_SECONDS = 60 * 60 * 24

class MainZenEvaluaterServicer(pb2_grpc.ZenEvaluaterServicer):
    def CalcEvaluate(self, request, context):
        try:
            # DONE: evaluate zen engine
            content = json.loads(request.graph)
            input = json.loads(request.input)
            engine = ZenEngine()
            decision = engine.create_decision(content)
            # 返回trace数据
            # opts = DecisionEvaluateOptions(trace=True, max_depth=10)
            opts = {
                "trace": True,
                "max_depth": 10,
            }
            result = decision.evaluate(input, opts)
            result_str = json.dumps(result)
            print("str=", result_str)
            return pb2.EvaluateResponse(success = True, result = result_str)
        except Exception as e:
            print("error Occurred: ", e)
            return pb2.EvaluateResponse(success = False, result = str(e))

def serve():
    # 创建 gRPC 服务器
    server = grpc.server(futures.ThreadPoolExecutor(max_workers=10))
    # 将服务实现注册到服务器
    pb2_grpc.add_ZenEvaluaterServicer_to_server(
        MainZenEvaluaterServicer(), server
    )
    # 绑定端口并启动
    server.add_insecure_port('[::]:50051')
    server.start()
    print("gRPC Server started on port 50051...")

    try:
        # 保持服务器运行
        while True:
            time.sleep(_ONE_DAY_IN_SECONDS)
    except KeyboardInterrupt:
        server.stop(0)

if __name__ == '__main__':
    serve()
```
client.py
```python
import grpc
import time
import json
import concurrent.futures

# 导入生成的代码
from proto import zen_evaluate_pb2 as pb2
from proto import zen_evaluate_pb2_grpc as pb2_grpc

# 配置
SERVER_ADDRESS = 'localhost:50051'
NUM_CONCURRENT_CALLS = 1000 # 调用并发数
REQUEST_DATA = (
        '{"nodes":[{"type":"inputNode","content":{"schema":""},"id":"130f2ac6-1b88-4a6b-b92c-1f0e80270659","name":"request","position":{"x":185,"y":380}},{"type":"outputNode","content":{"schema":""},"id":"a8068913-5d34-440d-970c-b0f12ead4f4f","name":"response","position":{"x":850,"y":385}},{"type":"decisionTableNode","content":{"hitPolicy":"first","rules":[{"_id":"c3de0f22-2785-41c7-bc34-ef42f1b8a16d","_description":"Tip","3724c9e1-b57b-4d90-8430-6fd6caffef6d":"\\"white\\"","851b7ba6-a931-4bf9-948b-43419f879d9f":"\\"US\\"","2140250d-7c26-4290-a40b-a999ace5256c":"20"},{"_id":"88a66029-9a3b-476e-97af-b162201a3592","_description":"Pay","3724c9e1-b57b-4d90-8430-6fd6caffef6d":"\\"white\\"","851b7ba6-a931-4bf9-948b-43419f879d9f":"\\"UK\\"","2140250d-7c26-4290-a40b-a999ace5256c":"10"},{"_id":"da6be1f4-56cc-4c72-9c24-e3cb92e9fe24","_description":"Rob","3724c9e1-b57b-4d90-8430-6fd6caffef6d":"\\"black\\"","851b7ba6-a931-4bf9-948b-43419f879d9f":"\\"US\\"","2140250d-7c26-4290-a40b-a999ace5256c":"-20"},{"_id":"64e4ed54-ebd4-4572-a0e4-e27fa6c24740","_description":"Steel","3724c9e1-b57b-4d90-8430-6fd6caffef6d":"\\"black\\"","851b7ba6-a931-4bf9-948b-43419f879d9f":"\\"UK\\"","2140250d-7c26-4290-a40b-a999ace5256c":"-10"}],"inputs":[{"id":"3724c9e1-b57b-4d90-8430-6fd6caffef6d","name":"Customer color","field":"customer.color"},{"id":"851b7ba6-a931-4bf9-948b-43419f879d9f","name":"Shop country","field":"shop.country"}],"outputs":[{"id":"2140250d-7c26-4290-a40b-a999ace5256c","name":"Billing amount","field":"biiling.amount"}],"passThrough":false,"inputField":null,"outputPath":null,"executionMode":"single"},"id":"f0e14c2e-ac09-4d59-8729-abe3d3796908","name":"decisionTable1","position":{"x":535,"y":455}},{"type":"expressionNode","content":{"expressions":[{"id":"c54613c9-0d40-48f6-af89-91f60df2f8bd","key":"1","value":"true"},{"id":"d16acba5-763f-48fc-a176-de21c688d8c7","key":"2","value":"false"},{"id":"582383da-f261-4bad-8070-bfc0f517bd4f","key":"calc","value":"\'1\' + \'1\'"}],"passThrough":false,"inputField":null,"outputPath":null,"executionMode":"single"},"id":"5d8576d0-2dd6-477f-8072-a18b2bdec872","name":"expression1","position":{"x":535,"y":320}}],"edges":[{"id":"681dc2b6-a369-40c6-80d7-b5319b1cf749","sourceId":"130f2ac6-1b88-4a6b-b92c-1f0e80270659","type":"edge","targetId":"5d8576d0-2dd6-477f-8072-a18b2bdec872"},{"id":"9c8df7cb-9072-4446-a474-6e3a8fb3095a","sourceId":"130f2ac6-1b88-4a6b-b92c-1f0e80270659","type":"edge","targetId":"f0e14c2e-ac09-4d59-8729-abe3d3796908"},{"id":"bb2cea8b-7660-48a5-85f3-aa8edf6c9b17","sourceId":"f0e14c2e-ac09-4d59-8729-abe3d3796908","type":"edge","targetId":"a8068913-5d34-440d-970c-b0f12ead4f4f"},{"id":"9ebdd074-6b35-46d4-8efe-f0fdbc4d1291","sourceId":"5d8576d0-2dd6-477f-8072-a18b2bdec872","type":"edge","targetId":"a8068913-5d34-440d-970c-b0f12ead4f4f"}]}',
    '{"customer": {"color": "black"}, "shop": {"country": "US"}}'
)

def run(thread_id):
    # 建立与服务器的连接通道 (Channel)
    with grpc.insecure_channel(SERVER_ADDRESS) as channel:
        # 创建客户端存根 (Stub)，用于调用远程方法
        stub = pb2_grpc.ZenEvaluaterStub(channel)

        # 构造请求消息
        graph, input = REQUEST_DATA
        request = pb2.EvaluateRequest(graph = graph, input = input)

        # print(f"Client sending: {request.input}")

        try:
            # 调用 SayHello RPC 方法
            response = stub.CalcEvaluate(request)
            print(f"Thread {thread_id:03d}: Client received: {response.result}")
            if response.success:
                return response.result
            else:
                return None
        except grpc.RpcError as e:
            print(f"Thread {thread_id:03d}: An RPC error occurred: {e.details()}")
            return None

def test_grpc_concurrency():
    """
    使用 ThreadPoolExecutor 进行并发测试
    """
    start_all = time.time()

    # 1.创建线程池
    with concurrent.futures.ThreadPoolExecutor(max_workers=NUM_CONCURRENT_CALLS) as executor:
        # 2.提交任务：使用列表推导生成线程ID，并用map提交给执行器
        # map会阻塞，直到所有 Future 完成
        results = list(executor.map(run, range(NUM_CONCURRENT_CALLS)))

    end_all = time.time()
    total_time = end_all - start_all
    success_times = [t for t in results if t is not None]

    print("\n--- ✅ 并发测试结果总结 ---")
    print(f"并发请求数: {NUM_CONCURRENT_CALLS}")
    print(f"成功请求数: {len(success_times)}")
    print(f"总耗时 (包括所有等待时间): {total_time:.4f} 秒")

    if success_times:
        print(f"平均单次请求耗时: {total_time / len(success_times):.4f} 秒")

    # 计算吞吐量 (Requests Per Second)
    throughput = len(success_times) / total_time
    print(f"吞吐量 (QPS/RPS): {throughput:.2f} 请求/秒")

if __name__ == '__main__':
    try:
        test_grpc_concurrency()
    except Exception as e:
        print(f"An main error occurred: {e}")
        exit(-1)
```
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
server.js
```javascript
import grpc from '@grpc/grpc-js';
import protoLoader from '@grpc/proto-loader';
import { ZenEngine } from '@gorules/zen-engine';

const PROTO_PATH = './proto/zen_evaluate.proto';
const options = {
  keepCase: true,
  longs: String,
  enums: String,
  defaults: true,
  oneofs: true,
};
var packageDefinition = protoLoader.loadSync(PROTO_PATH, options);
const ZenEvaluaterProto = grpc.loadPackageDefinition(packageDefinition).zen_evaluate;

// ZenEngine 实例
const engine = new ZenEngine();

/**
 * gRPC Evaluate 方法实现
 * @param {grpc.ServerUnaryCall} call - gRPC 调用对象，包含请求数据
 * @param {grpc.sendUnaryData} callback - 回调函数，用于发送响应
 */
async function calcEvaluate(call, callback) {
  const { graph, input } = call.request;
  console.log(`request graph: ${graph}`);
  console.log(`request input: ${input}`);

  try {
    // 1.解析决策模型和上下文
    const decisionDefinition = JSON.parse(graph);
    const inputJson = JSON.parse(input);
    // 2.编译决策模型
    const decision = engine.createDecision(decisionDefinition);
    // 3.评估决策模型
    const result = await decision.evaluate(inputJson, { trace: true, });
    // 4.发送成功响应
    const response = {
      success: true,
      result: JSON.stringify(result),
    };
    callback(null, response);
  } catch (error) {
    console.error('评估错误：', error.message);
    // 5.发送错误响应
    const errRes = {
      success: false,
      result: error.message,
    };
    callback(null, errRes);
  }
}

/**
 * 启动gRPC服务
 */
function serve() {
  const server = new grpc.Server();
  server.addService(ZenEvaluaterProto.ZenEvaluater.service, {
    calcEvaluate: calcEvaluate,
  });

  const PORT = '0.0.0.0:50051';
  server.bindAsync(
    PORT,
    grpc.ServerCredentials.createInsecure(),
    (error, port) => {
      if (error) {
        console.error('服务器绑定失败：', error);
        return;
      }
      console.log(`Server running at ${PORT}`);
      server.start();
    },
  )
}

serve();
```
client.js
```javascript
const path = require('path');
const grpc = require('@grpc/grpc-js');
const protoLoader = require('@grpc/proto-loader');
const { Worker, isMainThread, parentPort, workerData, } = require('worker_threads');

// constants
const CONCURRENCY_COUNT = 1000;
const HOST = 'localhost:50051';
const PROTO_PATH = path.join(__dirname, 'proto/zen_evaluate.proto');

// Worker 线程函数
function runGrpcClientCall(graph, input) {
  // 1.加载 Proto
  const options = {
    keepCase: true,
    longs: String,
    enums: String,
    defaults: true,
    oneofs: true,
  };
  const packageDefinition = protoLoader.loadSync(PROTO_PATH, options);
  const ZenEvaluaterProto = grpc.loadPackageDefinition(packageDefinition).zen_evaluate;
  // 2.创建客户端
  const client = new ZenEvaluaterProto.ZenEvaluater(
    HOST,
    grpc.credentials.createInsecure()
  );
  // 3.执行gRPC调用
  return new Promise((resolve, reject) => {
    const request = { graph, input, };
    client.CalcEvaluate(request, (error, res) => {
      if (error) {
        // 报告gRPC错误
        reject(new Error(`gRPC ERROR: ${error.message}`));
        return;
      }
      resolve(res);
    });
  });
}
// Worker 线程入口
if (!isMainThread) {
  // Worker 线程收到主线程数据
  const { graph, input } = workerData;
  runGrpcClientCall(graph, input)
    .then(result => parentPort.postMessage({ status: 'SUCCESS', result }))
    .catch(error => parentPort.postMessage({ status: 'ERROR', error: error.message }));
}

// 主线程函数
async function runMainThread() {
  const startTime = process.hrtime.bigint();
  // 1.创建Worker任务数组
  const workerPromises = [];
  for (let i = 0; i < CONCURRENCY_COUNT; i++) {
    const graph = '{"nodes":[{"type":"inputNode","content":{"schema":""},"id":"130f2ac6-1b88-4a6b-b92c-1f0e80270659","name":"request","position":{"x":185,"y":380}},{"type":"outputNode","content":{"schema":""},"id":"a8068913-5d34-440d-970c-b0f12ead4f4f","name":"response","position":{"x":850,"y":385}},{"type":"decisionTableNode","content":{"hitPolicy":"first","rules":[{"_id":"c3de0f22-2785-41c7-bc34-ef42f1b8a16d","_description":"Tip","3724c9e1-b57b-4d90-8430-6fd6caffef6d":"\\"white\\"","851b7ba6-a931-4bf9-948b-43419f879d9f":"\\"US\\"","2140250d-7c26-4290-a40b-a999ace5256c":"20"},{"_id":"88a66029-9a3b-476e-97af-b162201a3592","_description":"Pay","3724c9e1-b57b-4d90-8430-6fd6caffef6d":"\\"white\\"","851b7ba6-a931-4bf9-948b-43419f879d9f":"\\"UK\\"","2140250d-7c26-4290-a40b-a999ace5256c":"10"},{"_id":"da6be1f4-56cc-4c72-9c24-e3cb92e9fe24","_description":"Rob","3724c9e1-b57b-4d90-8430-6fd6caffef6d":"\\"black\\"","851b7ba6-a931-4bf9-948b-43419f879d9f":"\\"US\\"","2140250d-7c26-4290-a40b-a999ace5256c":"-20"},{"_id":"64e4ed54-ebd4-4572-a0e4-e27fa6c24740","_description":"Steel","3724c9e1-b57b-4d90-8430-6fd6caffef6d":"\\"black\\"","851b7ba6-a931-4bf9-948b-43419f879d9f":"\\"UK\\"","2140250d-7c26-4290-a40b-a999ace5256c":"-10"}],"inputs":[{"id":"3724c9e1-b57b-4d90-8430-6fd6caffef6d","name":"Customer color","field":"customer.color"},{"id":"851b7ba6-a931-4bf9-948b-43419f879d9f","name":"Shop country","field":"shop.country"}],"outputs":[{"id":"2140250d-7c26-4290-a40b-a999ace5256c","name":"Billing amount","field":"biiling.amount"}],"passThrough":false,"inputField":null,"outputPath":null,"executionMode":"single"},"id":"f0e14c2e-ac09-4d59-8729-abe3d3796908","name":"decisionTable1","position":{"x":535,"y":455}},{"type":"expressionNode","content":{"expressions":[{"id":"c54613c9-0d40-48f6-af89-91f60df2f8bd","key":"1","value":"true"},{"id":"d16acba5-763f-48fc-a176-de21c688d8c7","key":"2","value":"false"},{"id":"582383da-f261-4bad-8070-bfc0f517bd4f","key":"calc","value":"\'1\' + \'1\'"}],"passThrough":false,"inputField":null,"outputPath":null,"executionMode":"single"},"id":"5d8576d0-2dd6-477f-8072-a18b2bdec872","name":"expression1","position":{"x":535,"y":320}}],"edges":[{"id":"681dc2b6-a369-40c6-80d7-b5319b1cf749","sourceId":"130f2ac6-1b88-4a6b-b92c-1f0e80270659","type":"edge","targetId":"5d8576d0-2dd6-477f-8072-a18b2bdec872"},{"id":"9c8df7cb-9072-4446-a474-6e3a8fb3095a","sourceId":"130f2ac6-1b88-4a6b-b92c-1f0e80270659","type":"edge","targetId":"f0e14c2e-ac09-4d59-8729-abe3d3796908"},{"id":"bb2cea8b-7660-48a5-85f3-aa8edf6c9b17","sourceId":"f0e14c2e-ac09-4d59-8729-abe3d3796908","type":"edge","targetId":"a8068913-5d34-440d-970c-b0f12ead4f4f"},{"id":"9ebdd074-6b35-46d4-8efe-f0fdbc4d1291","sourceId":"5d8576d0-2dd6-477f-8072-a18b2bdec872","type":"edge","targetId":"a8068913-5d34-440d-970c-b0f12ead4f4f"}]}';
    const input = '{"customer": {"color": "black"}, "shop": {"country": "US"}}';
    // 创建新的Worker
    const worker = new Worker(__filename, {
      workerData: {
        graph,
        input,
      },
    });
    // 2.收集 Worker 结果的 Promise
    const workerPromise = new Promise((resolve, reject) => {
      worker.on('message', (msg) => {
        if (msg.status === 'SUCCESS') {
          if (msg.result.success) {
            resolve(msg.result);
          } else {
            reject(new Error(msg.result.result));
          }
        } else {
          reject(new Error(msg.error));
        }
      });
      worker.on('error', reject);
      worker.on('exit', (code) => {
        if (code !== 0) reject(new Error(`Worker stopped with exit code ${code}`));
      });
    });
    workerPromises.push(workerPromise);
  }

  try {
    // 3.并发等待所有 Worker 完成
    const results = await Promise.all(workerPromises);
    const endTime = process.hrtime.bigint();
    const durationNs = endTime - startTime;
    // 转换为毫秒：10个线程结果在90 - 100ms
    const durationSec = Number(durationNs) / 1000000000;
    console.log('--- ✅ 并发测试结果总结 ---');
    console.log(`并发请求数: ${CONCURRENCY_COUNT}`);
    console.log(`总耗时 (包括所有等待时间): ${durationSec} 秒`);
    console.log(`平均单次请求耗时: ${Number(durationSec / CONCURRENCY_COUNT).toFixed(4)} 秒`);
    console.log(`吞吐量 (QPS/RPS): ${Number(CONCURRENCY_COUNT / durationSec).toFixed(2)} 请求/秒`);
    // results.forEach((res, index) => {
      // console.log(`result[${index}]: ${res.result}`);
    // });
  } catch(err) {
    console.error(err.message);
  } finally {
    // 进程退出
    process.exit(0);
  }
}
// 检查是否主线程，如果是，则启动执行
if (isMainThread) {
  runMainThread();
}
```
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
server/server.go
```golang
package main

import (
	"context"
	"encoding/json"
	"fmt"
	"log"
	"net"
	"os"
	"os/signal"
	"runtime"
	"syscall"
	"time"

	pb "app-algo-rule-engine-core/proto"

	"github.com/gorules/zen-go"
	"google.golang.org/grpc"
	"google.golang.org/grpc/keepalive"
)

type server struct {
	pb.UnimplementedZenEvaluaterServer
}

func (s *server) CalcEvaluate(ctx context.Context, req *pb.EvaluateRequest) (*pb.EvaluateResponse, error) {
	// 1. 解析JSON请求
	content := []byte(req.GetGraph())
	input := []byte(req.GetInput())
	// if err := json.Unmarshal([]byte(req.GetGraph()), &content); err != nil {
	// return nil, fmt.Errorf("invalid graph json: %w", err)
	// }
	// if err := json.Unmarshal([]byte(req.GetInput()), &input); err != nil {
	// return nil, fmt.Errorf("invalid input json: %w", err)
	// }

	// 2. 调用zen引擎计算
	engine := zen.NewEngine(zen.EngineConfig{})
	decision, err := engine.CreateDecision(content)
	if err != nil {
		log.Printf("failed to create decision: %v", err)
		return &pb.EvaluateResponse{Success: false, Result: fmt.Sprintf("%v", err)}, nil
	}
	opts := zen.EvaluationOptions{Trace: true}
	result, err := decision.EvaluateWithOpts(input, opts)
	if err != nil {
		log.Printf("failed to evaluate: %v", err)
		return &pb.EvaluateResponse{Success: false, Result: fmt.Sprintf("%v", err)}, nil
	}

	// 3. 序列化结果返回
	resultBytes, err := json.Marshal(result)
	if err != nil {
		log.Printf("failed to Marshal result: %v", err)
		return &pb.EvaluateResponse{Success: false, Result: fmt.Sprintf("%v", err)}, nil
	}

	// log.Printf("CalcEvaluate request success, result len: %d", len(resultBytes))
	return &pb.EvaluateResponse{Success: true, Result: string(resultBytes)}, nil
}

func main() {
	// 1. 限制CPU核心数
	runtime.GOMAXPROCS(1)

	// 2. 监听端口（修复为50051）
	lis, err := net.Listen("tcp", ":50051")
	if err != nil {
		log.Fatalf("failed to listen: %v", err)
	}

	// 3. gRPC服务配置（添加保活）
	kaPolicy := keepalive.ServerParameters{
		MaxConnectionIdle: 5 * time.Minute,
		Time:              2 * time.Minute,
		Timeout:           20 * time.Second,
	}
	serverOpts := []grpc.ServerOption{
		grpc.MaxConcurrentStreams(1000), // 最大并发流数
		grpc.KeepaliveParams(kaPolicy),  // 服务端保活策略
	}
	s := grpc.NewServer(serverOpts...)

	// 4. 注册服务
	pb.RegisterZenEvaluaterServer(s, &server{})

	// 5. 优雅退出
	sigChan := make(chan os.Signal, 1)
	signal.Notify(sigChan, syscall.SIGINT, syscall.SIGTERM)
	go func() {
		<-sigChan
		log.Println("Received shutdown signal, stopping server...")
		s.GracefulStop()
		os.Exit(0)
	}()

	// 6. 启动信息
	log.Printf("gRPC Server started:")
	log.Printf("  - CPU cores: %d", runtime.GOMAXPROCS(0))
	log.Printf("  - Listening on :50051")

	// 7. 启动服务
	if err := s.Serve(lis); err != nil {
		log.Fatalf("failed to serve: %v", err)
	}
}
```
client/client.go
```golang
package main

import (
	"context"
	"fmt"
	"log"
	"sync"
	"time"

	pb "app-algo-rule-engine-core/proto"

	"google.golang.org/grpc"
	"google.golang.org/grpc/credentials/insecure"
	"google.golang.org/grpc/keepalive"
)

var graph = `{"nodes":[{"type":"inputNode","content":{"schema":""},"id":"130f2ac6-1b88-4a6b-b92c-1f0e80270659","name":"request","position":{"x":185,"y":380}},{"type":"outputNode","content":{"schema":""},"id":"a8068913-5d34-440d-970c-b0f12ead4f4f","name":"response","position":{"x":850,"y":385}},{"type":"decisionTableNode","content":{"hitPolicy":"first","rules":[{"_id":"c3de0f22-2785-41c7-bc34-ef42f1b8a16d","_description":"Tip","3724c9e1-b57b-4d90-8430-6fd6caffef6d":"\"white\"","851b7ba6-a931-4bf9-948b-43419f879d9f":"\"US\"","2140250d-7c26-4290-a40b-a999ace5256c":"20"},{"_id":"88a66029-9a3b-476e-97af-b162201a3592","_description":"Pay","3724c9e1-b57b-4d90-8430-6fd6caffef6d":"\"white\"","851b7ba6-a931-4bf9-948b-43419f879d9f":"\"UK\"","2140250d-7c26-4290-a40b-a999ace5256c":"10"},{"_id":"da6be1f4-56cc-4c72-9c24-e3cb92e9fe24","_description":"Rob","3724c9e1-b57b-4d90-8430-6fd6caffef6d":"\"black\"","851b7ba6-a931-4bf9-948b-43419f879d9f":"\"US\"","2140250d-7c26-4290-a40b-a999ace5256c":"-20"},{"_id":"64e4ed54-ebd4-4572-a0e4-e27fa6c24740","_description":"Steel","3724c9e1-b57b-4d90-8430-6fd6caffef6d":"\"black\"","851b7ba6-a931-4bf9-948b-43419f879d9f":"\"UK\"","2140250d-7c26-4290-a40b-a999ace5256c":"-10"}],"inputs":[{"id":"3724c9e1-b57b-4d90-8430-6fd6caffef6d","name":"Customer color","field":"customer.color"},{"id":"851b7ba6-a931-4bf9-948b-43419f879d9f","name":"Shop country","field":"shop.country"}],"outputs":[{"id":"2140250d-7c26-4290-a40b-a999ace5256c","name":"Billing amount","field":"biiling.amount"}],"passThrough":false,"inputField":null,"outputPath":null,"executionMode":"single"},"id":"f0e14c2e-ac09-4d59-8729-abe3d3796908","name":"decisionTable1","position":{"x":535,"y":455}},{"type":"expressionNode","content":{"expressions":[{"id":"c54613c9-0d40-48f6-af89-91f60df2f8bd","key":"1","value":"true"},{"id":"d16acba5-763f-48fc-a176-de21c688d8c7","key":"2","value":"false"},{"id":"582383da-f261-4bad-8070-bfc0f517bd4f","key":"calc","value":"'1' + '1'"}],"passThrough":false,"inputField":null,"outputPath":null,"executionMode":"single"},"id":"5d8576d0-2dd6-477f-8072-a18b2bdec872","name":"expression1","position":{"x":535,"y":320}}],"edges":[{"id":"681dc2b6-a369-40c6-80d7-b5319b1cf749","sourceId":"130f2ac6-1b88-4a6b-b92c-1f0e80270659","type":"edge","targetId":"5d8576d0-2dd6-477f-8072-a18b2bdec872"},{"id":"9c8df7cb-9072-4446-a474-6e3a8fb3095a","sourceId":"130f2ac6-1b88-4a6b-b92c-1f0e80270659","type":"edge","targetId":"f0e14c2e-ac09-4d59-8729-abe3d3796908"},{"id":"bb2cea8b-7660-48a5-85f3-aa8edf6c9b17","sourceId":"f0e14c2e-ac09-4d59-8729-abe3d3796908","type":"edge","targetId":"a8068913-5d34-440d-970c-b0f12ead4f4f"},{"id":"9ebdd074-6b35-46d4-8efe-f0fdbc4d1291","sourceId":"5d8576d0-2dd6-477f-8072-a18b2bdec872","type":"edge","targetId":"a8068913-5d34-440d-970c-b0f12ead4f4f"}]}`
var input = `{"customer": {"color": "black"}, "shop": {"country": "US"}}`

// 统计结果
type Stats struct {
	success int
	fail    int
	total   time.Duration
	mu      sync.Mutex // 保护统计字段的并发写
}

func (s *Stats) addSuccess(d time.Duration) {
	s.mu.Lock()
	defer s.mu.Unlock()
	s.success++
	s.total += d
}

func (s *Stats) addFail() {
	s.mu.Lock()
	defer s.mu.Unlock()
	s.fail++
}

func sendRequest(client pb.ZenEvaluaterClient, wg *sync.WaitGroup, idx int, stats *Stats) {
	defer wg.Done()

	start := time.Now()
	ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
	defer cancel()

	req := &pb.EvaluateRequest{
		Graph: graph,
		Input: input,
	}
	_, err := client.CalcEvaluate(ctx, req)
	duration := time.Since(start)
	if err != nil {
		stats.addFail()
		log.Printf("Request %d failed: %v (耗时: %v)", idx, err, duration)
		return
	}
	stats.addSuccess(duration)
}

func main() {
	// 配置
	totalRequests := 1000

	// gRPC连接配置（保活+长连接）
	// 连接复用：全局创建 1 个 gRPC 连接，所有 goroutine 复用该连接（避免每次请求新建连接的开销）；
	// 保活机制：防止长连接被网关 / 操作系统断开，提升高并发下的连接稳定性；
	conn, err := grpc.NewClient(
		"localhost:50051",
		grpc.WithTransportCredentials(insecure.NewCredentials()),
		grpc.WithKeepaliveParams(keepalive.ClientParameters{
			Time:                10 * time.Second, // 每10秒发保活包
			Timeout:             5 * time.Second,
			PermitWithoutStream: true, // 无流时也允许保活
		}),
	)
	if err != nil {
		log.Fatalf("did not connect: %v", err)
	}
	defer conn.Close()
	client := pb.NewZenEvaluaterClient(conn)

	// 统计初始化
	stats := &Stats{}
	var wg sync.WaitGroup

	// 全并发执行
	start := time.Now()
	for i := 0; i < totalRequests; i++ {
		wg.Add(1)
		go sendRequest(client, &wg, i, stats)
	}

	wg.Wait()
	totalDuration := time.Since(start)

	// 统计输出
	fmt.Printf("--- ✅ 并发测试结果总结 ---\n")
	fmt.Printf("并发请求数: %d\n", totalRequests)
	fmt.Printf("成功请求数: %d\n", stats.success)
	// fmt.Printf("失败数: %d\n", stats.fail)
	// fmt.Printf("成功率: %.2f%%\n", float64(stats.success)/float64(totalRequests)*100)
	fmt.Printf("总耗时 (包括所有等待时间): %v\n", totalDuration.Seconds())
	if stats.success > 0 {
		avgDuration := stats.total / time.Duration(stats.success)
		fmt.Printf("平均单次请求耗时: %v\n", avgDuration.Seconds())
	}
	fmt.Printf("吞吐量 (QPS/RPS): %.2f 请求/秒\n", float64(stats.success)/totalDuration.Seconds())
}
```
5. 执行代码；
```sh
go run server/server.go # 启动gRPC服务端
go run server/client.go # 启动gRPC客户端
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

## gRPC示例代码rust(1.92)
1. 安装必要的rust包:
Cargo.toml
```toml
[package]
name = "jdm-grpc-rust"
version = "0.1.0"
edition = "2024"

[dependencies]
tracing = "0.1"
tracing-subscriber = "0.3"
zen-engine = { version = "0" }
serde = { version = "1", features = ["derive"] }
serde_json = "1"
tonic = "0.14"
prost = "0.14"
tokio = { version = "1.48", features = ["macros", "rt-multi-thread"] }
tonic-prost = "0.14"
futures = "0.3"

[build-dependencies]
tonic-prost-build = "0.14"
```
2. 定义proto文件：proto/zen_evaluate.proto
```proto
syntax = "proto3";

package zen_evaluate;

option java_multiple_files = true;
option java_package = "io.grpc.zen_evaluater";
option java_outer_classname = "ZenEvaluateProto";

// service definition
service ZenEvaluater {
  rpc CalcEvaluate (EvaluateRequest) returns (EvaluateResponse) {}
}

// request obj
message EvaluateRequest {
  string graph = 1;
  string input = 2;
}

// response
message EvaluateResponse {
  bool success = 1;
  string result = 2;
}
```
3. 服务端代码：
src/rules.rs
```rust
use futures;
use std::sync::Arc;
use tracing::error;
use zen_engine::model::DecisionContent;
use zen_engine::{Decision, DecisionEngine, DecisionGraphResponse, EvaluationOptions, EvaluationTraceKind, Variable};
use serde_json::value::Serializer as JsonSerializer;

/**
 * Decision 是不可 Send 的，导致出现 sync 相关的错误
 * 使用 block_on 原地执行代码解决此问题
 */
pub fn rules_evaluate_blocking(
    input: String,
    graph: String,
    engine_clone: Arc<DecisionEngine>,
) -> Result<serde_json::Value, serde_json::Value> {
    let input_json: Variable = serde_json::from_str(&input).map_err(|_| {
        serde_json::json!({
            "type": "InternalError",
            "source": "parse request input error",
        })
    })?;
    let decision_content: DecisionContent = serde_json::from_str(&graph).map_err(|_| {
        serde_json::json!({
            "type": "InternalError",
            "source": "parse request graph error",
        })
    })?;
    let decision: Decision = engine_clone.create_decision(decision_content.into());
    let decision_opts: EvaluationOptions = EvaluationOptions {
        trace: true,
        max_depth: 10,
    };
    // 使用 block_on 原地执行 async 代码，将其转为同步阻塞调用
    // 这样 decision 对象就不需要跨越线程边界
    let result = futures::executor::block_on(async {
        decision.evaluate_with_opts(input_json, decision_opts).await
    });

    match result {
        Ok(r) => serde_json::to_value(&r).map_err(|_| {
            serde_json::json!({
                "type": "InternalError",
                "source": "parse evaluate result error",
            })
        }),
        Err(e) => {
            // 因为 EvaluationError 实现了 Serialize，手动转换为 Value
            let mode = EvaluationTraceKind::Default;
            let eval_err = *e;
            let error_value = eval_err.serialize_with_mode(JsonSerializer, mode)
                .unwrap_or_else(|_| serde_json::json!({ "type": "InternalError", "source": "Failed to serialize EvaluationError" }));
            error!("evaluate process error");
            Err(error_value)
        }
    }
}
```
src/main.rs
```rust
use std::net::SocketAddr;
use std::sync::Arc;

use tonic::{Request, Response, Status, transport::Server};
use tracing::info;
use zen_engine::DecisionEngine;

use zen_evaluate::zen_evaluater_server::{ZenEvaluater, ZenEvaluaterServer};
use zen_evaluate::{EvaluateRequest, EvaluateResponse};

pub mod rules;

pub mod zen_evaluate {
    tonic::include_proto!("zen_evaluate");
}

// My service
#[derive(Default)]
pub struct MainZenEvaluater {
    engine: Arc<DecisionEngine>,
}

#[tonic::async_trait]
impl ZenEvaluater for MainZenEvaluater {
    async fn calc_evaluate(
        &self,
        request: Request<EvaluateRequest>,
    ) -> Result<Response<EvaluateResponse>, Status> {
        // DONE: evaluate
        let req_obj = request.into_inner();
        let input_owned = req_obj.input;
        let graph_owned = req_obj.graph;
        let engine_clone = self.engine.clone();
        // 将计算任务放入 blocking 线程池
        // 这样即使 rules_evaluate 内部的数据不是 Send，只要它能在当前线程运行即可
        // 注意：这里需要 rules::rules_evaluate 变成同步函数，或者我们在 spawn_blocking 里调用同步逻辑
        let handle_result = tokio::task::spawn_blocking(move || {
            // 这里调用同步版本的 evaluate
            // 调用修改过的 rules::rules_evaluate_blocking
            rules::rules_evaluate_blocking(input_owned, graph_owned, engine_clone)
        })
        .await;
        let handle_result = match handle_result {
            Ok(r) => r,
            Err(e) => {
                let res = EvaluateResponse {
                    success: false,
                    result: serde_json::json!({"type": "InternalError", "source": format!("Join error: {}", e)}).to_string(),
                };
                return Ok(Response::new(res));
            }
        };

        let handle_result = match handle_result {
            Ok(r) => r,
            Err(e) => {
                let res = EvaluateResponse {
                    success: false,
                    result: e.to_string(),
                };
                return Ok(Response::new(res));
            }
        };
        let response = EvaluateResponse {
            success: true,
            result: serde_json::json!(handle_result).to_string(),
        };
        Ok(Response::new(response))
    }
}

#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    if std::env::var_os("RUST_LOG").is_none() {
        unsafe {
            std::env::set_var("RUST_LOG", "info");
        }
    }
    tracing_subscriber::fmt::init();

    // server
    let addr: SocketAddr = "[::1]:50051".parse().unwrap();
    let engine = Arc::new(DecisionEngine::default());
    let evaluater = MainZenEvaluater {
        engine: engine.clone(),
    };
    info!("🚀 Starting grpc server at {addr:?}...");

    Server::builder()
        .add_service(ZenEvaluaterServer::new(evaluater))
        .serve(addr)
        .await?;

    Ok(())
}
```
build.rs
```rust
fn main() -> Result<(), Box<dyn std::error::Error>> {
    tonic_prost_build::configure().compile_protos(&["proto/zen_evaluate.proto"], &["proto"])?;
    Ok(())
}
```
4. 运行服务端：
```bash
cargo run # 启动gRPC服务端，可设置启动参数：RUST_LOG=warn cargo run 打印日志，默认info级别
```

## 测试结果（CPU：R9 9900x）
| Client\Server  | Python v3.11 | nodejs v20.19 | go v1.25 | rust v1.91 |
| :------------- | :----------- | :------------ | :------- | :--------- |
| Python v3.11   | 并发请求数: 1000<br/>成功请求数: 1000<br/>总耗时 (包括所有等待时间): 0.5770 秒<br/>平均单次请求耗时: 0.0006 秒<br/>吞吐量 (QPS/RPS): 1733.19 请求/秒 | 并发请求数: 1000<br/>成功请求数: 1000<br/>总耗时 (包括所有等待时间): 0.2201 秒<br/>平均单次请求耗时: 0.0002 秒<br/>吞吐量 (QPS/RPS): 4543.62 请求/秒 | 并发请求数: 1000<br/>成功请求数: 1000<br/>总耗时 (包括所有等待时间): 0.1949 秒<br/>平均单次请求耗时: 0.0002 秒<br/>吞吐量 (QPS/RPS): 5131.08 请求/秒 | - |
| nodejs v20.19  | 并发请求数: 1000<br/>成功请求数: 1000<br/>总耗时 (包括所有等待时间): 5.466387314 秒<br/>平均单次请求耗时: 0.0055 秒<br/>吞吐量 (QPS/RPS): 182.94 请求/秒 | 并发请求数: 1000<br/>成功请求数: 1000<br/>总耗时 (包括所有等待时间): 5.30989529 秒<br/>平均单次请求耗时: 0.0053 秒<br/>吞吐量 (QPS/RPS): 188.33 请求/秒 | 并发请求数: 1000<br/>成功请求数: 1000<br/>总耗时 (包括所有等待时间): 6.830945164 秒<br/>平均单次请求耗时: 0.0068 秒<br/>吞吐量 (QPS/RPS): 146.39 请求/秒 | - |
| go v1.25       | 并发请求数: 1000<br/>成功请求数: 1000<br/>总耗时 (包括所有等待时间): 601.318751ms<br/>平均单次请求耗时: 372.216699ms<br/>吞吐量 (QPS/RPS): 1663.01 请求/秒 | 并发请求数: 1000<br/>成功请求数: 1000<br/>总耗时 (包括所有等待时间): 233.055916ms<br/>平均单次请求耗时: 149.514489ms<br/>吞吐量 (QPS/RPS): 4290.82 请求/秒 | 并发请求数: 1000<br/>成功请求数: 1000<br/>总耗时 (包括所有等待时间): 70.9154ms<br/>平均单次请求耗时: 0.071ms<br/>吞吐量 (QPS/RPS): 14101.31 请求/秒 | 并发请求数: 1000<br/>成功请求数: 1000<br/>总耗时 (包括所有等待时间): 89.5675ms<br/>平均单次请求耗时: 0.076ms<br/>吞吐量 (QPS/RPS): 11164.76 请求/秒 |

客户端为nodejs时，客户端为性能瓶颈，舍弃该测试结果；  
客户端为python时，服务端表现出的并发性能各不相同，其中server-go和server-nodejs表现基本在server-python的吞吐量的2倍以上。  
客户端为golang时，服务端表现出的并发性能差异明显，其中server-go吞吐量最大（14,101），server-rust略小一点（11,165），server-nodejs其次（4,290），sever-python最差（1,663）。  
综上所述，在客户端不瓶颈的情况下，服务端为go语言的并发性能最佳。  
