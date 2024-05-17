---
title: AI浅谈及如何手动搭建本地大模型
date: 2024-05-17 11:02:53
categories: tech
tags: [AI,linux, 大模型]
---
# AI浅谈及如何手动搭建本地大模型
## 背景介绍
1. AI是什么？  
ChatGPT?
2. AI发展简述  
早期的AI机器人：微博微软小娜、小冰  
可以进行简单的对话，短文本支持，长文本和上下文对话能力比较差。  
中期：各种语音助手，Siri，小度，小爱音响  
当前：ChatGPT，文心一言，车载语音助手，copilot等  
AI带来的经济收益  
OpenAI: 市值在10个月内翻三倍，达800亿美元。  
芯片行业：NVIDIA: 近两年成为第七个市值挺进万亿美元的公司，2023年更是突破2万亿美元，成为全球第三大市值上市公司，仅次于微软和苹果。  
AMD: 市值大涨  
……

3. AI使用场景  
**文本对话**(chatgpt, 文心一言, 云网融合)  
文生图(stable diffusion)，还存在少量问题  
文生视频(OpenAI sora)，存在比较大的问题  
实例：NVIDIA DLSS，游戏内动态帧生成技术，改善低性能显卡的游戏表现。  

4. AI必要的一些基础知识  
**硬件平台：**  
Q: 为什么不用中央处理器（CPU），而使用图形处理器（GPU）？  
**CUDA核心**（并行处理单元）可进行大量并行计算任务  
Tensor Core: AI加速核心  
从CPU和GPU来解释  
CPU：14900K，8+16，总共24核心32线程，价格4000左右  
GPU：RTX 4070，5888个CUDA核心，价格4000左右；  
4090具备14592个CUDA核心，价格14000左右  
如此多的核心可同时进行大量计算任务，机器学习模型涉及大规模的矩阵运算和深度神经网络层级计算。  
显存(GPU Memory)：用于一次性加载大模型的内存，一般与成品GPU绑定，GPU核心与BIOS程序固定了显存大小，显存越大能加载的大模型参数(见下)越大。  
民用级：4070 12G GDDR6X，4090 24G GDDR6X  
专业级：H100 80G，A6000 48G，NVLink(256xH100)  
软件：  
Python：为什么使用脚本语言？免编译随时修改随时运行调试，科学计算库丰富，底层可调用高性能语言(C/C++)；  
tensor/pytorch:张量(矢量，矩阵等，tensor)，tensor库  
**transformer**: Hugging Face的一个NLP(自然语言处理)包，可加载绝大部分预训练模型  
![](https://huggingface.co/datasets/huggingface-course/documentation-images/resolve/main/en/chapter1/transformers_blocks.svg)
![](https://huggingface.co/datasets/huggingface-course/documentation-images/resolve/main/en/chapter1/transformers.svg)

## 大语言模型(LLM——Large Language Model)
1. 自然语言与机器语言？  
机器语言：0 1(汇编最接近)  
编程语言：词法、语法、语义，编译为二进制(编译原理)  
自然语言：词法、语法、语义，推理，思考，情绪，感情，隐藏含义……(日常生活场景，例子)  
2. 大语言模型是什么？  
大模型提供了推理能力，支持多种任务，并已经对特定的任务进行了预训练。  
3. 常见的LLM大语言模型  
国内：Chatglm系列(1, 2, 3)，通义千问-QWen，Baichuan2等  
国外：Meta-llama系列，OpenAI-ChatGPT，Google-Gemma，  

4. 一些专业术语  
Prompt：提示词，对人说人话，对AI说AI话，(实例：string，字符串 or 细绳？)  
7B(Billion 10亿)参数: 存储知识和信息的变量，参数越多，记住的知识越多，输出结果更准确  
Tokens: 模型可以理解和生成的最小意义单位(一个单词，一个标点等)，越大可处理及可生成的内容越长  
Hugging Face: 大模型仓库(科学上网)  

## 如何在本地电脑上搭建大语言模型
## 准备工作
### 硬件环境
CPU：无具体要求，（R7 5800x）  
内存：16G以上，推荐32G，（48G）  
**GPU**：最好16G显存以上NVIDIA显卡(目前大模型基本都基于N卡CUDA)，（RTX 4060Ti 16G）  
硬盘：100G以上，最好1T以上，（2T SSD）  

### 软件环境
环境(本人)：
系统：Linux，(Archlinux，包管理器pacman)   
软件：NVIDIA驱动程序，CUDA  
python3.11(推荐可选：Anaconda/Miniconda管理环境)  
网络：**科学上网**（下载大模型必要条件）  
```shell
(optional) conda create -n llmLearning python=3.11 # 可选：新建conda环境
(optional) conda activate llmLearning # 可选：激活conda环境
pip config set global.extra-index-url "https://mirror.sjtu.edu.cn/pypi/web/simple" # 配置pip仓库镜像
pip install torch transformers # 安装必要的包
```
## transformers功能
| 任务 | 描述 | 模态 | Pipeline |
| ---- | --- | ---- | -------- |
| 文本分类 | 为给定的文本序列分配一个标签 | NLP | pipeline(task=“sentiment-analysis”)
| 文本生成 | 根据给定的提示生成文本 | NLP | pipeline(task=“text-generation”) |
| 命名实体识别 | 为序列里的每个 token 分配一个标签（人, 组织, 地址等等） | NLP | pipeline(task=“ner”) |
| 问答系统 | 通过给定的上下文和问题, 在文本中提取答案 | NLP | pipeline(task=“question-answering”) |
| 掩盖填充 | 预测出正确的在序列中被掩盖的token | NLP | pipeline(task=“fill-mask”) |
| 文本摘要 | 为文本序列或文档生成总结 | NLP | pipeline(task=“summarization”) |
| 文本翻译 | 将文本从一种语言翻译为另一种语言 | NLP | pipeline(task=“translation”) |
| 图像分类 | 为图像分配一个标签 | Computer vision | pipeline(task=“image-classification”) |
| 图像分割 | 为图像中每个独立的像素分配标签（支持语义、全景和实例分割） | Computer vision | pipeline(task=“image-segmentation”) |
| 目标检测 | 预测图像中目标对象的边界框和类别 | Computer vision | pipeline(task=“object-detection”) |
| 音频分类 | 给音频文件分配一个标签 | Audio | pipeline(task=“audio-classification”) |
| 自动语音识别 | 将音频文件中的语音提取为文本 | Audio | pipeline(task=“automatic-speech-recognition”) |
| 视觉问答 | 给定一个图像和一个问题，正确地回答有关图像的问题 | Multimodal | pipeline(task=“vqa”) |

实例(文本分类)：
```python
from transformers import pipeline

def sentimentAnalysisDemo():
    classifier = pipeline(task="sentiment-analysis") # ,device=0 使用GPU设备加载模型，可以使用model参数分配一个模型，不传transformers会自动分配一个该任务的模型
    results = classifier(
        ["I was inspired by Kobe so much.", "He has left four years!"]
    )
    for result in results:
        print(f"标签: {result['label']}, 权重分数: {round(result['score'], 4)}")

if __name__ == "__main__":
    sentimentAnalysisDemo()
```

实例(文本生成)：
```python
from transformers import pipeline

def textGenerationDemo():
    generator = pipeline(task="text-generation")
    results = generator(
        "Today's weather is so good, So I want", # 说话说一半，让AI根据上下文补充
        num_return_sequences=1,
        max_length=40
    )
    print(results)

if __name__ == "__main__":
    textGenerationDemo()
```

### 本地搭建
**注：以下实例推荐使用GPU，不推荐使用CPU进行计算**  
**以对话模型为例本地搭建**  
1. 安装git及git lfs (大模型通常文件比较大，需要让git支持大文件)：  
```shell
pacman -S git git-lfs
git lfs install
```
2. Hugging Face下载大模型  
大模型大小基本都在20G以上，  
也可以启动时下载，不过会很慢，最好提前下载，这里使用Google开源的gemma-2b-it模型  
google/gemma-2b-it大模型需要申请使用权限，请先去 [huggingface](https://huggingface.co/google/gemma-2b-it) 申请权限  
```shell
git clone https://huggingface.co/google/gemma-2b-it /datad/llm/models/google/gemma-2b-it # 示例大模型存放地址
```
3. 编写python代码加载调用大模型
编写python代码加载大模型，并调用大模型基础对话能力
```shell
pip install accelerate # 安装必要模块
```
实例：
```python
from transformers import AutoTokenizer, AutoModelForCausalLM
import torch

model_name_or_path = "/datad/llm/models/google/gemma-2b-it" # 如果设置为 google/gemma-2b-it 会自动从huggingface上下载模型，时间较长
tokenizer = AutoTokenizer.from_pretrained(model_name_or_path)
model = AutoModelForCausalLM.from_pretrained(
    model_name_or_path,
    device_map="auto", # 自动分配GPU设备
    torch_dtype=torch.bfloat16
)

input_text = "写一首关于机器学习的诗歌"
input_ids = tokenizer(input_text, return_tensors="pt").to("cuda")

outputs = model.generate(**input_ids)
print(tokenizer.decode(outputs[0]))
```

### 使用开源项目提供更多功能
必要性：需要对外提供API，对外提供多种服务，支持外部接入训练微调，每个都手写太麻烦  
项目：[LLaMA-Factory](https://github.com/hiyouga/LLaMA-Factory)  
项目简介：
- 支持多种大模型
- 支持提供OpenAI风格的API接口服务
- 支持多种微调训练方法(增量预训练，指令监督微调等)
- 支持浏览器中进行对话及微调训练，也提供相应的脚本
- 支持多种算法：LoRA，QLoRA，Agent等，16G及以下显存推荐QLoRA+8精度

**搭建步骤**  
1. 安装依赖  
```shell
git clone https://github.com/hiyouga/LLaMA-Factory.git
conda create -n llama_factory python=3.11
conda activate llama_factory
cd LLaMA-Factory
pip install -e .
```

2. 启动项目  
```shell
python src/train_web.py
```
浏览器中查看体验：[10.77.23.17:7860](http://10.77.23.17:7860)  
大模型path：`/datad/llm/models/google/gemma-2b-it`  

