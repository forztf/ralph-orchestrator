# Docker 部署指南

使用 Docker 部署 Ralph Orchestrator，以获得一致、可重现的环境。

## 前置条件

- 已安装 Docker Engine 20.10+
- Docker Compose 2.0+（可选，用于多容器设置）
- 至少配置一个 AI CLI 工具的 API 密钥
- 最少 2GB RAM，推荐 4GB
- 10GB 磁盘空间用于镜像和数据

## 快速开始

### 使用预构建镜像

```bash
# 拉取最新镜像
docker pull ghcr.io/mikeyobrien/ralph-orchestrator:latest

# 使用默认设置运行
docker run -it \
  -v $(pwd):/workspace \
  -e CLAUDE_API_KEY=$CLAUDE_API_KEY \
  ghcr.io/mikeyobrien/ralph-orchestrator:latest
```

### 从源代码构建

在项目根目录创建 `Dockerfile`：

```dockerfile
# 多阶段构建以优化镜像大小
FROM python:3.11-slim as builder

# 安装构建依赖
RUN apt-get update && apt-get install -y \
    gcc \
    git \
    && rm -rf /var/lib/apt/lists/*

# 复制依赖文件
WORKDIR /build
COPY pyproject.toml uv.lock ./
RUN pip install uv && uv sync --frozen

# 运行时阶段
FROM python:3.11-slim

# 安装运行时依赖
RUN apt-get update && apt-get install -y \
    git \
    nodejs \
    npm \
    && rm -rf /var/lib/apt/lists/*

# 安装 AI CLI 工具
RUN npm install -g @anthropic-ai/claude-code
RUN npm install -g @google/gemini-cli

# 复制应用程序
WORKDIR /app
COPY --from=builder /build/.venv /app/.venv
COPY . /app/

# 设置环境
ENV PATH="/app/.venv/bin:$PATH"
ENV PYTHONUNBUFFERED=1

# 创建工作区目录
RUN mkdir -p /workspace
WORKDIR /workspace

# 入口点
ENTRYPOINT ["python", "/app/ralph_orchestrator.py"]
CMD ["--help"]
```

构建并运行：

```bash
# 构建镜像
docker build -t ralph-orchestrator:local .

# 使用您的提示词运行
docker run -it \
  -v $(pwd):/workspace \
  -e CLAUDE_API_KEY=$CLAUDE_API_KEY \
  ralph-orchestrator:local \
  --agent claude \
  --prompt PROMPT.md
```

## Docker Compose 设置

用于包含多个服务的复杂部署：

```yaml
# docker-compose.yml
version: '3.8'

services:
  ralph:
    image: ghcr.io/mikeyobrien/ralph-orchestrator:latest
    container_name: ralph-orchestrator
    environment:
      - CLAUDE_API_KEY=${CLAUDE_API_KEY}
      - GEMINI_API_KEY=${GEMINI_API_KEY}
      - Q_API_KEY=${Q_API_KEY}
      - RALPH_MAX_ITERATIONS=100
      - RALPH_MAX_RUNTIME=14400
    volumes:
      - ./workspace:/workspace
      - ./prompts:/prompts:ro
      - ralph-cache:/app/.cache
    networks:
      - ralph-network
    restart: unless-stopped
    command:
      - --agent=auto
      - --prompt=/prompts/PROMPT.md
      - --verbose

  # 可选：使用 Prometheus 进行监控
  prometheus:
    image: prom/prometheus:latest
    container_name: ralph-prometheus
    volumes:
      - ./monitoring/prometheus.yml:/etc/prometheus/prometheus.yml
      - prometheus-data:/prometheus
    networks:
      - ralph-network
    ports:
      - "9090:9090"

  # 可选：使用 Grafana 进行可视化
  grafana:
    image: grafana/grafana:latest
    container_name: ralph-grafana
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=admin
    volumes:
      - grafana-data:/var/lib/grafana
      - ./monitoring/dashboards:/etc/grafana/provisioning/dashboards
    networks:
      - ralph-network
    ports:
      - "3000:3000"

volumes:
  ralph-cache:
  prometheus-data:
  grafana-data:

networks:
  ralph-network:
    driver: bridge
```

启动服务栈：

```bash
# 启动所有服务
docker-compose up -d

# 查看日志
docker-compose logs -f ralph

# 停止所有服务
docker-compose down
```

## 环境变量

通过环境变量配置 Ralph：

| 变量 | 描述 | 默认值 |
|----------|-------------|---------|
| `CLAUDE_API_KEY` | Anthropic Claude API 密钥 | 使用 Claude 时必需 |
| `GEMINI_API_KEY` | Google Gemini API 密钥 | 使用 Gemini 时必需 |
| `Q_API_KEY` | Q Chat API 密钥 | 使用 Q 时必需 |
| `RALPH_AGENT` | 默认代理 (claude/gemini/q/auto) | auto |
| `RALPH_MAX_ITERATIONS` | 最大循环迭代次数 | 100 |
| `RALPH_MAX_RUNTIME` | 最大运行时间（秒） | 14400 |
| `RALPH_MAX_TOKENS` | 最大总令牌数 | 1000000 |
| `RALPH_MAX_COST` | 最大成本（美元） | 50.0 |
| `RALPH_CHECKPOINT_INTERVAL` | Git 检查点频率 | 5 |
| `RALPH_VERBOSE` | 启用详细日志 | false |
| `RALPH_DRY_RUN` | 测试模式，不执行 | false |

## 卷挂载

需要挂载的重要目录：

