# 生产环境部署指南

## 概述

本指南介绍在生产环境中部署 Ralph Orchestrator，包括服务器设置、自动化、监控和扩展性考虑。

## 部署选项

### 1. 本地服务器部署

#### 系统要求
- **操作系统**: Linux (Ubuntu 20.04+, RHEL 8+, Debian 11+)
- **Python**: 3.9+
- **Git**: 2.25+
- **内存**: 最低 4GB，推荐 8GB
- **存储**: 20GB 可用空间
- **网络**: 稳定的互联网连接用于 AI agent APIs

#### 安装脚本
```bash
#!/bin/bash
# ralph-install.sh

# 更新系统
sudo apt-get update && sudo apt-get upgrade -y

# 安装依赖
sudo apt-get install -y python3 python3-pip git nodejs npm

# 安装 AI agents
npm install -g @anthropic-ai/claude-code
npm install -g @google/gemini-cli
# 按照 Q 的文档安装 Q

# 克隆 Ralph
git clone https://github.com/yourusername/ralph-orchestrator.git
cd ralph-orchestrator

# 设置权限
chmod +x ralph_orchestrator.py ralph

# 创建 systemd 服务
sudo cp ralph.service /etc/systemd/system/
sudo systemctl daemon-reload
sudo systemctl enable ralph
```

### 2. Docker 部署

#### Dockerfile
```dockerfile
FROM python:3.11-slim

# 安装系统依赖
RUN apt-get update && apt-get install -y \
    git \
    nodejs \
    npm \
    && rm -rf /var/lib/apt/lists/*

# 安装 AI CLI 工具
RUN npm install -g @anthropic-ai/claude-code @google/gemini-cli

# 创建 ralph 用户
RUN useradd -m -s /bin/bash ralph
WORKDIR /home/ralph

# 复制应用
COPY --chown=ralph:ralph . /home/ralph/ralph-orchestrator/
WORKDIR /home/ralph/ralph-orchestrator

# 设置权限
RUN chmod +x ralph_orchestrator.py ralph

# 切换到 ralph 用户
USER ralph

# 默认命令
CMD ["./ralph", "run"]
```

#### Docker Compose
```yaml
# docker-compose.yml
version: '3.8'

services:
  ralph:
    build: .
    container_name: ralph-orchestrator
    restart: unless-stopped
    volumes:
      - ./workspace:/home/ralph/workspace
      - ./prompts:/home/ralph/prompts
      - ralph-agent:/home/ralph/ralph-orchestrator/.agent
    environment:
      - RALPH_MAX_ITERATIONS=100
      - RALPH_AGENT=auto
      - RALPH_CHECKPOINT_INTERVAL=5
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "3"

volumes:
  ralph-agent:
```

### 3. 云部署

#### AWS EC2
```bash
# EC2 实例的用户数据脚本
#!/bin/bash
yum update -y
yum install -y python3 git nodejs

# 安装 Ralph
cd /opt
git clone https://github.com/yourusername/ralph-orchestrator.git
cd ralph-orchestrator
chmod +x ralph_orchestrator.py ralph

# 配置为服务
cat > /etc/systemd/system/ralph.service << EOF
[Unit]
Description=Ralph Orchestrator
After=network.target

[Service]
Type=simple
User=ec2-user
WorkingDirectory=/opt/ralph-orchestrator
ExecStart=/opt/ralph-orchestrator/ralph run
Restart=on-failure
RestartSec=10

[Install]
WantedBy=multi-user.target
EOF

systemctl enable ralph
systemctl start ralph
```

#### Kubernetes 部署
```yaml
# ralph-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ralph-orchestrator
spec:
  replicas: 1
  selector:
    matchLabels:
      app: ralph
  template:
    metadata:
      labels:
        app: ralph
    spec:
      containers:
      - name: ralph
        image: ralph-orchestrator:latest
        resources:
          requests:
            memory: "2Gi"
            cpu: "1"
          limits:
            memory: "4Gi"
            cpu: "2"
        volumeMounts:
        - name: workspace
          mountPath: /workspace
        - name: config
          mountPath: /config
      volumes:
      - name: workspace
        persistentVolumeClaim:
          claimName: ralph-workspace
      - name: config
        configMap:
          name: ralph-config
```

