### OpenCompass：大模型评测实战指南

#### 学习资源
- **视频教程**：提供了Bilibili上的实战视频链接，帮助理解操作流程。[OpenCompass 大模型评测实战_哔哩哔哩_bilibili](https://link.zhihu.com/?target=https%3A//www.bilibili.com/video/BV1Pm41127jU/)
- **课程文档**：GitHub上的文档提供了全面的教程内容。[github.com/InternLM/Tut](https://link.zhihu.com/?target=https%3A//github.com/InternLM/Tutorial/blob/camp2/opencompass/readme.md)

#### 评测理论
- 探讨了能力评测在模型发展中的重要性，以及未来评测能力维度的拓展方向。
  1. 面向未来，拓展能力维度
  2. 扎根通用能力，聚焦垂直行业
  3. 高质量中文基准
  4. 性能评测，反哺能力迭代

#### 评测挑战
- 讨论了大模型评测面临的主要挑战，包括全面性、成本、数据污染和鲁棒性。

#### 评测对象
- 涉及基座模型、对话模型、开源模型和API模型等。

#### 评测方法
- 结合了客观评测（如问答题和选择题）和主观评测（开放式问答）。

#### OpenCompass评测体系
- **CompassRank**：提供模型性能排名。
- **CompassKit**：一套大模型评测工具链条。
- **OpenCompass流水线**：提供一站式的评测服务。

#### OpenCompass特点
- 系统开源可复现，覆盖全面的能力维度，支持多种模型，实现分布式高效评测，具备多样化的评测范式和灵活的拓展性。

#### 工具架构
- 评测系统的工具架构，包括模型层、能力层和方法层。

#### 评测流程
- 包括配置、推理、评估和可视化四个主要阶段。

#### 实战演练
- 使用OpenCompass评测特定模型在C-Eval数据集上的性能的步骤。

  ```bash
  studio-conda -o internlm-base -t opencompass
  source activate opencompass
  git clone -b 0.2.4 https://github.com/open-compass/opencompass
  cd opencompass
  pip install -e .
  
  pip install -r requirements.txt
  ```

```text
cp /share/temp/datasets/OpenCompassData-core-20231110.zip /root/opencompass/
unzip OpenCompassData-core-20231110.zip
```

```text
python tools/list_configs.py internlm ceval
```

```bash
python run.py --datasets ceval_gen --hf-path /share/new_models/Shanghai_AI_Laboratory/internlm2-chat-1_8b --tokenizer-path /share/new_models/Shanghai_AI_Laboratory/internlm2-chat-1_8b --tokenizer-kwargs padding_side='left' truncation='left' trust_remote_code=True --model-kwargs trust_remote_code=True device_map='auto' --max-seq-len 1024 --max-out-len 16 --batch-size 2 --num-gpus 1 --debug
```

#### 数据集贡献

- 介绍了如何在OpenCompass平台上贡献新的数据集，包括创建私人数据集和提交相关文档。

文章通过实战演练和详细的操作指南，为读者提供了OpenCompass评测系统的全面认识。如果您需要进一步的信息或对特定部分有疑问，请告知。