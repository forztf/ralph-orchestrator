# Web 监控仪表板

Ralph Orchestrator 包含一个强大的基于 Web 的监控仪表板,可实时显示代理执行、任务进度和系统健康指标。本指南涵盖有关设置、运行和使用 Web 监控接口所需了解的所有内容。

## 功能

### 核心能力

- **实时监控**: 通过 WebSocket 连接进行实时更新,支持自动重连
- **编排器管理**: 查看活动的编排器,暂停/恢复执行,监控状态
- **任务队列可视化**: 查看当前、待处理和已完成的任务,并显示进度指示器
- **系统指标**: CPU、内存和进程监控,实时更新
- **执行历史**: 在 SQLite 数据库中持久存储运行历史
- **身份验证**: 基于 JWT 的身份验证,具有可配置的安全性
- **提示词编辑**: 实时提示词编辑功能,在下次迭代时生效
- **深色/浅色主题**: 在主题之间切换,并持久保存偏好设置
- **响应式设计**: 适用于从 320px 到 4K 显示器的各种屏幕

### 安全功能

- 基于 JWT token 的身份验证,使用 bcrypt 密码哈希
- 可配置的身份验证(可禁用以用于本地使用)
- 仅限管理员端点用于用户管理
- 未授权响应时自动刷新 token
- 用于生产部署的环境变量配置

## 安装与设置

### 前提条件

Web 监控仪表板包含在 Ralph Orchestrator 安装中。确保您已安装:

```bash
# Ralph Orchestrator 已安装
uv sync  # 或 pip install -e .

# 所需的 Python 包(自动安装)
# - fastapi
# - uvicorn
# - websockets
# - python-jose[cryptography]
# - bcrypt
# - aiosqlite
```

### 配置

Web 服务器可以通过环境变量进行配置:

```bash
# 身份验证设置
export RALPH_WEB_SECRET_KEY="your-secret-key-here"  # JWT 签名密钥
export RALPH_WEB_USERNAME="admin"                    # 默认用户名
export RALPH_WEB_PASSWORD="your-secure-password"     # 默认密码

# 服务器设置
export RALPH_WEB_HOST="0.0.0.0"                     # 绑定的主机
export RALPH_WEB_PORT="8000"                        # 监听的端口
export RALPH_WEB_ENABLE_AUTH="true"                 # 启用/禁用身份验证

# 数据库设置
export RALPH_WEB_DB_PATH="~/.ralph/history.db"      # SQLite 数据库路径
```

## 启动 Web 服务器

### 方法 1: 独立服务器

独立于编排器运行 Web 服务器:

```python
from ralph_orchestrator.web import WebMonitor

# 启动 Web 服务器
monitor = WebMonitor(
    host="0.0.0.0",
    port=8000,
    enable_auth=True  # 本地开发设置为 False
)

# 运行服务器(阻塞)
await monitor.run()
```

### 方法 2: 与编排器集成

与编排器并行运行 Web 服务器:

```python
from ralph_orchestrator import RalphOrchestrator
from ralph_orchestrator.web import WebMonitor

# 在后台启动 Web 服务器
monitor = WebMonitor(host="0.0.0.0", port=8000)
monitor_task = asyncio.create_task(monitor.run())

# 创建并注册编排器
orchestrator = RalphOrchestrator(
    agent_name="claude",
    prompt_file="PROMPT.md",
    web_monitor=monitor  # 传递监控器实例
)

# 编排器将自动注册到监控器
orchestrator.run()
```

### 方法 3: 命令行(即将推出)

```bash
# 仅启动 Web 服务器
ralph web --host 0.0.0.0 --port 8000

# 启动带 Web 服务器的编排器
ralph run --web --web-port 8000
```

## 访问仪表板

### 首次登录

1. 在 Web 浏览器中导航到 `http://localhost:8000`
2. 您将被重定向到登录页面
3. 输入凭据:
   - 默认用户名: `admin`
   - 默认密码: `ralph-admin-2024`
   - 或使用您配置的环境变量

### 仪表板概述

主仪表板由几个部分组成:

#### 顶部栏
- **连接状态**: 显示 WebSocket 连接状态(已连接/已断开)
- **用户名显示**: 显示登录用户
- **主题切换**: 在深色和浅色主题之间切换
- **注销按钮**: 结束当前会话

#### 系统指标面板
- **CPU 使用率**: 实时 CPU 利用率百分比
- **内存使用率**: 当前内存使用量和总可用量
- **活动进程**: 运行中的进程数
- **更新**: 每 5 秒刷新一次

