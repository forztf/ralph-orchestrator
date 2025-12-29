# 检查点和恢复指南

Ralph Orchestrator 提供强大的检查点机制,确保工作永远不会丢失,任务可以在中断后恢复。

## 概述

检查点定期保存编排的状态,实现:
- **恢复**从崩溃或中断
- **进度跟踪**跨迭代
- **状态检查**用于调试
- **审计跟踪**用于合规

## 检查点类型

### 1. Git 检查点

按指定间隔自动 git 提交:

```bash
# 启用 git 检查点(默认)
python ralph_orchestrator.py --checkpoint-interval 5

# 禁用 git 检查点
python ralph_orchestrator.py --no-git
```

**保存内容:**
- 当前提示文件状态
- 代理创建/修改的任何文件
- 时间戳和迭代编号

### 2. 提示归档

提示文件的历史版本:

```bash
# 启用提示归档(默认)
python ralph_orchestrator.py

# 禁用提示归档
python ralph_orchestrator.py --no-archive
```

**位置:** `.agent/prompts/prompt_YYYYMMDD_HHMMSS.md`

### 3. 状态快照

包含编排器状态的 JSON 文件:

```json
{
  "iteration": 15,
  "agent": "claude",
  "start_time": "2024-01-10T10:00:00",
  "tokens_used": 50000,
  "cost_incurred": 2.50,
  "status": "running"
}
```

**位置:** `.agent/metrics/state_*.json`

## 配置

### 检查点间隔

控制检查点的频率:

```bash
# 每次迭代检查点(最大安全性)
python ralph_orchestrator.py --checkpoint-interval 1

# 每 10 次迭代检查点(平衡)
python ralph_orchestrator.py --checkpoint-interval 10

# 每 50 次迭代检查点(最小开销)
python ralph_orchestrator.py --checkpoint-interval 50
```

### 检查点策略

#### 激进检查点
对于关键或实验性任务:

```bash
python ralph_orchestrator.py \
  --checkpoint-interval 1 \
  --metrics-interval 1 \
  --verbose
```

#### 平衡检查点
对于标准生产任务:

```bash
python ralph_orchestrator.py \
  --checkpoint-interval 5 \
  --metrics-interval 10
```

#### 最小检查点
对于简单、快速的任务:

```bash
python ralph_orchestrator.py \
  --checkpoint-interval 20 \
  --no-archive
```

## 恢复程序

### 自动恢复

Ralph Orchestrator 自动从最后一个检查点恢复:

1. **检测中断**
2. **加载最后检查点**
3. **从最后已知状态恢复**
4. **继续迭代**

### 手动恢复

#### 从 Git 检查点

```bash
# 查看检查点历史
git log --oneline | grep "Ralph checkpoint"

# 恢复特定检查点
git checkout <commit-hash>

# 恢复编排
python ralph_orchestrator.py --prompt PROMPT.md
```

#### 从提示归档

```bash
# 列出归档的提示
ls -la .agent/prompts/

# 恢复归档的提示
cp .agent/prompts/prompt_20240110_100000.md PROMPT.md

# 恢复编排
python ralph_orchestrator.py
```

#### 从状态快照

```python
# 以编程方式加载状态
import json

with open('.agent/metrics/state_20240110_100000.json') as f:
    state = json.load(f)

print(f"Last iteration: {state['iteration']}")
print(f"Tokens used: {state['tokens_used']}")
print(f"Cost incurred: ${state['cost_incurred']}")
```

## 检查点存储

### 目录结构

```
.agent/
├── checkpoints/       # Git 检查点元数据
├── prompts/          # 归档的提示文件
│   ├── prompt_20240110_100000.md
│   ├── prompt_20240110_101500.md
│   └── prompt_20240110_103000.md
├── metrics/          # 状态和指标
│   ├── state_20240110_100000.json
│   ├── state_20240110_101500.json
│   └── metrics_20240110_103000.json
└── logs/            # 执行日志
```

### 存储管理

#### 清理旧检查点

```bash
# 删除 7 天前的检查点
find .agent/prompts -mtime +7 -delete
find .agent/metrics -name "*.json" -mtime +7 -delete

# 仅保留最后 100 个检查点
ls -t .agent/prompts/*.md | tail -n +101 | xargs rm -f
```

#### 备份检查点

```bash
# 创建备份归档
tar -czf ralph_checkpoints_$(date +%Y%m%d).tar.gz .agent/

# 备份到远程
rsync -av .agent/ user@backup-server:/backups/ralph/
```

## 高级检查点

### 自定义检查点触发器

除了基于间隔的检查点,您可以在提示中触发检查点:

```markdown
## 进度
- 步骤 1 完成 [CHECKPOINT]
- 步骤 2 完成 [CHECKPOINT]
- 步骤 3 完成 [CHECKPOINT]
```

### 检查点钩子

使用 git 钩子进行自定义检查点处理:

