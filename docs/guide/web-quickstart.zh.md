# Web 监控快速入门

在 5 分钟内启动并运行 Ralph Orchestrator Web 监控仪表板。

## 1. 启动 Web 服务器

### 选项 A:独立 Python 脚本

创建 `start_web.py`:

```python
#!/usr/bin/env python3
import asyncio
from ralph_orchestrator.web import WebMonitor

async def main():
    # 创建并启动 web 服务器
    monitor = WebMonitor(
        host="0.0.0.0",
        port=8000,
        enable_auth=False  # 快速测试时禁用认证
    )

    print("🚀 Web 服务器正在启动 http://localhost:8000")
    print("📊 仪表板将自动打开...")

    await monitor.run()

if __name__ == "__main__":
    asyncio.run(main())
```

运行它:
```bash
python start_web.py
```

### 选项 B:与 Orchestrator 集成

```python
#!/usr/bin/env python3
import asyncio
from ralph_orchestrator import RalphOrchestrator
from ralph_orchestrator.web import WebMonitor

async def main():
    # 启动 web 监控器
    monitor = WebMonitor(host="0.0.0.0", port=8000, enable_auth=False)
    monitor_task = asyncio.create_task(monitor.run())

    # 运行带有 web 监控的 orchestrator
    orchestrator = RalphOrchestrator(
        agent_name="claude",
        prompt_file="PROMPT.md",
        web_monitor=monitor
    )

    print(f"🌐 Web 仪表板: http://localhost:8000")
    print(f"🤖 正在使用 {orchestrator.agent_name} 启动 orchestrator")

    # 运行 orchestrator
    orchestrator.run()

if __name__ == "__main__":
    asyncio.run(main())
```

## 2. 访问仪表板

在浏览器中打开:**http://localhost:8000**

您将看到:
- 📊 **系统指标**: CPU、内存和进程数
- 🤖 **活跃的 Orchestrators**: 正在运行的任务和状态
- 📝 **实时日志**: 实时 agent 输出
- 📜 **历史记录**: 以前的执行运行

## 3. 启用认证(生产环境)

对于生产部署,启用认证:

```bash
# 设置环境变量
export RALPH_WEB_SECRET_KEY="your-secret-key-minimum-32-chars"
export RALPH_WEB_USERNAME="admin"
export RALPH_WEB_PASSWORD="secure-password-here"

# 更新您的脚本
monitor = WebMonitor(
    host="0.0.0.0",
    port=8000,
    enable_auth=True  # 启用认证
)
```

## 4. 监控您的 Orchestrators

### 查看任务进度
点击任何 orchestrator 卡片上的 **Tasks** 按钮以查看:
- 当前正在执行的任务
- 队列中待处理的任务
- 已完成的任务及其时间

### 控制执行
- **暂停**: 临时停止 orchestrator
- **恢复**: 继续执行
- **编辑提示**: 即时修改任务

### 检查系统健康状况
指标面板每 5 秒更新一次,显示:
- CPU 使用百分比
- 内存使用量(已用/总计)
- 活跃进程数

## 5. 常用命令

### 检查 web 服务器是否正在运行
```bash
curl http://localhost:8000/api/status
```

### 查看活跃的 orchestrators
```bash
curl http://localhost:8000/api/orchestrators
```

### 获取系统指标
```bash
curl http://localhost:8000/api/metrics
```

## 6. Docker 快速入门

```dockerfile
# Dockerfile
FROM python:3.11-slim
WORKDIR /app
COPY . .
RUN pip install -e .
EXPOSE 8000
CMD ["python", "-c", "import asyncio; from ralph_orchestrator.web import WebMonitor; asyncio.run(WebMonitor(host='0.0.0.0', port=8000).run())"]
```

构建和运行:
```bash
docker build -t ralph-web .
docker run -p 8000:8000 ralph-web
```

## 7. 故障排除

### 端口已被占用
```bash
# 查找使用端口 8000 的进程
lsof -i :8000
# 或使用不同的端口
monitor = WebMonitor(port=8080)
```

### 无法连接到仪表板
```bash
# 检查服务器是否正在运行
ps aux | grep ralph
# 检查防火墙设置
sudo ufw allow 8000
```

### WebSocket 断开连接
- 检查浏览器控制台是否有错误
- 确保没有代理阻止 WebSocket
- 尝试禁用认证以进行测试

## 后续步骤

- 📖 阅读[完整的 Web 监控指南](./web-monitoring.md)
- 🔒 配置[认证和安全](./web-monitoring.md#security-considerations)
- 🚀 部署到[生产环境](./web-monitoring.md#production-deployment)
- 📊 探索 [API 端点](./web-monitoring.md#api-endpoints)
