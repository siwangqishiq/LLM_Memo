# VLLM 教程

    vLLM 是一个快速且易于使用的 LLM 推理和服务库.

## 环境
    OS: Linux 
    Python : 3.9 ~ 3.13

## 安装

#### 设置网络代理
    export http_proxy="http://192.168.102.179:21882"
    export https_proxy="http://192.168.102.179:21882"
    export http_proxy="socks5h://192.168.100.79:21881"
    export https_proxy="socks5h://192.168.100.79:21881"

## 安装VLLM
   
### 创建python虚拟环境
    python -m venv vllm_env # 激活后，pip install 的包只会装到这个环境中。
    
    source vllm_env/bin/activate # 激活虚拟环境

### 安装VLLM

    pip install vllm

### 验证是否VLLM安装成功
``` python
import vllm
print(__vllm__.version)
```

### 离线推理
```python
from vllm import LLM, SamplingParams
prompts = [
    "Hello, my name is",
    "The president of the United States is",
    "The capital of France is",
    "The future of AI is",
]
sampling_params = SamplingParams(temperature=0.8, top_p=0.95)
llm = LLM(model="facebook/opt-125m")

outputs = llm.generate(prompts, sampling_params)

for output in outputs:
    prompt = output.prompt
    generated_text = output.outputs[0].text
    print(f"Prompt: {prompt!r}, Generated text: {generated_text!r}")
```

### 部署OpenAI兼容服务器

    vllm serve Qwen/Qwen2.5-1.5B-Instruct # 部署千问模型 

### 验证服务 

    curl http://192.168.200.129:8000/v1/completions -H "Content-Type: application/json" -d '{
        "model": "Qwen/Qwen2.5-1.5B-Instruct",
        "prompt": "San Francisco is a",
        "max_tokens": 100,
        "temperature": 0
    }'
