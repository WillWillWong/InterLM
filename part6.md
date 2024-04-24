Lagent与AgentLego：构建智能体的新工具

#### 学习资源
- **视频教程**：Bilibili上的视频提供了直观的学习体验。https://www.bilibili.com/video/BV1Xt4217728/
- **课程文档**：在GitHub上可以找到详尽的教程文档。https://link.zhihu.com/?target=https%3A//github.com/InternLM/Tutorial/tree/camp2/agent

#### 智能体范式概览
- **AutoGPT**：一个自动化流程，用户任务通过GPT4执行并循环。
- **ReWoo**：通过有向无环图规划工具依赖和执行顺序。
- **ReAct**：结合推理与行为，动态选择工具以完成任务。

#### Lagent智能体框架
- **Lagent**：一个高效的开源框架，支持AutoGPT和ReAct等智能体范式。
- **工具集成**：提供了一系列工具支持，如搜索引擎和交互式解释器。

#### AgentLego多模态工具包
- **AgentLego**：一个多功能的工具包，提供API简化自定义工具开发。
- **应用场景**：与智能体框架结合，增强语言模型的功能性。

#### Lagent与AgentLego的协同
- **框架与工具包**：Lagent作为智能体框架，AgentLego作为功能增强的工具包，两者相互配合，共同构建智能体。

Lagent 目前已经支持了包括 AutoGPT、ReAct 等在内的多个经典智能体范式，也支持了如下工具：

- Arxiv 搜索
- Bing 地图
- Google 学术搜索
- Google 搜索
- 交互式 IPython 解释器
- IPython 解释器
- PPT
- 
- Python 解释器

AgentLego: 是一个提供了多种开源工具 API 的多模态工具包，旨在像是乐高积木一样，让用户可以快速简便地拓展自定义工具，从而组装出自己的智能体。通过 AgentLego 算法库，不仅可以直接使用多种工具，也可以利用这些工具，在相关智能体框架（如 Lagent，Transformers Agent 等）的帮助下，快速构建可以增强大语言模型能力的智能体。