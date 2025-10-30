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


### VLLM服务启动参数

| 参数                    | 默认值       | 说明                                     |
| --------------------- | --------- | -------------------------------------- |
| `--model MODEL_PATH`  | 必填        | 模型路径，可以是本地目录或者 Hugging Face 模型标识       |
| `--host HOST`         | `0.0.0.0` | HTTP 服务监听的 IP                          |
| `--port PORT`         | `8000`    | HTTP 服务监听的端口                           |
| `--num-gpus N`        | 自动检测      | 使用的 GPU 数量                             |
| `--max-batch-size N`  | 32        | 单次批处理的最大请求数                            |
| `--max-num-tokens N`  | 2048      | 模型输出最大 token 数                         |
| `--fp16 / --no-fp16`  | 默认开启      | 是否使用 FP16 进行推理                         |
| `--low-memory-mode`   | False     | 降低显存占用，可能稍慢                            |
| `--log-level LEVEL`   | `info`    | 日志级别，可选：debug / info / warning / error |
| `--num-workers N`     | CPU 数量    | 用于异步处理请求的 worker 数                     |
| `--max-concurrency N` | 64        | 单个 GPU 上允许的最大并发请求数        



| 参数                         | 说明                                                   |
| -------------------------- | ---------------------------------------------------- |
| `--no-streaming`           | 关闭流式输出                                               |
| `--allow-remote-access`    | 允许外部网络访问（非 localhost）                                |
| `--disable-auto-devices`   | 禁止自动分配 GPU 或 CPU                                     |
| `--seed N`                 | 设置随机种子，保证生成可复现                                       |
| `--torch_dtype DTYPE`      | 指定 PyTorch 数据类型，如 `float16` / `bfloat16` / `float32` |
| `--tensor_parallel N`      | Tensor 并行的数量（多 GPU 分配策略）                             |
| `--pipeline_parallel N`    | Pipeline 并行层数（多 GPU 分配策略）                            |
| `--quantize {int8,int4}`   | 是否量化模型以节省显存                                          |
| `--max-memory-per-gpu MEM` | 每个 GPU 最大显存限制，例如 `12GB`                              |
             |
