# 成本管理指南

在规模化运行 AI 编排时,有效的成本管理至关重要。本指南帮助您在保持任务质量的同时优化支出。

## 理解成本

### Token 定价

每百万 token 的当前价格:

| Agent | 输入成本 | 输出成本 | 平均成本/任务 |
|-------|------------|-------------|---------------|
| **Claude** | $3.00 | $15.00 | $5-50 |
| **Q Chat** | $0.50 | $1.50 | $1-10 |
| **Gemini** | $0.50 | $1.50 | $1-10 |

### 成本计算

```python
total_cost = (input_tokens / 1_000_000 * input_price) +
             (output_tokens / 1_000_000 * output_price)
```

**示例:**
- 任务使用 100K 输入 token,50K 输出 token
- 使用 Claude: (0.1 × $3) + (0.05 × $15) = $1.05
- 使用 Q Chat: (0.1 × $0.50) + (0.05 × $1.50) = $0.125

## 成本控制机制

### 1. 硬性限制

设置最大支出上限:

```bash
# 严格的 $10 限制
python ralph_orchestrator.py --max-cost 10.0

# 保守的 token 限制
python ralph_orchestrator.py --max-tokens 100000
```

### 2. 上下文管理

通过智能上下文处理减少 token 使用:

```bash
# 激进的上下文管理
python ralph_orchestrator.py \
  --context-window 50000 \
  --context-threshold 0.6  # 在 60% 满时进行摘要
```

### 3. Agent 选择

选择成本效益高的 agent:

```bash
# 开发:使用更便宜的 agent
python ralph_orchestrator.py --agent q --max-cost 5.0

# 生产:使用高质量 agent 并设置限制
python ralph_orchestrator.py --agent claude --max-cost 50.0
```

## 优化策略

### 1. 分层 Agent 策略

在不同任务阶段使用不同的 agent:

```bash
# 阶段 1: 使用 Q 进行研究(便宜)
echo "Research the problem" > research.md
python ralph_orchestrator.py --agent q --prompt research.md --max-cost 2.0

# 阶段 2: 使用 Claude 实施(高质量)
echo "Implement the solution" > implement.md
python ralph_orchestrator.py --agent claude --prompt implement.md --max-cost 20.0

# 阶段 3: 使用 Q 进行测试(便宜)
echo "Test the solution" > test.md
python ralph_orchestrator.py --agent q --prompt test.md --max-cost 2.0
```

### 2. 提示词优化

通过高效提示词减少 token 使用:

#### 之前(昂贵)
```markdown
Please create a comprehensive web application with the following features:
- User authentication system with registration, login, password reset
- Dashboard with charts and graphs
- API with full CRUD operations
- Complete test suite
- Detailed documentation
[... 5000 tokens of requirements ...]
```

#### 之后(优化)
```markdown
Build user auth API:
- Register/login endpoints
- JWT tokens
- PostgreSQL storage
- Basic tests
See spec.md for details.
```

### 3. 上下文窗口管理

#### 自动摘要

```bash
# 提前触发摘要以节省 token
python ralph_orchestrator.py \
  --context-window 100000 \
  --context-threshold 0.5  # 在 50% 时摘要
```

#### 手动上下文控制

```markdown
## Context Management
When context reaches 50%, summarize:
- Keep only essential information
- Remove completed task details
- Compress verbose outputs
```

### 4. 迭代优化

更少、更智能的迭代可以节省资金:

```bash
# 多次快速迭代(昂贵)
python ralph_orchestrator.py --max-iterations 100  # ❌

# 更少、专注的迭代(经济)
python ralph_orchestrator.py --max-iterations 20   # ✅
```

## 成本监控

### 实时跟踪

在执行期间监控成本:

```bash
# 详细的成本报告
python ralph_orchestrator.py \
  --verbose \
  --metrics-interval 1
```

**输出:**
```
[INFO] Iteration 5: Tokens: 25,000 | Cost: $1.25 | Remaining: $48.75
```

### 成本报告

访问详细的成本分解:

```python
import json
from pathlib import Path

# 加载指标
metrics_dir = Path('.agent/metrics')
total_cost = 0

for metric_file in metrics_dir.glob('metrics_*.json'):
    with open(metric_file) as f:
        data = json.load(f)
        total_cost += data.get('cost', 0)

print(f"Total cost: ${total_cost:.2f}")
```

### 成本仪表板

创建监控仪表板:

```python
#!/usr/bin/env python3
import json
import matplotlib.pyplot as plt
from pathlib import Path

costs = []
iterations = []

for metric_file in sorted(Path('.agent/metrics').glob('*.json')):
    with open(metric_file) as f:
        data = json.load(f)
        costs.append(data.get('total_cost', 0))
        iterations.append(data.get('iteration', 0))

plt.plot(iterations, costs)
plt.xlabel('Iteration')
plt.ylabel('Cumulative Cost ($)')
plt.title('Ralph Orchestrator Cost Progression')
plt.savefig('cost_report.png')
```

## 预算规划

### 任务成本估算

| 任务类型 | 复杂度 | 推荐预算 | Agent |
|-----------|------------|-------------------|--------|
| 简单脚本 | 低 | $0.50 - $2 | Q Chat |
| Web API | 中 | $5 - $20 | Gemini/Claude |
| 完整应用 | 高 | $20 - $100 | Claude |
| 数据分析 | 中 | $5 - $15 | Gemini |
| 文档 | 低-中 | $2 - $10 | Q/Claude |
| 调试 | 可变 | $5 - $50 | Claude |

### 月度预算规划

