# 监控与可观测性

## 概述

Ralph Orchestrator 提供全面的监控功能，用于跟踪执行、性能和系统健康状况。本指南涵盖监控工具、指标和最佳实践。

## 内置监控

Ralph 的监控系统通过多个通道收集和路由执行数据：

```
                           📊 指标收集流程

                                                                        ┌────────────────────┐
                                                                   ┌──> │ .agent/metrics/    │
                                                                   │    └────────────────────┘
┌─────────────────┐     ┌──────────────────┐     ┌───────────────┐ │    ┌────────────────────┐
│   Orchestrator  │ ──> │ Iteration Events │ ──> │    Metrics    │ ┼──> │   .agent/logs/     │
└─────────────────┘     └──────────────────┘     │   Collector   │ │    └────────────────────┘
                                                 └───────────────┘ │    ┌────────────────────┐
                                                                   └──> │     Console        │
                                                                        └────────────────────┘
```

<details>
<summary>graph-easy 源码</summary>

```
graph { label: "📊 Metrics Collection Flow"; flow: east; }
[ Orchestrator ] -> [ Iteration Events ] -> [ Metrics Collector ]
[ Metrics Collector ] -> [ .agent/metrics/ ]
[ Metrics Collector ] -> [ .agent/logs/ ]
[ Metrics Collector ] -> [ Console ]
```

</details>

### 状态文件

Ralph 自动在 `.agent/metrics/` 中生成状态文件：

```json
{
  "iteration_count": 15,
  "runtime": 234.5,
  "start_time": "2025-09-07T15:44:35",
  "agent": "claude",
  "prompt_file": "PROMPT.md",
  "status": "running",
  "errors": [],
  "checkpoints": [5, 10, 15],
  "last_output_size": 2048
}
```

### 实时状态

```bash
# 检查当前状态
./ralph status

# 输出：
Ralph Orchestrator Status
=========================
Status: RUNNING
Current Iteration: 15
Runtime: 3m 54s
Agent: claude
Last Checkpoint: iteration 15
Errors: 0
```

### 执行日志

#### 详细模式

```bash
# 启用详细日志记录
./ralph run --verbose

# 输出包括：
# - Agent 命令
# - 执行时间
# - 输出摘要
# - 错误详情
```

#### 日志级别

```python
import logging

# 配置日志级别
logging.basicConfig(
    level=logging.DEBUG,  # DEBUG, INFO, WARNING, ERROR
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s',
    handlers=[
        logging.FileHandler('.agent/logs/ralph.log'),
        logging.StreamHandler()
    ]
)
```

## 指标收集

### 性能指标

```python
# 自动收集的指标
metrics = {
    'iteration_times': [],      # 每次迭代的时间
    'agent_response_times': [], # Agent 执行时长
    'output_sizes': [],         # 每次迭代的响应大小
    'error_rate': 0.0,         # 每次迭代的错误数
    'checkpoint_times': [],     # 检查点创建时长
    'total_api_calls': 0       # Agent 调用总数
}
```

### 自定义指标

```python
# 添加自定义指标收集
class MetricsCollector:
    def record_metric(self, name: str, value: float):
        """记录自定义指标"""
        timestamp = time.time()
        self.metrics.append({
            'name': name,
            'value': value,
            'timestamp': timestamp
        })

    def export_metrics(self):
        """将指标导出为 JSON"""
        with open('.agent/metrics/custom.json', 'w') as f:
            json.dump(self.metrics, f, indent=2)
```

## 监控工具

### 1. Ralph Monitor（内置）

```bash
# 持续监控
watch -n 5 './ralph status'

# 跟踪日志
tail -f .agent/logs/ralph.log

# 监控指标
watch -n 10 'cat .agent/metrics/state_*.json | jq .'
```

### 2. Git 历史监控

```bash
# 查看检查点历史
git log --oneline | grep "Ralph checkpoint"

# 分析代码变化
git diff --stat HEAD~10..HEAD

# 跟踪文件修改
git log --follow -p PROMPT.md
```