## 配置管理

### 环境变量
```bash
# /etc/environment 或 .env 文件
RALPH_HOME=/opt/ralph-orchestrator
RALPH_WORKSPACE=/var/ralph/workspace
RALPH_LOG_LEVEL=INFO
RALPH_MAX_ITERATIONS=100
RALPH_MAX_RUNTIME=14400
RALPH_AGENT=claude
RALPH_CHECKPOINT_INTERVAL=5
RALPH_RETRY_DELAY=2
RALPH_GIT_ENABLED=true
RALPH_ARCHIVE_ENABLED=true
```

### 配置文件
```json
{
  "production": {
    "agent": "claude",
    "max_iterations": 100,
    "max_runtime": 14400,
    "checkpoint_interval": 5,
    "retry_delay": 2,
    "retry_max": 5,
    "timeout_per_iteration": 300,
    "git_enabled": true,
    "archive_enabled": true,
    "monitoring": {
      "enabled": true,
      "metrics_endpoint": "http://metrics.example.com",
      "log_level": "INFO"
    },
    "security": {
      "sandbox_enabled": true,
      "allowed_directories": ["/workspace"],
      "forbidden_commands": ["rm -rf", "sudo", "su"],
      "max_file_size": 10485760
    }
  }
}
```

## 自动化

### Systemd 服务
```ini
# /etc/systemd/system/ralph.service
[Unit]
Description=Ralph Orchestrator Service
Documentation=https://github.com/yourusername/ralph-orchestrator
After=network.target

[Service]
Type=simple
User=ralph
Group=ralph
WorkingDirectory=/opt/ralph-orchestrator
ExecStart=/opt/ralph-orchestrator/ralph run --config production.json
ExecReload=/bin/kill -HUP $MAINPID
Restart=on-failure
RestartSec=30
StandardOutput=journal
StandardError=journal
SyslogIdentifier=ralph
Environment="PYTHONUNBUFFERED=1"

# 安全设置
NoNewPrivileges=true
PrivateTmp=true
ProtectSystem=strict
ProtectHome=true
ReadWritePaths=/opt/ralph-orchestrator /var/ralph

[Install]
WantedBy=multi-user.target
```

### Cron 作业
```bash
# /etc/cron.d/ralph
# 每周清理旧日志
0 2 * * 0 ralph /opt/ralph-orchestrator/scripts/cleanup.sh

# 每日备份状态
0 3 * * * ralph tar -czf /backup/ralph-$(date +\%Y\%m\%d).tar.gz /opt/ralph-orchestrator/.agent

# 每 5 分钟健康检查
*/5 * * * * ralph /opt/ralph-orchestrator/scripts/health-check.sh || systemctl restart ralph
```

### CI/CD 管道
```yaml
# .github/workflows/deploy.yml
name: Deploy Ralph

on:
  push:
    branches: [main]
    paths:
      - 'ralph_orchestrator.py'
      - 'ralph'
      - 'requirements.txt'

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - name: 运行测试
        run: python test_comprehensive.py

      - name: 构建 Docker 镜像
        run: docker build -t ralph-orchestrator:${{ github.sha }} .

      - name: 推送到镜像仓库
        run: |
          docker tag ralph-orchestrator:${{ github.sha }} ${{ secrets.REGISTRY }}/ralph:latest
          docker push ${{ secrets.REGISTRY }}/ralph:latest

      - name: 部署到服务器
        uses: appleboy/ssh-action@v0.1.5
        with:
          host: ${{ secrets.HOST }}
          username: ${{ secrets.USERNAME }}
          key: ${{ secrets.SSH_KEY }}
          script: |
            cd /opt/ralph-orchestrator
            git pull
            systemctl restart ralph
```