```bash
docker run -it \
  -v $(pwd)/workspace:/workspace \           # 工作目录
  -v $(pwd)/prompts:/prompts:ro \           # 提示词文件（只读）
  -v $(pwd)/.agent:/app/.agent \            # 代理状态
  -v $(pwd)/.git:/workspace/.git \          # Git 仓库
  -v ~/.ssh:/root/.ssh:ro \                 # SSH 密钥（如需要）
  ralph-orchestrator:latest
```

## 安全考虑

### 以非 Root 用户运行

```dockerfile
# 添加到 Dockerfile
RUN useradd -m -u 1000 ralph
USER ralph
```

```bash
# 使用用户映射运行
docker run -it \
  --user $(id -u):$(id -g) \
  -v $(pwd):/workspace \
  ralph-orchestrator:latest
```

### 密钥管理

切勿硬编码 API 密钥。使用 Docker secrets 或环境文件：

```bash
# .env 文件（添加到 .gitignore！）
CLAUDE_API_KEY=sk-ant-...
GEMINI_API_KEY=AIza...
Q_API_KEY=...

# 使用环境文件运行
docker run -it \
  --env-file .env \
  -v $(pwd):/workspace \
  ralph-orchestrator:latest
```

### 网络隔离

```bash
# 创建隔离网络
docker network create ralph-isolated

# 使用网络隔离运行
docker run -it \
  --network ralph-isolated \
  --network-alias ralph \
  -v $(pwd):/workspace \
  ralph-orchestrator:latest
```

## 资源限制

防止容器失控：

```bash
docker run -it \
  --memory="4g" \
  --memory-swap="4g" \
  --cpu-shares=512 \
  --pids-limit=100 \
  -v $(pwd):/workspace \
  ralph-orchestrator:latest
```

## 健康检查

添加健康监控：

```dockerfile
# 添加到 Dockerfile
HEALTHCHECK --interval=30s --timeout=3s --retries=3 \
  CMD python -c "import sys; sys.exit(0)" || exit 1
```

## 调试

### 交互式 Shell

```bash
# 使用 shell 而不是 ralph 启动
docker run -it \
  -v $(pwd):/workspace \
  --entrypoint /bin/bash \
  ralph-orchestrator:latest

# 在容器内
python /app/ralph_orchestrator.py --dry-run
```

### 查看日志

```bash
# 跟踪容器日志
docker logs -f <container-id>

# 保存日志到文件
docker logs <container-id> > ralph.log 2>&1
```

### 检查运行中的容器

```bash
# 在运行中的容器中执行命令
docker exec -it <container-id> /bin/bash

# 检查进程状态
docker exec <container-id> ps aux

# 查看环境变量
docker exec <container-id> env
```

## 生产部署

### 使用 Docker Swarm

```bash
# 初始化 swarm
docker swarm init

# 创建 secrets
echo $CLAUDE_API_KEY | docker secret create claude_key -
echo $GEMINI_API_KEY | docker secret create gemini_key -

# 部署栈
docker stack deploy -c docker-compose.yml ralph-stack

# 扩展服务
docker service scale ralph-stack_ralph=3
```

### 使用 Kubernetes

参见 [Kubernetes 部署指南](kubernetes.md) 了解大规模容器编排。

## 监控和指标

### 导出指标

```python
# 在配置中启用指标
docker run -it \
  -e RALPH_ENABLE_METRICS=true \
  -e RALPH_METRICS_PORT=8080 \
  -p 8080:8080 \
  ralph-orchestrator:latest
```

### Prometheus 配置

```yaml
# prometheus.yml
scrape_configs:
  - job_name: 'ralph'
    static_configs:
      - targets: ['ralph:8080']
    metrics_path: '/metrics'
```

## 故障排除

### 常见问题

#### 权限被拒绝

```bash
# 修复卷权限
docker run -it \
  --user $(id -u):$(id -g) \
  -v $(pwd):/workspace:Z \  # SELinux 上下文
  ralph-orchestrator:latest
```

#### 内存不足

```bash
# 增加内存限制
docker run -it \
  --memory="8g" \
  --memory-swap="8g" \
  ralph-orchestrator:latest
```

#### 网络超时

```bash
# 增加超时值
docker run -it \
  -e RALPH_RETRY_DELAY=5 \
  -e RALPH_MAX_RETRIES=10 \
  ralph-orchestrator:latest
```

### 调试模式

```bash
# 启用调试日志
docker run -it \
  -e LOG_LEVEL=DEBUG \
  -e RALPH_VERBOSE=true \
  ralph-orchestrator:latest \
  --verbose --dry-run
```

## 最佳实践

1. **在生产环境中始终使用特定的镜像标签**（不要使用 `latest`）
2. **将提示词挂载为只读**以防止意外修改
3. **使用 .dockerignore** 排除不必要的文件
4. **实施健康检查**以实现自动恢复
5. **设置资源限制**以防止资源耗尽
6. **使用多阶段构建**最小化镜像大小
7. **使用 Trivy 等工具扫描镜像漏洞**
8. **切勿将密钥提交到版本控制**
9. **使用卷挂载**保存持久化数据
10. **监控容器日志和指标**

## 示例 .dockerignore

```
# .dockerignore
.git
.github
*.pyc
__pycache__
.pytest_cache
.venv
site/
docs/
tests/
*.md
!README.md
.env
.env.*
```

## 下一步

- [Kubernetes 部署](kubernetes.md) - 容器编排
- [CI/CD 集成](ci-cd.md) - 自动化 Docker 构建
- [生产指南](production.md) - 生产环境最佳实践
