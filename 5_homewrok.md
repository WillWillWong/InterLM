### 实战指南：InternLM模型的高效部署

#### 学习资源
- **视频课程**：推荐观看Bilibili上的LMDeploy量化部署教程。[LMDeploy 量化部署 LLM-VLM 实践_哔哩哔哩_bilibili](https://link.zhihu.com/?target=https%3A//www.bilibili.com/video/BV1tr421x75B/)
- **文档指南**：提供了GitHub上的详细文档，方便进一步学习。[Tutorial/lmdeploy/README.md at camp2 · InternLM/Tutorial](https://link.zhihu.com/?target=https%3A//github.com/InternLM/Tutorial/blob/camp2/lmdeploy/README.md)

#### HuggingFace与TurboMind
- **HuggingFace社区**：一个活跃的开源平台，支持模型和数据集的托管，采用HF格式。
- **TurboMind引擎**：由LMDeploy团队开发的高效推理引擎，专为LLaMa模型设计，具备连续批处理和KV缓存管理功能。

#### 实操演练
- **模型获取**：通过软连接或OpenXLab平台简化模型下载流程。

  ```bash
  ln -s /root/share/new_models/Shanghai_AI_Laboratory/internlm2-chat-1_8b /root/models
  ```

- **模型执行**：利用Transformer库快速运行HF格式模型，并计算执行时间。

  ```python
  import torch
  from transformers import AutoTokenizer, AutoModelForCausalLM
  import time
  start = time.time()
  tokenizer = AutoTokenizer.from_pretrained("/root/models/internlm2-chat-1_8b", trust_remote_code=True)
  
  # Set `torch_dtype=torch.float16` to load model in float16, otherwise it will be loaded as float32 and cause OOM Error.
  model = AutoModelForCausalLM.from_pretrained("/root/models/internlm2-chat-1_8b", torch_dtype=torch.float16, trust_remote_code=True).cuda()
  model = model.eval()
  inp = "hello"
  print("[INPUT]", inp)
  response, history = model.chat(tokenizer, inp, history=[])
  print("[OUTPUT]", response)
  
  inp = "please provide three suggestions about ielts' preparation"
  print("[INPUT]", inp)
  response, history = model.chat(tokenizer, inp, history=history)
  print("[OUTPUT]", response)
  print(f'We use {time.time{}-start}s to complete the task.')
  ```

#### LMDeploy功能应用
- **对话功能**：使用LMDeploy的chat模式在终端与模型进行互动。
- **量化技术**：探索了计算密集与访存密集场景下的优化策略，以及KV8和W4A16量化技术的使用。

#### KV Cache机制
- **缓存优化**：介绍了KV Cache技术，通过缓存键值对减少重复计算，提升推理速度。

#### 量化部署实操
- **AWQ算法**：LMDeploy采用AWQ算法进行模型的4bit权重量化，配合TurboMind引擎实现高效推理。

#### 模型服务化
- **API服务封装**：讨论了将模型作为API服务部署的架构，包括模型推理、API服务器和客户端交互。

  ```bash
  lmdeploy serve api_server \
      /root/models/internlm2-chat-1_8b-4bit \
      --model-format awq \
      --cache-max-entry-count 0.4\
      --quant-policy 0 \
      --server-name 0.0.0.0 \
      --server-port 23333 \
      --tp 1
  ```

```bash
lmdeploy serve api_client http://localhost:23333
```

#### Python集成部署

- **编程集成**：展示了在Python脚本中集成模型部署的方法，并设置了量化参数。

  ```python
  from lmdeploy import pipeline, TurbomindEngineConfig
  # 调低 k/v cache内存占比调整为总显存的 40%，采用 W4A16量化
  backend_config = TurbomindEngineConfig(cache_max_entry_count=0.4,model_format="awq")
  pipe = pipeline('/root/models/internlm2-chat-1_8b-4bit',
                  backend_config=backend_config)
  inp1 = "Give 3 advices on preparing IELTS"
  print("[INPUT1]", inp1)
  inp2 = "中国科学院是"
  print("[INPUT2]", inp2)
  responses = pipe([inp1, inp2])
  for i,response in enumerate(responses):
      print(f"[OUTPUT{i}]", response.text)
  ```

#### 视觉模型部署案例
- **llava模型部署**：演示了如何部署视觉大模型llava，并使用图片进行测试。

```python
from lmdeploy.vl import load_image
from lmdeploy import pipeline, TurbomindEngineConfig
backend_config = TurbomindEngineConfig(session_len=8192, cache_max_entry_count=0.4) # 图片分辨率较高时请调高session_len
pipe = pipeline('/root/models/llava-v1.6-vicuna-7b', backend_config=backend_config)
image = load_image('https://rr-cchen.github.io/web/image/img-v-0.jpeg')
response = pipe(('describe this image in english', image))
print(response)
```