这篇文章主要介绍了如何使用XTuner工具对个人助手进行微调和多模态微调。

### XTuner工具实战：个性化助手微调及多模态微调指南

**环境配置：**
- 使用CUDA 11.7版本和如何激活XTuner 0.1.17环境。

  ```bash
  # 列出所有内置配置文件
  # xtuner list-cfg
  # 假如我们想找到 internlm2-1.8b 模型里支持的配置文件
  xtuner list-cfg -p internlm2_1_8b
  ```

**数据集构建：**
- 创建数据文件夹，并使用Python脚本生成个性化的对话数据集。避免过拟合的重要性，并使用模型蒸馏方法来生成小型而高效的模型。

  ```bash
  # 前半部分是创建一个文件夹，后半部分是进入该文件夹。
  mkdir -p /root/ft && cd /root/ft
  
  # 在ft这个文件夹里再创建一个存放数据的data文件夹
  mkdir -p /root/ft/data && cd /root/ft/data
  ```

**模型准备：**
- 创建模型文件的软链接，并使用XTuner工具中的`list-cfg`命令列出所有配置文件，根据需要选择和复制特定的配置文件。

  ```bash
  ln -s /root/share/new_models/Shanghai_AI_Laboratory/internlm2-chat-1_8b /root/models/
  ```

**模型训练：**
- 通过`xtuner train`命令开始训练过程，并通过`--work-dir`参数指定模型保存位置。

  ```bash
  # 列出所有内置配置文件
  # xtuner list-cfg
  # 假如我们想找到 internlm2-1.8b 模型里支持的配置文件
  xtuner list-cfg -p internlm2_1_8b
  ```

```bash
# 创建一个存放 config 文件的文件夹
mkdir -p /root/finetuning/config

# 使用 XTuner 中的 copy-cfg 功能将 config 文件复制到指定的位置
xtuner copy-cfg internlm2_1_8b_qlora_alpaca_e3 /root/finetuning/config
```





**混合数据集训练与过拟合问题：**

- 在训练过程中可能出现的过拟合现象，两种解决方案：一是调整权重文件保存策略；二是增加常规对话数据集以稀释特定数据的占比。

**模型转换：**
- 将Pytorch模型权重文件转换为Huggingface格式，以便于在其他平台上使用。

  ```bash
  # 模型转换
  # xtuner convert pth_to_hf ${配置文件地址} ${权重文件地址} ${转换后模型保存地址}
  xtuner convert pth_to_hf /root/finetuning/train_alpaca_8g_1_9/internlm2_1_8b_qlora_alpaca_e3_copy_0.py   /root/finetuning/train_alpaca_8g_1_9/iter_1068.pth /root/finetuning/huggingface
  ```

**模型整合：**

- LoRA或QLoRA微调出的模型是一个额外层（adapter），需要与原模型组合使用。提供了整合模型的步骤和命令。

  ```bash
  # 创建一个名为 final_model 的文件夹存储整合后的模型文件
  mkdir -p /root/finetuning/final_model
  # 解决一下线程冲突的 Bug 
  export MKL_SERVICE_FORCE_INTEL=1
  
  # 进行模型整合
  # xtuner convert merge  ${NAME_OR_PATH_TO_LLM} ${NAME_OR_PATH_TO_ADAPTER} ${SAVE_PATH} 
  xtuner convert merge /root/models/internlm2-chat-1_8b /root/finetuning/huggingface /root/finetuning/final_model/
  ```

**对话测试：**
- 在终端与Huggingface格式的模型进行对话测试，并强调了正确选择提示词模版的重要性。

  ```text
  # 与模型进行对话
  xtuner chat /root/finetuning/final_model --prompt-template default
  ```

**Web部署：**

- 训练好的模型部署为Web应用的，包括客户端界面的制作和使用Streamlit运行聊天界面。

```python
class GenerationConfig:
    # this config is used for chat to provide more diversity
    max_length: int = 2048
    top_p: float = 0.75
    temperature: float = 0.1
    do_sample: bool = True
    repetition_penalty: float = 1.000
def load_model():
    model = (AutoModelForCausalLM.from_pretrained('/root/finetuning/final_model',
                                                  trust_remote_code=True).to(
                                                      torch.bfloat16).cuda())
    tokenizer = AutoTokenizer.from_pretrained('/root/finetuning/final_model',
                                              trust_remote_code=True)
    return model, tokenizer
def prepare_generation_config():
    with st.sidebar:
        max_length = st.slider('Max Length',
                               min_value=8,
                               max_value=2048,
                               value=2048)
        top_p = st.slider('Top P', 0.0, 1.0, 0.75, step=0.01)
        temperature = st.slider('Temperature', 0.0, 1.0, 0.1, step=0.01)
        st.button('Clear Chat History', on_click=on_btn_click)

    generation_config = GenerationConfig(max_length=max_length,
                                         top_p=top_p,
                                         temperature=temperature)

    return generation_config
user_prompt = '<|User|>:{user}</s>\n'
robot_prompt = '<|Bot|>:\n{robot}</s>\n'
cur_query_prompt = '<|User|>:{user}</s>\n\
    <|Bot|>:</s>\n'

def combine_history(prompt):
    messages = st.session_state.messages
    meta_instruction = ('')
    total_prompt = f"<s><|im_start|>system\n{meta_instruction}<|im_end|>\n"
    for message in messages:
        cur_content = message['content']
        if message['role'] == 'user':
            cur_prompt = user_prompt.format(user=cur_content)
        elif message['role'] == 'robot':
            cur_prompt = robot_prompt.format(robot=cur_content)
        else:
            raise RuntimeError
        total_prompt += cur_prompt
    total_prompt = total_prompt + cur_query_prompt.format(user=prompt)
    return total_prompt
def main():
    st.title('HoroFinetuning-InternLM2')
```

然后，运行该 `chat.py` 文件:

```bash
streamlit run ./chat.py --server.address 127.0.0.1 --server.port 6006
```

windows power shell 中使用：

```bash
ssh -CNg -L 6006:127.0.0.1:6006 root@ssh.intern-ai.org.cn -p 端口号
```

**OpenXLab部署：**

- 在OpenXLab平台上部署模型的过程，包括使用Git LFS管理大文件和编写应用代码。

**LLaVA方案微调：**

- LLaVA方案的微调过程，包括Pretrain阶段和Finetune阶段，以及如何使用XTuner工具进行聊天测试。

