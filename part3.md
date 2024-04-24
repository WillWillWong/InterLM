## RAG介绍

**RAG 是什么？**

`Retrieval Agumented Generation` 结合检索和生成技术，旨在通过利用外部知识库来增强LLM的性能。通过检索和用户输入相关的信息片段，并结合这些信息来生成更准确丰富的回答。

**RAG的工作原理？**

索引：知识源分割存储，建立向量数据库 Vector-DB——数据存储、相似性检索以及向量表示的优化

检索：向量库检索，找到相关文本库

生成：检索和原始问题一起送入 LLM 进行生成。

RAG工作原理

**RAG 的范式：**

1. Naive RAG : 最原始的三个基础部分，用于问答系统、信息检索
2. Advanced RAG: 检索前对用户问题路由扩展或者重写，检索后对信息重排序融合等处理，用于摘要生成以及内容推荐等。
3. Modular RAG:用于多模态任务以及对话系统。基础部分和其它功能进行模块化应用。

**RAG 常见的优化方法：**

嵌入优化：结合稀疏和密集检索以及多任务(Vector-DB)

索引优化：细粒度分割和元数据(Vector-DB)

查询优化：查询扩展、转换以及多查询（Advanced RAG 检索前)

上下文管理： 重排以及上下文选择和压缩。（Advanced RAG 检索后)

迭代检索：根据初始查询和迄今为止生成的文本进行重复检索

递归检索： 迭代细化搜索查询、链式推理指导检索过程

自适应检索： Flare,self-RAG, 使用LLMs 主动决定检索的最佳时机和内容。

**RAG vs 微调**

RAG：适用于需要结合最新信息和实时数据的任务；开放域问答、时事新闻摘要等等。严重依赖于大模型的能力，依赖于外部知识库的质量和覆盖范围。

Fine-tuning: 适用于数据可用且需要模型高度专业化的任务，如特定领域的文本分类、情感分析以及文本生成等。需要大量的标注数据，对新任务的适应性较差。

**如何评价RAG技术**

评测框架：RGB、RECALL和CRUD

评测工具：RAGAS、ARES、TruLens

## RAG 应用

**茴香豆是什么？**

开源大模型应用，基于LLMs的领域知识助手。

**茴香豆的工作流？**

三个部分：

预处理：转化为合适的问询

Rejection Pipeline: 对问询分析以及和数据库比较来给出得分判断是否进入回答环节

Response Pipeline: 使用模型进行生成。多来源检索、混合大模型包括本地和远程LLM，多重评分、安全检查。

## 实战部分

[茴香豆项目地址](https://link.zhihu.com/?target=https%3A//github.com/InternLM/HuixiangDou)

[指导文档地址](https://link.zhihu.com/?target=https%3A//github.com/InternLM/Tutorial/blob/camp2/huixiangdou/readme.md)