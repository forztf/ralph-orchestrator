# Q Chat 适配器生产环境部署指南

本指南提供了在 Ralph Orchestrator 生产环境中部署 Q Chat 适配器的全面说明。

## 概述

Q Chat 适配器已经过彻底的测试和验证，可用于生产环境，具备以下功能：
- 线程安全的并发消息处理
- 健壮的错误处理和恢复机制
- 优雅的关闭和资源清理
- 非阻塞 I/O 以防止死锁
- 指数退避的自动重试
- 信号处理以确保正常终止

## 前置条件

### 系统要求
- Python 3.8 或更高版本
- 已安装并配置 Q CLI
- 足够的内存用于并发操作（建议至少 2GB）
- 类 Unix 操作系统（Linux、macOS）

### 安装
```bash
# 安装 Q CLI
pip install q-cli

# 验证安装
qchat --version

# 安装支持 Q 适配器的 Ralph Orchestrator
pip install ralph-orchestrator
```

## 配置

### 环境变量

使用这些环境变量配置 Q Chat 适配器的行为：

```bash
# 核心配置
export QCHAT_TIMEOUT=300          # 请求超时时间（秒）（默认：120）
export QCHAT_MAX_RETRIES=5        # 最大重试次数（默认：3）
export QCHAT_RETRY_DELAY=2        # 初始重试延迟（秒）（默认：1）
export QCHAT_VERBOSE=1            # 启用详细日志（默认：0）

# 性能调优
export QCHAT_BUFFER_SIZE=8192     # 管道缓冲区大小（字节）（默认：4096）
export QCHAT_POLL_INTERVAL=0.1    # 消息队列轮询间隔（默认：0.1）
export QCHAT_MAX_CONCURRENT=10    # 最大并发请求数（默认：5）

# 资源限制
export QCHAT_MAX_MEMORY_MB=4096   # 最大内存使用量（MB）
export QCHAT_MAX_OUTPUT_SIZE=10485760  # 最大输出大小（字节）（10MB）
```

### 配置文件

创建配置文件以保存持久化设置：

```yaml
# config/qchat.yaml
adapter:
  name: qchat
  timeout: 300
  max_retries: 5
  retry_delay: 2

performance:
  buffer_size: 8192
  poll_interval: 0.1
  max_concurrent: 10

logging:
  level: INFO
  format: "%(asctime)s - %(name)s - %(levelname)s - %(message)s"
  file: /var/log/ralph/qchat.log

monitoring:
  metrics_enabled: true
  metrics_interval: 60
  health_check_port: 8080
```

## 部署场景

### 1. 单实例部署

适用于中等负载的简单生产环境部署：

```bash
#!/bin/bash
# deploy-qchat.sh

# 设置生产环境
export ENVIRONMENT=production
export QCHAT_TIMEOUT=300
export QCHAT_VERBOSE=1

# 启动 Ralph Orchestrator 与 Q Chat
python -m ralph_orchestrator \
  --agent q \
  --config config/qchat.yaml \
  --checkpoint-interval 10 \
  --max-iterations 1000 \
  --metrics-interval 60 \
  --log-file /var/log/ralph/orchestrator.log
```

### 2. 高可用性部署

适用于需要高可用性的关键任务应用程序：

```bash
#!/bin/bash
# ha-deploy-qchat.sh

# 配置高可用性
export QCHAT_MAX_RETRIES=10
export QCHAT_RETRY_DELAY=5
export QCHAT_MAX_CONCURRENT=20

# 启用健康监控
export HEALTH_CHECK_ENABLED=true
export HEALTH_CHECK_INTERVAL=30

# 使用 supervisor 启动以自动重启
supervisorctl start ralph-qchat

# 或使用 systemd
systemctl start ralph-qchat.service
```

### 3. 容器化部署

用于容器部署的 Docker 配置：

```dockerfile
# Dockerfile
FROM python:3.11-slim

WORKDIR /app

# 安装依赖
RUN pip install ralph-orchestrator q-cli

# 复制配置
COPY config/qchat.yaml /app/config/

# 设置环境变量
ENV QCHAT_TIMEOUT=300
ENV QCHAT_VERBOSE=1
ENV PYTHONUNBUFFERED=1

# 健康检查
HEALTHCHECK --interval=30s --timeout=10s --retries=3 \
  CMD python -c "import requests; requests.get('http://localhost:8080/health')"

# 运行 orchestrator
CMD ["python", "-m", "ralph_orchestrator", "--agent", "q", "--config", "config/qchat.yaml"]
```

Docker Compose 配置：