```bash
# .git/hooks/post-commit
#!/bin/bash
if [[ $1 == *"Ralph checkpoint"* ]]; then
    # 自定义备份或通知
    cp PROMPT.md /backup/location/
    echo "Checkpoint created" | mail -s "Ralph Progress" admin@example.com
fi
```

### 分布式检查点

对于团队环境:

```bash
# 将检查点推送到共享存储库
python ralph_orchestrator.py --checkpoint-interval 5

# 在另一个终端/机器
git pull  # 获取最新检查点

# 或使用自动同步
watch -n 60 'git pull'
```

## 最佳实践

### 1. 选择适当的间隔

| 任务类型 | 建议间隔 | 理由 |
|-----------|---------------------|-----------|
| 实验 | 1-2 | 最大恢复点 |
| 开发 | 5-10 | 平衡安全/性能 |
| 生产 | 10-20 | 最小化开销 |
| 简单 | 20-50 | 低风险任务 |

### 2. 监控检查点大小

```bash
# 检查检查点存储使用情况
du -sh .agent/

# 监控增长
watch -n 60 'du -sh .agent/*'
```

### 3. 测试恢复

定期测试恢复程序:

```bash
# 模拟中断
python ralph_orchestrator.py &
PID=$!
sleep 30
kill $PID

# 验证恢复
python ralph_orchestrator.py  # 应该恢复
```

### 4. 定期清理

实现检查点轮换:

```bash
# 保留最后 50 个检查点
#!/bin/bash
MAX_CHECKPOINTS=50
COUNT=$(ls .agent/prompts/*.md 2>/dev/null | wc -l)
if [ $COUNT -gt $MAX_CHECKPOINTS ]; then
    ls -t .agent/prompts/*.md | tail -n +$(($MAX_CHECKPOINTS+1)) | xargs rm
fi
```

## 故障排除

### 常见问题

#### 1. Git 检查点失败

**错误:** "Not a git repository"

**解决方案:**
```bash
# 初始化 git 存储库
git init
git add .
git commit -m "Initial commit"

# 或禁用 git 检查点
python ralph_orchestrator.py --no-git
```

#### 2. 检查点存储已满

**错误:** "No space left on device"

**解决方案:**
```bash
# 清理旧检查点
find .agent -type f -mtime +30 -delete

# 移动到更大的存储
mv .agent /larger/disk/
ln -s /larger/disk/.agent .agent
```

#### 3. 损坏的检查点

**错误:** "Invalid checkpoint data"

**解决方案:**
```bash
# 使用以前的检查点
ls -la .agent/prompts/  # 查找早期版本
cp .agent/prompts/prompt_EARLIER.md PROMPT.md
```

### 恢复验证

验证检查点完整性:

```python
#!/usr/bin/env python3
import json
import os
from pathlib import Path

def validate_checkpoints():
    checkpoint_dir = Path('.agent/metrics')
    for state_file in checkpoint_dir.glob('state_*.json'):
        try:
            with open(state_file) as f:
                data = json.load(f)
                assert 'iteration' in data
                assert 'agent' in data
                print(f"✓ {state_file.name}")
        except Exception as e:
            print(f"✗ {state_file.name}: {e}")

validate_checkpoints()
```

## 性能影响

### 检查点开销

| 间隔 | 开销 | 使用案例 |
|----------|----------|----------|
| 1 | 高 (5-10%) | 关键任务 |
| 5 | 中等 (2-5%) | 标准任务 |
| 10 | 低 (1-2%) | 长任务 |
| 20+ | 最小 (<1%) | 简单任务 |

### 优化提示

1. **使用 SSD** 用于检查点存储
2. **禁用不必要的功能** (例如,如果不需要则使用 `--no-archive`)
3. **根据任务关键性调整间隔**
4. **定期清理** 以维持性能

## 集成

### CI/CD 集成

```yaml
# .github/workflows/ralph.yml
name: Ralph Orchestration
on:
  push:
    branches: [main]

jobs:
  orchestrate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2

      - name: Run Ralph
        run: |
          python ralph_orchestrator.py \
            --checkpoint-interval 10 \
            --max-iterations 100

      - name: Save Checkpoints
        uses: actions/upload-artifact@v2
        with:
          name: ralph-checkpoints
          path: .agent/
```

### 监控集成

```bash
# 发送检查点事件到监控
#!/bin/bash
CHECKPOINT_COUNT=$(ls .agent/prompts/*.md 2>/dev/null | wc -l)
curl -X POST https://metrics.example.com/api/v1/metrics \
  -d "ralph.checkpoints.count=$CHECKPOINT_COUNT"
```

## 下一步

- 了解[成本管理](cost-management.md)以优化检查点成本
- 探索[配置](configuration.md)以了解检查点选项
- 查看[故障排除](../troubleshooting.md)以了解恢复问题
- 查看[示例](../examples/index.md)以了解检查点模式
