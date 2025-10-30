# Python性能测试工具 Locust

## 安装
    pip install locust

## 架构
    1. Python工具 
    2. 使用request 发送Http请求
    3. 使用协程 低成本并发实现
    4. 支持分布式
    5. 使用Flask提供WebUI
    6. 三方插件扩展

## locust测试用例
```python

class MyUser(locust.HttpUser):
    wait_time = locust.between(1, 2)

    @locust.task
    def test_api(self):
        resp = self.client.post("/api/test", json={"key": "value"})
        assert resp.status_code == 200
    
```

## locust执行压测

### WebUI执行
    locust -f test.py
    简单 + 绘制图表 直观

### 命令行执行
    locust -f test_case.py --headless -u 500 -r 10 --host 127.0.0.1:8080 -t 10m

    -u 用户数
    -r 增长速度
    -host 请求主机地址
    -t 测试时间

## locust核心组件 

 1. 父类
 2. 行为
 3. 等待时间