```yaml
# docker-compose.yml
version: '3.8'

services:
  ralph-qchat:
    build: .
    container_name: ralph-qchat
    restart: unless-stopped
    environment:
      - QCHAT_TIMEOUT=300
      - QCHAT_MAX_RETRIES=5
      - QCHAT_VERBOSE=1
    volumes:
      - ./prompts:/app/prompts
      - ./checkpoints:/app/checkpoints
      - ./logs:/app/logs
    ports:
      - "8080:8080"  # 健康检查端点
    logging:
      driver: json-file
      options:
        max-size: "10m"
        max-file: "3"
```

## 监控和可观测性

### 日志配置

为生产环境配置结构化日志：

```python
# logging_config.py
import logging
import logging.handlers

def setup_logging():
    logger = logging.getLogger('ralph.qchat')
    logger.setLevel(logging.INFO)

    # 带轮转的文件处理器
    file_handler = logging.handlers.RotatingFileHandler(
        '/var/log/ralph/qchat.log',
        maxBytes=10485760,  # 10MB
        backupCount=5
    )

    # 结构化日志格式
    formatter = logging.Formatter(
        '{"time": "%(asctime)s", "level": "%(levelname)s", '
        '"module": "%(module)s", "message": "%(message)s"}'
    )
    file_handler.setFormatter(formatter)

    logger.addHandler(file_handler)
    return logger
```

### 指标收集

监控关键性能指标：

```python
# metrics.py
from prometheus_client import Counter, Histogram, Gauge

# 定义指标
request_count = Counter('qchat_requests_total', 'Q Chat 请求总数')
request_duration = Histogram('qchat_request_duration_seconds', '请求持续时间')
active_requests = Gauge('qchat_active_requests', '活动请求数量')
error_count = Counter('qchat_errors_total', '错误总数', ['error_type'])
```

### 健康检查

实现健康检查端点：

```python
# health_check.py
from flask import Flask, jsonify
import psutil

app = Flask(__name__)

@app.route('/health')
def health():
    """基本健康检查端点"""
    return jsonify({
        'status': 'healthy',
        'adapter': 'qchat',
        'version': '1.0.0'
    })

@app.route('/health/detailed')
def health_detailed():
    """带有系统指标的详细健康检查"""
    return jsonify({
        'status': 'healthy',
        'adapter': 'qchat',
        'version': '1.0.0',
        'system': {
            'cpu_percent': psutil.cpu_percent(),
            'memory_percent': psutil.virtual_memory().percent,
            'disk_usage': psutil.disk_usage('/').percent
        }
    })

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=8080)
```

## 性能优化

### 1. 连接池

为高并发场景优化：

```python
# connection_pool.py
from concurrent.futures import ThreadPoolExecutor
import queue

class QChatConnectionPool:
    def __init__(self, max_connections=10):
        self.executor = ThreadPoolExecutor(max_workers=max_connections)
        self.semaphore = threading.Semaphore(max_connections)

    def execute(self, prompt):
        with self.semaphore:
            future = self.executor.submit(self._execute_qchat, prompt)
            return future.result()
```

### 2. 缓存策略

为重复查询实现响应缓存：

```python
# cache.py
from functools import lru_cache
import hashlib

class QChatCache:
    def __init__(self, max_size=1000):
        self.cache = {}
        self.max_size = max_size

    def get_cache_key(self, prompt):
        return hashlib.sha256(prompt.encode()).hexdigest()

    def get(self, prompt):
        key = self.get_cache_key(prompt)
        return self.cache.get(key)

    def set(self, prompt, response):
        if len(self.cache) >= self.max_size:
            # 移除最旧的条目
            self.cache.pop(next(iter(self.cache)))
        key = self.get_cache_key(prompt)
        self.cache[key] = response
```

### 3. 资源限制

为生产环境稳定性配置资源限制：

```bash
# 设置系统限制
ulimit -n 4096          # 增加文件描述符限制
ulimit -u 2048          # 增加进程限制
ulimit -m 4194304       # 设置内存限制（4GB）

# 为容器环境配置 cgroups
echo "4G" > /sys/fs/cgroup/memory/ralph-qchat/memory.limit_in_bytes
echo "80" > /sys/fs/cgroup/cpu/ralph-qchat/cpu.shares
```

## 故障排除

### 常见问题和解决方案

#### 1. 死锁预防
```bash
# 检查管道缓冲区问题
strace -p <PID> -e read,write

# 如需要，增加缓冲区大小
export QCHAT_BUFFER_SIZE=16384
```

#### 2. 内存泄漏
```bash
# 监控内存使用
watch -n 1 'ps aux | grep qchat'

# 启用内存分析
export PYTHONTRACEMALLOC=1
```

#### 3. 进程挂起
```bash
# 检查进程状态
ps -eLf | grep qchat

# 发送诊断信号
kill -USR1 <PID>  # 触发诊断转储
```

#### 4. 高 CPU 使用率
```bash
# 分析 CPU 使用情况
py-spy top --pid <PID>

# 调整轮询间隔
export QCHAT_POLL_INTERVAL=0.5
```