### 3. 系统资源监控

```bash
# 监控 Ralph 进程
htop -p $(pgrep -f ralph_orchestrator)

# 跟踪资源使用
pidstat -p $(pgrep -f ralph_orchestrator) 1

# 监控文件系统变化
inotifywait -m -r . -e modify,create,delete
```

## 仪表板设置

### 终端仪表板

创建 `monitor.sh`：

```bash
#!/bin/bash
# Ralph 监控仪表板

while true; do
    clear
    echo "=== RALPH ORCHESTRATOR MONITOR ==="
    echo ""

    # 状态
    ./ralph status
    echo ""

    # 最近的错误
    echo "Recent Errors:"
    tail -n 5 .agent/logs/ralph.log | grep ERROR || echo "No errors"
    echo ""

    # 资源使用
    echo "Resource Usage:"
    ps aux | grep ralph_orchestrator | grep -v grep
    echo ""

    # 最新的检查点
    echo "Latest Checkpoint:"
    ls -lt .agent/checkpoints/ | head -2

    sleep 5
done
```

### Web 仪表板（可选）

```python
# 简单的 Flask 仪表板
from flask import Flask, jsonify, render_template_string
import json
import glob

app = Flask(__name__)

@app.route('/metrics')
def metrics():
    # 获取最新状态文件
    state_files = glob.glob('.agent/metrics/state_*.json')
    if state_files:
        latest = max(state_files)
        with open(latest) as f:
            return jsonify(json.load(f))
    return jsonify({'status': 'no data'})

@app.route('/')
def dashboard():
    return render_template_string('''
    <html>
        <head>
            <title>Ralph Dashboard</title>
            <script>
                function updateMetrics() {
                    fetch('/metrics')
                        .then(response => response.json())
                        .then(data => {
                            document.getElementById('metrics').innerHTML =
                                JSON.stringify(data, null, 2);
                        });
                }
                setInterval(updateMetrics, 5000);
            </script>
        </head>
        <body onload="updateMetrics()">
            <h1>Ralph Orchestrator Dashboard</h1>
            <pre id="metrics"></pre>
        </body>
    </html>
    ''')

if __name__ == '__main__':
    app.run(debug=True, port=5000)
```

## 告警

### 错误检测

```python
# 监控错误
def check_errors():
    with open('.agent/metrics/state_latest.json') as f:
        state = json.load(f)

    if state.get('errors'):
        send_alert(f"Ralph encountered errors: {state['errors']}")

    if state.get('iteration_count', 0) > 100:
        send_alert("Ralph exceeded 100 iterations")

    if state.get('runtime', 0) > 14400:  # 4 hours
        send_alert("Ralph runtime exceeded 4 hours")
```

### 通知方法

```bash
# 桌面通知
notify-send "Ralph Alert" "Task completed successfully"

# 邮件告警
echo "Ralph task failed" | mail -s "Ralph Alert" admin@example.com

# Slack webhook
curl -X POST -H 'Content-type: application/json' \
    --data '{"text":"Ralph task completed"}' \
    YOUR_SLACK_WEBHOOK_URL
```

## 性能分析

### 迭代分析

```python
# 分析迭代性能
import pandas as pd
import matplotlib.pyplot as plt

def analyze_iterations():
    # 加载指标
    metrics = []
    for file in glob.glob('.agent/metrics/state_*.json'):
        with open(file) as f:
            metrics.append(json.load(f))

    # 创建 DataFrame
    df = pd.DataFrame(metrics)

    # 绘制迭代时间
    plt.figure(figsize=(10, 6))
    plt.plot(df['iteration_count'], df['runtime'])
    plt.xlabel('Iteration')
    plt.ylabel('Cumulative Runtime (seconds)')
    plt.title('Ralph Execution Performance')
    plt.savefig('.agent/performance.png')

    # 统计信息
    print(f"Average iteration time: {df['runtime'].diff().mean():.2f}s")
    print(f"Total iterations: {df['iteration_count'].max()}")
    print(f"Error rate: {len(df[df['errors'].notna()]) / len(df):.2%}")
```