```python
# 计算月度预算需求
tasks_per_month = 50
avg_cost_per_task = 10.0
safety_margin = 1.5

monthly_budget = tasks_per_month * avg_cost_per_task * safety_margin
print(f"Recommended monthly budget: ${monthly_budget}")
```

## 成本优化配置

### 最低成本配置

最大节省,可接受的质量:

```bash
python ralph_orchestrator.py \
  --agent q \
  --max-tokens 50000 \
  --max-cost 2.0 \
  --context-window 30000 \
  --context-threshold 0.5 \
  --checkpoint-interval 10
```

### 平衡配置

良好的质量,合理的成本:

```bash
python ralph_orchestrator.py \
  --agent gemini \
  --max-tokens 200000 \
  --max-cost 10.0 \
  --context-window 100000 \
  --context-threshold 0.7 \
  --checkpoint-interval 5
```

### 质量配置

最佳结果,受控的支出:

```bash
python ralph_orchestrator.py \
  --agent claude \
  --max-tokens 500000 \
  --max-cost 50.0 \
  --context-window 200000 \
  --context-threshold 0.8 \
  --checkpoint-interval 3
```

## 高级成本管理

### 动态 Agent 切换

根据剩余预算切换 agent:

```python
# 动态切换伪代码
if remaining_budget > 20:
    agent = "claude"
elif remaining_budget > 5:
    agent = "gemini"
else:
    agent = "q"
```

### 成本感知提示词

在提示词中包含成本考虑:

```markdown
## Budget Constraints
- Maximum budget: $10
- Optimize for efficiency
- Skip non-essential features if approaching limit
- Prioritize core functionality
```

### 批处理

组合多个小任务:

```bash
# 低效:多次编排
python ralph_orchestrator.py --prompt task1.md  # $5
python ralph_orchestrator.py --prompt task2.md  # $5
python ralph_orchestrator.py --prompt task3.md  # $5
# 总计: $15

# 高效:批量编排
cat task1.md task2.md task3.md > batch.md
python ralph_orchestrator.py --prompt batch.md  # $10
# 总计: $10 (节省 33%)
```

## 成本告警

### 设置告警

```bash
#!/bin/bash
# cost_monitor.sh

COST_LIMIT=25.0
CURRENT_COST=$(python -c "
import json
with open('.agent/metrics/state_latest.json') as f:
    print(json.load(f)['total_cost'])
")

if (( $(echo "$CURRENT_COST > $COST_LIMIT" | bc -l) )); then
    echo "ALERT: Cost exceeded $COST_LIMIT" | mail -s "Ralph Cost Alert" admin@example.com
fi
```

### 自动停止

实现熔断器:

```python
# cost_breaker.py
import json
import sys

with open('.agent/metrics/state_latest.json') as f:
    state = json.load(f)

if state['total_cost'] > state['max_cost'] * 0.9:
    print("WARNING: 90% of budget consumed")
    sys.exit(1)
```

## ROI 分析

### 计算 ROI

```python
# ROI 计算
hours_saved = 10  # 节省的手动工作小时数
hourly_rate = 50  # 开发人员时薪
ai_cost = 25  # AI 编排成本

value_created = hours_saved * hourly_rate
roi = (value_created - ai_cost) / ai_cost * 100

print(f"Value created: ${value_created}")
print(f"AI cost: ${ai_cost}")
print(f"ROI: {roi:.1f}%")
```

### 成本效益矩阵

| 任务 | 手工工时 | 手工成本 | AI 成本 | 节省 |
|------|-------------|-------------|---------|---------|
| API 开发 | 40h | $2000 | $50 | $1950 |
| 文档编写 | 20h | $1000 | $20 | $980 |
| 测试套件 | 30h | $1500 | $30 | $1470 |
| Bug 修复 | 10h | $500 | $25 | $475 |

## 最佳实践

### 1. 从小开始

首先使用最小预算进行测试:

```bash
# 测试运行
python ralph_orchestrator.py --max-cost 1.0 --max-iterations 5

# 成功后扩展
python ralph_orchestrator.py --max-cost 10.0 --max-iterations 50
```

### 2. 持续监控

实时跟踪成本:

```bash
# 终端 1:运行编排
python ralph_orchestrator.py --verbose

# 终端 2:监控成本
watch -n 5 'tail -n 20 .agent/metrics/state_latest.json'
```

### 3. 迭代优化

- 分析成本报告
- 识别昂贵操作
- 优化提示词和设置
- 测试优化效果

### 4. 设置现实预算

- 开发:生产预算的 50%
- 测试:生产预算的 25%
- 生产:具有安全余量的完整预算

### 5. 记录成本

保留记录以供分析:

```bash
# 每次运行后保存成本报告
python ralph_orchestrator.py && \
  cp .agent/metrics/state_latest.json "reports/run_$(date +%Y%m%d_%H%M%S).json"
```

## 故障排除

### 常见问题

1. **意外的高成本**
   - 检查指标中的 token 使用情况
   - 审查提示词效率
   - 验证上下文设置

2. **预算快速超支**
   - 降低上下文窗口
   - 提高摘要阈值
   - 使用更便宜的 agent

3. **预算约束下结果不佳**
   - 略微增加预算
   - 优化提示词
   - 考虑分阶段方法

## 下一步

- 查看 [Agent 选择](agents.md) 了解成本效益选择
- 优化 [提示词](prompts.md) 以提高效率
- 配置 [检查点](checkpointing.md) 以保存进度
- 探索 [示例](../examples/index.md) 了解成本优化模式