### 调试模式

启用调试模式以获取详细诊断信息：

```bash
# 启用所有调试功能
export QCHAT_DEBUG=1
export QCHAT_VERBOSE=1
export PYTHONVERBOSE=1
export RUST_LOG=debug  # 如果使用基于 Rust 的组件

# 使用调试日志运行
python -m ralph_orchestrator \
  --agent q \
  --verbose \
  --debug \
  --log-level DEBUG
```

## 安全考虑

### 1. 输入验证

始终验证和清理输入：

```python
def validate_prompt(prompt):
    # 检查提示词长度
    if len(prompt) > MAX_PROMPT_LENGTH:
        raise ValueError("提示词超过最大长度")

    # 清理特殊字符
    prompt = prompt.replace('\0', '')

    # 检查注入尝试
    if any(pattern in prompt for pattern in BLOCKED_PATTERNS):
        raise SecurityError("检测到可能恶意的提示词")

    return prompt
```

### 2. 进程隔离

以受限权限运行 Q Chat 进程：

```bash
# 创建专用用户
useradd -r -s /bin/false qchat-user

# 以受限权限运行
sudo -u qchat-user python -m ralph_orchestrator --agent q
```

### 3. 网络安全

为健康检查端点配置防火墙规则：

```bash
# 仅允许来自监控系统的健康检查端口
iptables -A INPUT -p tcp --dport 8080 -s 10.0.0.0/8 -j ACCEPT
iptables -A INPUT -p tcp --dport 8080 -j DROP
```

## 维护和更新

### 滚动更新

执行零停机更新：

```bash
#!/bin/bash
# rolling-update.sh

# 启动新版本
docker-compose up -d ralph-qchat-new

# 等待健康检查
while ! curl -f http://localhost:8081/health; do
  sleep 5
done

# 切换流量（更新负载均衡器/代理）
nginx -s reload

# 停止旧版本
docker-compose stop ralph-qchat-old
```

### 备份和恢复

定期检查点备份：

```bash
# 备份检查点
tar -czf checkpoints-$(date +%Y%m%d).tar.gz checkpoints/

# 备份配置
cp -r config/ backup/config-$(date +%Y%m%d)/

# 从备份恢复
tar -xzf checkpoints-20240101.tar.gz
cp -r backup/config-20240101/* config/
```

## 性能基准

生产环境中的预期性能指标：

| 指标 | 数值 | 说明 |
|--------|-------|-------|
| **延迟 (p50)** | < 500ms | 简单提示词 |
| **延迟 (p99)** | < 2000ms | 复杂提示词 |
| **吞吐量** | 100 req/min | 单实例 |
| **并发度** | 10-20 | 并发请求 |
| **内存使用** | < 500MB | 每实例 |
| **CPU 使用** | < 50% | 平均利用率 |
| **错误率** | < 0.1% | 生产环境目标 |
| **可用性** | > 99.9% | 配合适当监控 |

## 最佳实践

1. **始终使用检查点** 用于长时间运行的任务
2. **持续监控资源使用**
3. **实施速率限制** 以防止过载
4. **使用连接池** 以获得更好的性能
5. **启用结构化日志** 以便于调试
6. **根据工作负载设置适当的超时**
7. **实施熔断器** 以实现容错
8. **定期备份** 检查点和配置
9. **定期测试灾难恢复** 程序
10. **保持 Q CLI 更新** 到最新的稳定版本

## 支持和资源

- **文档**: [Ralph Orchestrator 文档](https://ralph-orchestrator.readthedocs.io)
- **问题**: [GitHub Issues](https://github.com/your-org/ralph-orchestrator/issues)
- **社区**: [Discord 服务器](https://discord.gg/ralph-orchestrator)
- **紧急支持**: support@ralph-orchestrator.com

## 附录：Systemd 服务

```ini
# /etc/systemd/system/ralph-qchat.service
[Unit]
Description=Ralph Orchestrator with Q Chat Adapter
After=network.target

[Service]
Type=simple
User=qchat-user
Group=qchat-group
WorkingDirectory=/opt/ralph-orchestrator
Environment="QCHAT_TIMEOUT=300"
Environment="QCHAT_VERBOSE=1"
ExecStart=/usr/bin/python3 -m ralph_orchestrator --agent q --config /etc/ralph/qchat.yaml
Restart=always
RestartSec=10
StandardOutput=append:/var/log/ralph/qchat.log
StandardError=append:/var/log/ralph/qchat-error.log

[Install]
WantedBy=multi-user.target
```

启用并启动服务：

```bash
systemctl daemon-reload
systemctl enable ralph-qchat.service
systemctl start ralph-qchat.service
systemctl status ralph-qchat.service
```
