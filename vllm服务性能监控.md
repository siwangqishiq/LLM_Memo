# VLLM服务性能监控

启动LLM服务后针对 以下性能指标的测量方法: 
    1. 吞吐量 throughput
    2. 延迟 latency
    3. 资源使用 CPU/GPU/Memory
    4. 请求并发能力
    5. Token消耗

## 启动一个LLM服务 
    vllm serve facebook/opt-13b --host 0.0.0.0 --port 8000

## Python性能测试脚本
```Python
import asyncio
import httpx
import time
import statistics

# 配置
VLLM_URL = "http://192.168.200.129:8000/v1/completions"  # vLLM HTTP 服务地址
PROMPT = "介绍一下广义相对论"
MAX_OUTPUT_TOKENS = 5000
NUM_REQUESTS = 50
CONCURRENCY = 20  # 并发请求数

async def request(client, prompt):
    payload = {
        "prompt": PROMPT,
        "max_tokens": MAX_OUTPUT_TOKENS
    }
    start = time.time()
    r = await client.post(VLLM_URL, json=payload)
    end = time.time()
    # print(r)
    data = r.json()
    # print(data.get("choices"))
   
    # 获取 token 信息
    usage = data.get("usage", {})
    input_tokens = usage.get("prompt_tokens", 0)
    output_tokens = usage.get("completion_tokens", 0)
    latency = end - start
    return latency, input_tokens, output_tokens

async def main():
    latencies = []
    total_input_tokens = 0
    total_output_tokens = 0
    
    cnt = 0
    start_time = time.time()  # 测试起始时间

    async with httpx.AsyncClient(timeout=600.0) as client:
        # 分批并发请求
        tasks = []
        for _ in range(NUM_REQUESTS):
            print(f"progress : {cnt} / {NUM_REQUESTS}")
            cnt = cnt + 1
            tasks.append(request(client, PROMPT))
            if len(tasks) >= CONCURRENCY:
                results = await asyncio.gather(*tasks)
                tasks = []
                for latency, in_tokens, out_tokens in results:
                    latencies.append(latency)
                    total_input_tokens += in_tokens
                    total_output_tokens += out_tokens

        # 处理剩余任务
        if tasks:
            results = await asyncio.gather(*tasks)
            for latency, in_tokens, out_tokens in results:
                latencies.append(latency)
                total_input_tokens += in_tokens
                total_output_tokens += out_tokens
                
    end_time = time.time()  # 测试结束时间            
    total_duration = end_time - start_time  # 总耗时
    total_requests = len(latencies)
    total_tokens = total_input_tokens + total_output_tokens
    total_time = sum(latencies)

    # 输出性能指标
    print(f"总请求数: {total_requests}")
    print(f"平均延迟: {statistics.mean(latencies):.3f} 秒")
    print(f"最大延迟: {max(latencies):.3f} 秒")
    print(f"最小延迟: {min(latencies):.3f} 秒")
    print(f"总输入 token: {total_input_tokens}")
    print(f"总输出 token: {total_output_tokens}")
    print(f"平均每请求输出 token: {total_output_tokens / total_requests:.2f}")
    print(f"总 token 数量: {total_tokens}")
    print(f"平均 token/s: {total_tokens / total_time:.2f}")
    
    tps = total_tokens / total_duration
    tpm = tps * 60
    
    # print(f"平均 TPM (Tokens Per Minute): {tpm:.2f}")
    print(f"总耗时: {total_duration:.2f} 秒")

if __name__ == "__main__":
    asyncio.run(main())

```


## 使用 locust 测试并发性能

### 安装 pip install locust requests