#### 编排器卡片
每个活动的编排器显示:
- **ID 和代理类型**: 唯一标识符和使用的 AI 代理
- **状态徽章**: 运行中、已暂停、已完成或已失败
- **当前迭代**: 任务迭代进度
- **运行时间**: 总执行时间
- **任务队列**: 当前任务的内联视图及进度
- **控制按钮**:
  - **暂停/恢复**: 控制编排器执行
  - **任务**: 查看详细的任务队列
  - **编辑提示词**: 实时修改提示词
  - **查看详情**: (未来功能)详细的编排器视图

#### 实时日志面板
- **实时输出**: 显示代理输出和系统消息
- **暂停/恢复**: 控制日志自动滚动
- **清除**: 清除当前日志显示
- **严重性指示器**: 按消息类型进行颜色编码

#### 执行历史表
- **最近运行**: 显示最近 10 次完成的运行
- **运行详情**: 开始时间、持续时间、迭代次数、最终状态
- **数据库持久性**: 服务器重启后仍然保留

## API 端点

Web 服务器提供全面的 REST API:

### 身份验证端点

```http
POST /api/auth/login
Content-Type: application/json
{
  "username": "admin",
  "password": "your-password"
}

Response:
{
  "access_token": "eyJ...",
  "token_type": "bearer"
}
```

```http
GET /api/auth/verify
Authorization: Bearer <token>

Response:
{
  "valid": true,
  "username": "admin"
}
```

### 监控端点

```http
GET /api/status
Authorization: Bearer <token>

Response:
{
  "status": "running",
  "orchestrators": 2,
  "version": "1.0.0"
}
```

```http
GET /api/orchestrators
Authorization: Bearer <token>

Response:
[
  {
    "id": "orch_abc123",
    "agent": "claude",
    "status": "running",
    "iteration": 5,
    "start_time": "2024-01-01T00:00:00Z"
  }
]
```

```http
GET /api/orchestrators/{id}/tasks
Authorization: Bearer <token>

Response:
{
  "current_task": {
    "content": "Implement authentication",
    "status": "in_progress",
    "start_time": "2024-01-01T00:00:00Z"
  },
  "queue": [...],
  "completed": [...]
}
```

### 控制端点

```http
POST /api/orchestrators/{id}/pause
Authorization: Bearer <token>
```

```http
POST /api/orchestrators/{id}/resume
Authorization: Bearer <token>
```

### 提示词管理

```http
GET /api/orchestrators/{id}/prompt
Authorization: Bearer <token>

Response:
{
  "content": "# Task: ...",
  "path": "/path/to/PROMPT.md",
  "last_modified": "2024-01-01T00:00:00Z"
}
```

```http
POST /api/orchestrators/{id}/prompt
Authorization: Bearer <token>
Content-Type: application/json
{
  "content": "# Updated Task: ..."
}
```

### WebSocket 连接

```javascript
const ws = new WebSocket('ws://localhost:8000/ws');

ws.onopen = () => {
  // 发送身份验证
  ws.send(JSON.stringify({
    type: 'auth',
    token: 'your-jwt-token'
  }));
};

ws.onmessage = (event) => {
  const data = JSON.parse(event.data);
  // 处理更新: orchestrator_update, metrics_update, log_message
};
```

## 数据库架构

Web 服务器使用 SQLite 进行持久存储:

### 表

#### orchestrator_runs
```sql
CREATE TABLE orchestrator_runs (
    run_id TEXT PRIMARY KEY,
    orchestrator_id TEXT NOT NULL,
    agent_name TEXT,
    prompt_path TEXT,
    start_time REAL NOT NULL,
    end_time REAL,
    status TEXT NOT NULL,
    total_iterations INTEGER DEFAULT 0,
    final_error TEXT,
    metadata TEXT
);
```

#### iteration_history
```sql
CREATE TABLE iteration_history (
    iteration_id TEXT PRIMARY KEY,
    run_id TEXT NOT NULL,
    iteration_number INTEGER NOT NULL,
    start_time REAL NOT NULL,
    end_time REAL,
    status TEXT NOT NULL,
    agent_output TEXT,
    error_message TEXT,
    metrics TEXT,
    FOREIGN KEY (run_id) REFERENCES orchestrator_runs(run_id)
);
```

#### task_history
```sql
CREATE TABLE task_history (
    task_id TEXT PRIMARY KEY,
    run_id TEXT NOT NULL,
    task_content TEXT NOT NULL,
    status TEXT NOT NULL,
    start_time REAL,
    end_time REAL,
    iteration_completed INTEGER,
    FOREIGN KEY (run_id) REFERENCES orchestrator_runs(run_id)
);
```

### 数据库位置

默认情况下,数据库存储在 `~/.ralph/history.db`。您可以直接查询它:

```bash
sqlite3 ~/.ralph/history.db "SELECT * FROM orchestrator_runs ORDER BY start_time DESC LIMIT 10;"
```

## 生产部署

### 使用 Docker

```dockerfile
FROM python:3.11-slim

WORKDIR /app

# 安装依赖
COPY requirements.txt .
RUN pip install -r requirements.txt

# 复制应用程序
COPY . .

# 设置环境变量
ENV RALPH_WEB_HOST=0.0.0.0
ENV RALPH_WEB_PORT=8000
ENV RALPH_WEB_ENABLE_AUTH=true

# 运行 Web 服务器
CMD ["python", "-m", "ralph_orchestrator.web"]
```

### 使用 systemd

创建 `/etc/systemd/system/ralph-web.service`:

```ini
[Unit]
Description=Ralph Orchestrator Web Monitor
After=network.target

[Service]
Type=simple
User=ralph
WorkingDirectory=/opt/ralph-orchestrator
Environment="RALPH_WEB_SECRET_KEY=your-secret-key"
Environment="RALPH_WEB_USERNAME=admin"
Environment="RALPH_WEB_PASSWORD=secure-password"
ExecStart=/usr/bin/python3 -m ralph_orchestrator.web
Restart=always

[Install]
WantedBy=multi-user.target
```

### Nginx 反向代理

```nginx
server {
    listen 80;
    server_name ralph.example.com;

    location / {
        proxy_pass http://localhost:8000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    location /ws {
        proxy_pass http://localhost:8000/ws;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_read_timeout 86400;
    }
}
```

## 故障排除

### 常见问题

#### WebSocket 连接失败
```javascript
// 检查浏览器控制台是否有错误
// 确保身份验证 token 有效
// 如果在不同端口上运行,请验证 CORS 设置
```

#### 身份验证问题
```bash
# 重置为默认凭据
unset RALPH_WEB_USERNAME
unset RALPH_WEB_PASSWORD
# 默认值: admin / ralph-admin-2024
```

#### 数据库错误
```bash
# 检查数据库权限
ls -la ~/.ralph/history.db

# 如果损坏则重置数据库
rm ~/.ralph/history.db
# 将在下次启动时重新创建
```

#### 性能问题
```python
# 增加 WebSocket ping 间隔
monitor = WebMonitor(
    ws_ping_interval=60,  # 默认为 30
    metrics_interval=10    # 默认为 5
)
```

### 调试模式

启用详细日志记录:

```python
import logging
logging.basicConfig(level=logging.DEBUG)

monitor = WebMonitor(debug=True)
```

## 安全注意事项

### 生产清单

- [ ] 更改默认凭据
- [ ] 使用强 JWT 密钥(最少 32 个字符)
- [ ] 在生产环境中启用 HTTPS
- [ ] 配置防火墙规则
- [ ] 实施速率限制
- [ ] 定期安全更新
- [ ] 监控访问日志
- [ ] 定期备份数据库

### 环境变量

切勿将凭据提交到版本控制。使用 `.env` 文件:

```bash
# .env(添加到 .gitignore)
RALPH_WEB_SECRET_KEY=your-very-long-secret-key-here
RALPH_WEB_USERNAME=admin
RALPH_WEB_PASSWORD=super-secure-password-2024
```

## API 速率限制

Web 服务器包含内置速率限制(未来功能):

```python
from ralph_orchestrator.web import WebMonitor

monitor = WebMonitor(
    rate_limit_enabled=True,
    rate_limit_requests=100,  # 每分钟请求数
    rate_limit_window=60       # 时间窗口(秒)
)
```

## 扩展仪表板

### 自定义指标

向仪表板添加自定义指标:

```python
# 在您的编排器中
await monitor.send_metric("custom_metric", {
    "value": 42,
    "label": "Custom Metric",
    "unit": "items"
})
```

### 自定义 API 端点

使用自定义端点扩展 Web 服务器:

```python
from fastapi import APIRouter

router = APIRouter()

@router.get("/api/custom/endpoint")
async def custom_endpoint():
    return {"message": "Custom response"}

monitor.app.include_router(router)
```

## 支持

如有问题、疑问或功能请求:

- **GitHub Issues**: [报告错误或请求功能](https://github.com/mikeyobrien/ralph-orchestrator/issues)
- **文档**: [完整文档](https://mikeyobrien.github.io/ralph-orchestrator/)
- **社区**: [GitHub Discussions](https://github.com/mikeyobrien/ralph-orchestrator/discussions)

## 版本历史

- **v1.0.0**: 初始 Web 监控仪表板
  - 通过 WebSocket 实时监控
  - JWT 身份验证
  - SQLite 持久性
  - 任务队列可视化
  - 提示词编辑功能
  - 响应式设计