## 生产环境监控

### Prometheus 指标
```python
# metrics_exporter.py
from prometheus_client import Counter, Histogram, Gauge, start_http_server
import json
import glob

# 定义指标
iteration_counter = Counter('ralph_iterations_total', '总迭代次数')
error_counter = Counter('ralph_errors_total', '总错误数')
runtime_gauge = Gauge('ralph_runtime_seconds', '当前运行时间')
iteration_duration = Histogram('ralph_iteration_duration_seconds', '迭代持续时间')

def collect_metrics():
    """从 Ralph 状态文件收集指标"""
    state_files = glob.glob('.agent/metrics/state_*.json')
    if state_files:
        latest = max(state_files)
        with open(latest) as f:
            state = json.load(f)

        iteration_counter.inc(state.get('iteration_count', 0))
        runtime_gauge.set(state.get('runtime', 0))

        if state.get('errors'):
            error_counter.inc(len(state['errors']))

if __name__ == '__main__':
    # 启动指标服务器
    start_http_server(8000)

    # 定期收集指标
    while True:
        collect_metrics()
        time.sleep(30)
```

### 日志设置
```python
# logging_config.py
import logging
import logging.handlers
import json

def setup_production_logging():
    """配置生产环境日志"""

    # JSON 格式化器用于结构化日志
    class JSONFormatter(logging.Formatter):
        def format(self, record):
            log_obj = {
                'timestamp': self.formatTime(record),
                'level': record.levelname,
                'logger': record.name,
                'message': record.getMessage(),
                'module': record.module,
                'function': record.funcName,
                'line': record.lineno
            }
            if record.exc_info:
                log_obj['exception'] = self.formatException(record.exc_info)
            return json.dumps(log_obj)

    # 配置根日志记录器
    logger = logging.getLogger()
    logger.setLevel(logging.INFO)

    # 带轮转的文件处理器
    file_handler = logging.handlers.RotatingFileHandler(
        '/var/log/ralph/ralph.log',
        maxBytes=100*1024*1024,  # 100MB
        backupCount=10
    )
    file_handler.setFormatter(JSONFormatter())

    # Syslog 处理器
    syslog_handler = logging.handlers.SysLogHandler(address='/dev/log')
    syslog_handler.setFormatter(JSONFormatter())

    logger.addHandler(file_handler)
    logger.addHandler(syslog_handler)
```

## 安全加固

### 用户隔离
```bash
# 创建专用用户
sudo useradd -r -s /bin/bash -m -d /opt/ralph ralph
sudo chown -R ralph:ralph /opt/ralph-orchestrator

# 设置限制性权限
chmod 750 /opt/ralph-orchestrator
chmod 640 /opt/ralph-orchestrator/*.py
chmod 750 /opt/ralph-orchestrator/ralph
```

### 网络安全
```bash
# 防火墙规则 (iptables)
iptables -A OUTPUT -p tcp --dport 443 -j ACCEPT  # AI agents 的 HTTPS
iptables -A OUTPUT -p tcp --dport 22 -j ACCEPT   # Git SSH
iptables -A OUTPUT -j DROP                       # 阻止其他出站流量

# 或使用 ufw
ufw allow out 443/tcp
ufw allow out 22/tcp
ufw default deny outgoing
```

### API 密钥管理
```bash
# 使用系统密钥环
pip install keyring

# 安全存储 API 密钥
python -c "import keyring; keyring.set_password('ralph', 'claude_api_key', 'your-key')"

# 或从安全存储使用环境变量
source /etc/ralph/secrets.env
```

## 扩展性考虑

