
本周：
1、hot100    
2、hello-Agents
3、总结一个面经（15个问题左右，背下来）
4、开始手撕之前的代码。

每周要有可以输出的东西，可以讲出来的。

第一周：hello_agents   7、8、9、10
第二周：hello_agents    11、12、13、14、15  

最近要学会的。45天
	算法：
		hot100
	agent：
		hello-agents（难度有点大，慢慢来），以及一些agenticRL
		building AI Agents in Pure Python
		Camel[‍​﻿‬⁠​​​⁠⁠‬​﻿‍‌​​‌﻿‍⁠​​‍﻿​​​​​⁠​​​​​​⁠​​‍‌​‬​‬​​‍‬Handy Multi-Agent Tutorial - 飞书云文档](https://my.feishu.cn/docx/AF4XdOZpIo6TOaxzDK8cxInNnCe)
		acwing（项目）
		TradingAgents（项目）
	rag：
		all in rag代码手撕
	train:（GRPO\ppo\dpo\GSPO-----）学会手撕
		minimind(手撕)
		Tinyzero[TinyZero最详细复现笔记（二）：VeRL框架与PPO训练细节 - 知乎](https://zhuanlan.zhihu.com/p/1903855264207200959)
	手撕：
		1、transformer架构，MHA，gpt\llama\qwen等常见架构
		2、ReActAgent\ReflectionAgent\PlanAndSlveAgent
		3、lora微调
		4、tiny-universe
	
一些手搓
[datawhalechina/tiny-universe: 《大模型白盒子构建指南》：一个全手搓的Tiny-Universe](https://github.com/datawhalechina/tiny-universe)
1. 手写图像生成模型--Tiny Diffusion
2. 深入剖析大模型原理——Qwen Blog
3. 逐步预训练一个手搓大模型——Tiny Llama3
4. 如何评估你的大模型——Tiny Eval
5. 纯手工搭建 RAG 框架——Tiny RAG
6. 手搓一个最小的 Agent 系统——Tiny Agent
7. 深入理解大模型基础——Tiny Transformer
8. 手搓一个基本的 GraphRAG 系统——Tiny GraphRAG






```
# MTP  Multi-token-prediction

  
  

# 问题1：模型是非常依赖perfect context，否则模型会错的越来越离谱。  

# 问题2：模型每次都只生成一个token，会导致模型是一个非常短视的模型，只能看到眼 前的输出 .它的planning的能力会很差。

# 问题3：training signal比较弱，提供更多监督信息

  
  

#自然就会想到，每次不预测一个token，每次预测多个token。


# 强化学习可以让模型在没见过的任务上表现可以

#supervised learing学习，不可能超过人类，有上限（因为你的label就是来自人类的

# 只能达到接近人类的水平）

# 但是强化学习可以超越人类的水平



# 学习一下 Andrej Karpathy



# 学习方法

# 1.平时得积累, 大量的泛读很多paper, 我就是扫一眼标题和摘要, 知道这篇paper在干什么就可以了, 我用X追踪领域的最新发展 2.如果遇到新的领域, 我之前完全不了解的, 就用google scholar找几篇领域内citation

# 最高的先读一下, 看看这些paper related work里面提到哪些paper, 然后再顺藤摸瓜



# deepseek r1是如何让大语言模型学会自我思考的

# LLM思考10s为什么比思考1s得到的效果好.

# cot 是如何诞生的


```

训练一个神经网络，就是找到一个拟合函数。（world is a function）（找到一个规则）（拟合一一个世界）

大语言模型本质上还是一个分类问题。分的是各个字类别。

attention核心是加权求和。注意力大就是权重大。找出最相关的信息。


在pytorch中是一个层时，就要继承nn.Module类.要有__init__  和forward方法
 
 nn.parameter（）使参数可训练
 
 
 
 




这个项目是DeepSeek R1-Zero的最小复制（也要学习一下）
[Jiayi-Pan/TinyZero: Minimal reproduction of DeepSeek R1-Zero](https://github.com/Jiayi-Pan/TinyZero)
		

参考飞书云文档，肯定要过一遍

背，总结 八股面经


做一个agenticRL项目

最好把小红书和微信公众号收藏都看一看



强化学习公式：[大语言模型RLHF全链路揭秘：从策略梯度、PPO、GAE到DPO的实战指南](https://mp.weixin.qq.com/s/S72LO26IsZ8AED8sQKIWnQ)

EZ_encoder的社区里面内容知识挺多的尤其是CS336共学项目



前沿信息
## paper

[https://arxiv.org/](https://arxiv.org/)  
[https://huggingface.co/papers](https://huggingface.co/papers) （主要看看热点）

## blog

[https://lilianweng.github.io/](https://lilianweng.github.io/)  
[https://blog.langchain.com/](https://blog.langchain.com/)  
[https://www.anthropic.com/engineering](https://www.anthropic.com/engineering)  
[https://cognition.ai/blog/1](https://cognition.ai/blog/1)  
[https://sierra.ai/blog](https://sierra.ai/blog)  
[https://sebastianraschka.com/](https://sebastianraschka.com/)  
[https://kexue.fm/](https://kexue.fm/)  
[https://yuanchaofa.com/](https://yuanchaofa.com/) （我自己的）

## Twitter

[https://x.com/karpathy](https://x.com/karpathy)  
[https://x.com/_jasonwei](https://x.com/_jasonwei)  
[https://x.com/sama](https://x.com/sama)  
[https://x.com/huybery](https://x.com/huybery)  
[https://x.com/Francis_YAO](https://x.com/Francis_YAO)_  
[https://x.com/ShunyuYao12](https://x.com/ShunyuYao12)


对于技术所有的知识开头都要讲，其解决了什么问题，是怎么解决的，有没有更好的方法。

1、leetcode   hot100.每天10道题。
2、手撕题目全部做了
3、做几个开源的项目
4、八股背一下
5、self llm   
6、以及其他要写的代码


以下都要会
- [ ] 1、蒸馏
- [ ] 2、llama
- [ ] 3、qwen
- [ ] 4、deepseek
- [ ] 5、clip
- [ ] 6、llama factory
- [ ] 7、peft
- [ ] 8、paged attention
- [ ] 9、显存计算
- [ ] 10、hello agent
- [ ] 11、熟练掌握dify和coze

以下都要会手撕
**hot-100**
- [ ] 1、lora
- [ ] 2、dpo
- [ ] 3、ppo
- [ ] 4、grpo
- [ ] 5、spo
- [ ] 6、rlhf
- [ ] 7、transformer
- [ ] 8、clip
- [ ] 9、bert
- [ ] 10、flash attention
- [ ] 11、kv-cache
- [ ] 12、MQA / GQA
- [ ] 13、MHA
- [ ] 14、旋转位置编码
- [ ] 15、moe






infer （vllm的小复现）
[Nano-vLLM](https://github.com/liguodongiot/nano-vllm)

cs336
追更最先的、原本的大厂公司发布的blog、各种技术报告




这个项目包含
1、模型，预训练、后训练、监督微调、偏好对齐、评估、量子化
2、向量存储、rag、高级rag、agent、推理、部署
[mlabonne/LLM课程：面向进入大型语言模型（LLM）的课程，配有路线图和Colab笔记本。](https://github.com/mlabonne/llm-course)


这个仓库学习起来很全
[liguodongiot/llm-action: 本项目旨在分享大模型相关技术原理以及实战经验（大模型工程化、大模型应用落地）](https://github.com/liguodongiot/llm-action?tab=readme-ov-file)

这个里面有很多大模型应用的项目（可以用来找项目）
[Shubhamsaboo/awesome-llm-apps：包含 OpenAI、Anthropic、Gemini 和开源模型的优秀 LLM 应用集，包含 AI 代理和 RAG。](https://github.com/Shubhamsaboo/awesome-llm-apps?tab=readme-ov-file)





kaggle竞赛
[【kaggle大模型】快速上手kaggle平台_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1vVayeREVe/?vd_source=8d312f93d604a2ab845de30b9eaed0ed)








这个人的视频质量很高很高很高
[EZ-Encoder投稿视频-EZ-Encoder视频分享-哔哩哔哩视频](https://space.bilibili.com/3546829121652889/upload/video)



苏神苏剑林
Transformer架构的深刻理解以及提出的**旋转位置编码（RoPE）**和**RoFormer模型**而闻名。
[科学空间|Scientific Spaces](https://kexue.fm/)

图解各种系列（如vllm、flash attention、位置编码、ppo、并行训练等等）
[猛猿 - 知乎](https://www.zhihu.com/people/lemonround/posts)

亚马逊首席应用科学家
[尹尤金](https://eugeneyan.com/)


[周星星 - 知乎](https://www.zhihu.com/people/zhoujx4/posts)deepresearch系列


张俊林[张俊林 - 知乎](https://www.zhihu.com/people/zhang-jun-lin-76/posts)


GLM技术报告[GLM-4.5: Agentic, Reasoning, and Coding (ARC) Foundation Models](https://arxiv.org/pdf/2508.06471)


![[Pasted image 20260122201128.png]]
openai     Lilian's Blog 
[由LLM驱动的自主代理 |小洛格](https://lilianweng.github.io/posts/2023-06-23-agent/)


anthropic
[工程 \ 人性](https://www.anthropic.com/engineering)
[Building Effective AI Agents \ Anthropic](https://www.anthropic.com/engineering/building-effective-agents)