### 成本跟踪

```python
# 估算 API 成本
def calculate_costs():
    costs = {
        'claude': 0.01,    # $ per call
        'gemini': 0.005,   # $ per call
        'q': 0.0           # Free
    }

    total_cost = 0
    for file in glob.glob('.agent/metrics/state_*.json'):
        with open(file) as f:
            state = json.load(f)
            agent = state.get('agent', 'claude')
            total_cost += costs.get(agent, 0)

    print(f"Estimated cost: ${total_cost:.2f}")
    return total_cost
```

## 日志管理

### 日志轮转

```python
# 配置日志轮转
import logging.handlers

handler = logging.handlers.RotatingFileHandler(
    '.agent/logs/ralph.log',
    maxBytes=10*1024*1024,  # 10MB
    backupCount=5
)
```

### 日志聚合

```bash
# 合并所有日志
cat .agent/logs/*.log > combined.log

# 按日期过滤
grep "2025-09-07" .agent/logs/*.log

# 仅提取错误
grep -E "ERROR|CRITICAL" .agent/logs/*.log > errors.log
```

### 日志分析

```bash
# 按类型统计错误
grep ERROR .agent/logs/*.log | cut -d: -f4 | sort | uniq -c

# 查找运行时间最长的迭代
grep "Iteration .* completed" .agent/logs/*.log | \
    awk '{print $NF}' | sort -rn | head -10

# Agent 使用统计
grep "Using agent:" .agent/logs/*.log | \
    cut -d: -f4 | sort | uniq -c
```

## 健康检查

### 自动健康检查

```python
def health_check():
    """综合健康检查"""
    health = {
        'status': 'healthy',
        'checks': []
    }

    # 检查提示文件是否存在
    if not os.path.exists('PROMPT.md'):
        health['status'] = 'unhealthy'
        health['checks'].append('PROMPT.md missing')

    # 检查 agent 可用性
    for agent in ['claude', 'q', 'gemini']:
        if shutil.which(agent):
            health['checks'].append(f'{agent}: available')
        else:
            health['checks'].append(f'{agent}: not found')

    # 检查磁盘空间
    stat = os.statvfs('.')
    free_space = stat.f_bavail * stat.f_frsize / (1024**3)  # GB
    if free_space < 1:
        health['status'] = 'warning'
        health['checks'].append(f'Low disk space: {free_space:.2f}GB')

    # 检查 Git 状态
    result = subprocess.run(['git', 'status', '--porcelain'],
                          capture_output=True, text=True)
    if result.stdout:
        health['checks'].append('Uncommitted changes present')

    return health
```

## 通过监控进行故障排除

### 常见问题

| 症状              | 检查                         | 解决方案                         |
| -------------------- | ----------------------------- | -------------------------------- |
| 迭代次数高 | `.agent/metrics/state_*.json` | 检查提示词清晰度            |
| 性能缓慢     | 日志中的迭代时间       | 检查 agent 响应时间       |
| 内存问题        | 系统监控                | 增加限制或添加交换空间      |
| 重复错误      | 日志中的错误模式        | 修复根本问题             |
| 无进展          | Git diff 输出               | 检查 agent 是否在做出更改 |

### 调试模式

```bash
# 最大详细度
RALPH_DEBUG=1 ./ralph run --verbose

# 跟踪执行
python -m trace -t ralph_orchestrator.py

# 性能分析
python -m cProfile -o profile.stats ralph_orchestrator.py
```

## 最佳实践

1. **定期监控**
   - 每 10-15 分钟检查一次状态
   - 查看日志中的异常
   - 监控资源使用

2. **指标保留**
   - 每周归档旧指标
   - 每月压缩日志
   - 保留 30 天历史记录

3. **告警疲劳**
   - 设置合理的阈值
   - 对相关告警进行分组
   - 优先处理关键问题

4. **文档记录**
   - 记录自定义指标
   - 跟踪性能基线
   - 记录配置更改