### 水平扩展
```python
# job_queue.py
import redis
import json

class RalphJobQueue:
    def __init__(self):
        self.redis = redis.Redis(host='localhost', port=6379)

    def add_job(self, prompt_file, config):
        """添加作业到队列"""
        job = {
            'id': str(uuid.uuid4()),
            'prompt_file': prompt_file,
            'config': config,
            'status': 'pending',
            'created': time.time()
        }
        self.redis.lpush('ralph:jobs', json.dumps(job))
        return job['id']

    def get_job(self):
        """从队列获取下一个作业"""
        job_data = self.redis.rpop('ralph:jobs')
        if job_data:
            return json.loads(job_data)
        return None
```

### 资源限制
```python
# resource_limits.py
import resource

def set_production_limits():
    """设置生产环境的资源限制"""

    # 内存限制 (4GB)
    resource.setrlimit(
        resource.RLIMIT_AS,
        (4 * 1024 * 1024 * 1024, -1)
    )

    # CPU 时间限制 (1 小时)
    resource.setrlimit(
        resource.RLIMIT_CPU,
        (3600, 3600)
    )

    # 文件大小限制 (100MB)
    resource.setrlimit(
        resource.RLIMIT_FSIZE,
        (100 * 1024 * 1024, -1)
    )

    # 进程数限制
    resource.setrlimit(
        resource.RLIMIT_NPROC,
        (100, 100)
    )
```

## 备份和恢复

### 自动备份
```bash
#!/bin/bash
# backup.sh

BACKUP_DIR="/backup/ralph"
TIMESTAMP=$(date +%Y%m%d_%H%M%S)

# 创建备份
tar -czf $BACKUP_DIR/ralph_$TIMESTAMP.tar.gz \
    /opt/ralph-orchestrator/.agent \
    /opt/ralph-orchestrator/*.json \
    /opt/ralph-orchestrator/PROMPT.md

# 仅保留最近 30 天
find $BACKUP_DIR -name "ralph_*.tar.gz" -mtime +30 -delete

# 同步到 S3（可选）
aws s3 sync $BACKUP_DIR s3://my-bucket/ralph-backups/
```

### 灾难恢复
```bash
#!/bin/bash
# restore.sh

BACKUP_FILE=$1
RESTORE_DIR="/opt/ralph-orchestrator"

# 停止服务
systemctl stop ralph

# 恢复备份
tar -xzf $BACKUP_FILE -C /

# 重置 Git 仓库
cd $RESTORE_DIR
git reset --hard HEAD

# 重启服务
systemctl start ralph
```

## 健康检查

### HTTP 健康端点
```python
# health_server.py
from flask import Flask, jsonify
import os
import json

app = Flask(__name__)

@app.route('/health')
def health():
    """健康检查端点"""
    try:
        # 检查 Ralph 进程
        pid_file = '/var/run/ralph.pid'
        if os.path.exists(pid_file):
            with open(pid_file) as f:
                pid = int(f.read())
            os.kill(pid, 0)  # 检查进程是否存在
            status = 'healthy'
        else:
            status = 'unhealthy'

        # 检查最后状态
        state_files = glob.glob('.agent/metrics/state_*.json')
        if state_files:
            latest = max(state_files)
            with open(latest) as f:
                state = json.load(f)
        else:
            state = {}

        return jsonify({
            'status': status,
            'iteration': state.get('iteration_count', 0),
            'runtime': state.get('runtime', 0),
            'errors': len(state.get('errors', []))
        })
    except Exception as e:
        return jsonify({'status': 'error', 'message': str(e)}), 500

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=8080)
```

## 生产环境检查清单

### 部署前
- [ ] 所有测试通过
- [ ] 配置已审核
- [ ] API 密钥已保护
- [ ] 备份策略就位
- [ ] 监控已配置
- [ ] 资源限制已设置
- [ ] 安全加固已应用

### 部署
- [ ] 服务已安装
- [ ] 权限设置正确
- [ ] 日志已配置
- [ ] 健康检查工作正常
- [ ] 指标收集激活
- [ ] 备份作业已调度

### 部署后
- [ ] 服务运行中
- [ ] 日志正在生成
- [ ] 指标可见
- [ ] 测试作业成功
- [ ] 告警已配置
- [ ] 文档已更新
