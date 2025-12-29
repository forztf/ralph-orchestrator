# 配置指南

Ralph Orchestrator 提供了丰富的配置选项来控制执行、管理成本并确保安全运行。本指南涵盖所有配置参数和最佳实践。

## 配置方法

### 1. 命令行参数

配置 Ralph Orchestrator 的主要方式是通过命令行参数:

```bash
python ralph_orchestrator.py --agent claude --max-iterations 50 --max-cost 25.0
```

### 2. 环境变量

某些设置可以通过环境变量配置:

```bash
export RALPH_AGENT=claude
export RALPH_MAX_COST=25.0
python ralph_orchestrator.py
```

### 3. 配置文件 (未来版本)

配置文件支持计划在未来版本中提供。

## 核心配置选项

### Agent 选择

| 参数 | 默认值 | 描述 |
|-----------|---------|-------------|
| `--agent` | `auto` | 要使用的 AI agent:`claude`、`q`、`gemini`、`acp` 或 `auto` |
| `--agent-args` | None | 传递给 agent 的额外参数 |
| `--acp-agent` | `gemini` | ACP agent 命令(用于 `-a acp`) |
| `--acp-permission-mode` | `auto_approve` | 权限处理:`auto_approve`、`deny_all`、`allowlist`、`interactive` |

**示例:**
```bash
# 专门使用 Claude
python ralph_orchestrator.py --agent claude

# 自动检测可用的 agent
python ralph_orchestrator.py --agent auto

# 向 agent 传递额外参数
python ralph_orchestrator.py --agent claude --agent-args "--model claude-3-sonnet"

# 使用符合 ACP 的 agent
python ralph_orchestrator.py --agent acp --acp-agent gemini

# 使用带有特定权限模式的 ACP
python ralph_orchestrator.py --agent acp --acp-agent gemini --acp-permission-mode deny_all
```

### 提示词配置

| 参数 | 默认值 | 描述 |
|-----------|---------|-------------|
| `--prompt` | `PROMPT.md` | 提示词文件路径 |
| `--max-prompt-size` | 10MB | 允许的最大提示词文件大小 |

**示例:**
```bash
# 使用自定义提示词文件
python ralph_orchestrator.py --prompt tasks/my-task.md

# 设置最大提示词大小(以字节为单位)
python ralph_orchestrator.py --max-prompt-size 5242880  # 5MB
```

## 执行限制

### 迭代和运行时

| 参数 | 默认值 | 描述 |
|-----------|---------|-------------|
| `--max-iterations` | 100 | 最大迭代次数 |
| `--max-runtime` | 14400 | 最大运行时间(以秒为单位,4 小时) |

**示例:**
```bash
# 快速任务,少量迭代
python ralph_orchestrator.py --max-iterations 10 --max-runtime 600

# 长时间运行的任务
python ralph_orchestrator.py --max-iterations 500 --max-runtime 86400  # 24 小时
```

### Token 和成本管理

| 参数 | 默认值 | 描述 |
|-----------|---------|-------------|
| `--max-tokens` | 1,000,000 | 使用的最大总 token 数 |
| `--max-cost` | 50.0 | 最大成本(以美元为单位) |
| `--context-window` | 200,000 | 上下文窗口大小(以 token 为单位) |
| `--context-threshold` | 0.8 | 在此上下文百分比时触发摘要 |

**示例:**
```bash
# 预算敏感的配置
python ralph_orchestrator.py \
  --max-tokens 100000 \
  --max-cost 5.0 \
  --context-window 100000

# 高容量配置
python ralph_orchestrator.py \
  --max-tokens 5000000 \
  --max-cost 200.0 \
  --context-window 500000
```

## 检查点和恢复

| 参数 | 默认值 | 描述 |
|-----------|---------|-------------|
| `--checkpoint-interval` | 5 | 检查点之间的迭代次数 |
| `--no-git` | False | 禁用 git 检查点 |
| `--no-archive` | False | 禁用提示词归档 |

**示例:**
```bash
# 针对关键任务的频繁检查点
python ralph_orchestrator.py --checkpoint-interval 1

# 禁用 git 操作(用于非 git 目录)
python ralph_orchestrator.py --no-git

# 最小化持久化
python ralph_orchestrator.py --no-git --no-archive
```

## 监控和调试

| 参数 | 默认值 | 描述 |
|-----------|---------|-------------|
| `--metrics-interval` | 10 | 指标日志之间的迭代次数 |
| `--no-metrics` | False | 禁用指标收集 |
| `--verbose` | False | 启用详细日志记录 |
| `--dry-run` | False | 测试配置而不执行 |

**示例:**
```bash
# 详细监控
python ralph_orchestrator.py --verbose --metrics-interval 1

# 测试配置
python ralph_orchestrator.py --dry-run --verbose

# 最小化日志记录
python ralph_orchestrator.py --no-metrics
```

## 安全选项

| 参数 | 默认值 | 描述 |
|-----------|---------|-------------|
| `--allow-unsafe-paths` | False | 允许潜在不安全的文件路径 |

**示例:**
```bash
# 标准安全(推荐)
python ralph_orchestrator.py

# 允许不安全的路径(谨慎使用)
python ralph_orchestrator.py --allow-unsafe-paths
```

## 重试和恢复

| 参数 | 默认值 | 描述 |
|-----------|---------|-------------|
| `--retry-delay` | 2 | 重试之间的延迟(以秒为单位) |

**示例:**
```bash
# 针对速率限制 API 的较慢重试
python ralph_orchestrator.py --retry-delay 10

# 本地 agent 的快速重试
python ralph_orchestrator.py --retry-delay 1
```

## ACP (Agent Client Protocol) 配置

### ACP 选项

| 参数 | 默认值 | 描述 |
|-----------|---------|-------------|
| `--acp-agent` | `gemini` | 运行符合 ACP 的 agent 的命令 |
| `--acp-permission-mode` | `auto_approve` | 权限处理模式 |

### 权限模式

| 模式 | 描述 | 使用场景 |
|------|-------------|----------|
| `auto_approve` | 自动批准所有工具请求 | 受信任的环境、CI/CD |
| `deny_all` | 拒绝所有工具请求 | 测试、沙盒执行 |
| `allowlist` | 仅批准匹配的模式 | 带有特定工具的生产环境 |
| `interactive` | 为每个请求提示用户 | 开发、手动监督 |

### 配置文件 (ralph.yml)

```yaml
adapters:
  acp:
    enabled: true
    timeout: 300
    tool_permissions:
      agent_command: gemini        # ACP agent CLI 命令
      agent_args: []               # 额外的 CLI 参数
      permission_mode: auto_approve
      permission_allowlist:        # 用于 allowlist 模式
        - "fs/read_text_file:*.py"
        - "fs/write_text_file:src/*"
        - "terminal/create:pytest*"
```

### 环境变量

| 变量 | 描述 |
|----------|-------------|
| `RALPH_ACP_AGENT` | 覆盖 `agent_command` |
| `RALPH_ACP_PERMISSION_MODE` | 覆盖 `permission_mode` |
| `RALPH_ACP_TIMEOUT` | 覆盖 `timeout`(整数) |

**示例:**
```bash
# 使用环境变量
export RALPH_ACP_AGENT=gemini
export RALPH_ACP_PERMISSION_MODE=deny_all
python ralph_orchestrator.py --agent acp
```

### ACP 配置文件

对于符合 ACP 的 agent:

```bash
python ralph_orchestrator.py \
  --agent acp \
  --acp-agent gemini \
  --acp-permission-mode auto_approve \
  --max-iterations 100 \
  --max-runtime 14400
```

## 配置文件

### 开发配置文件

用于本地开发和测试:

```bash
python ralph_orchestrator.py \
  --agent q \
  --max-iterations 10 \
  --max-cost 1.0 \
  --verbose \
  --checkpoint-interval 1 \
  --metrics-interval 1
```

### 生产配置文件

用于生产工作负载:

```bash
python ralph_orchestrator.py \
  --agent claude \
  --max-iterations 100 \
  --max-runtime 14400 \
  --max-tokens 1000000 \
  --max-cost 50.0 \
  --checkpoint-interval 5 \
  --metrics-interval 10
```

### 预算配置文件

用于成本敏感的操作:

```bash
python ralph_orchestrator.py \
  --agent q \
  --max-tokens 50000 \
  --max-cost 2.0 \
  --context-window 50000 \
  --context-threshold 0.7
```

### 高性能配置文件

用于复杂的、资源密集型任务:

```bash
python ralph_orchestrator.py \
  --agent claude \
  --max-iterations 500 \
  --max-runtime 86400 \
  --max-tokens 5000000 \
  --max-cost 500.0 \
  --context-window 500000 \
  --checkpoint-interval 10
```

## 配置最佳实践

### 1. 从保守配置开始

从较低的限制开始,根据需要增加:

```bash
# 从小规模开始
python ralph_orchestrator.py --max-iterations 5 --max-cost 1.0

# 根据需要增加
python ralph_orchestrator.py --max-iterations 50 --max-cost 10.0
```

### 2. 使用试运行

在生产环境之前始终测试配置:

```bash
python ralph_orchestrator.py --dry-run --verbose
```

### 3. 监控指标

为生产工作负载启用指标:

```bash
python ralph_orchestrator.py --metrics-interval 5 --verbose
```

### 4. 设置适当的限制

根据任务复杂性选择限制:

- **简单任务**: 10-20 次迭代,$1-5 成本
- **中等任务**: 50-100 次迭代,$10-25 成本
- **复杂任务**: 100-500 次迭代,$50-200 成本

### 5. 频繁检查点

对于长时间运行的任务,经常设置检查点:

```bash
python ralph_orchestrator.py --checkpoint-interval 3
```

## 环境特定配置

### CI/CD 管道

```bash
python ralph_orchestrator.py \
  --agent auto \
  --max-iterations 50 \
  --max-runtime 3600 \
  --no-git \
  --metrics-interval 10
```

### Docker 容器

```dockerfile
ENV RALPH_AGENT=claude
ENV RALPH_MAX_COST=25.0
CMD ["python", "ralph_orchestrator.py", "--no-git", "--max-runtime", "7200"]
```

### Kubernetes

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: ralph-config
data:
  RALPH_AGENT: "claude"
  RALPH_MAX_COST: "50.0"
  RALPH_MAX_ITERATIONS: "100"
```

## 配置故障排除

### 常见问题

1. **找不到 agent**
   - 解决方案:使用 `--agent auto` 检查 agent 安装

2. **超出成本限制**
   - 解决方案:增加 `--max-cost` 或使用更便宜的 agent

3. **上下文溢出**
   - 解决方案:降低 `--context-threshold` 或增加 `--context-window`

4. **性能缓慢**
   - 解决方案:增加 `--checkpoint-interval` 和 `--metrics-interval`

### 调试命令

```bash
# 检查配置
python ralph_orchestrator.py --dry-run --verbose

# 列出可用的 agent
python ralph_orchestrator.py --agent auto --dry-run

# 使用最小配置进行测试
python ralph_orchestrator.py --max-iterations 1 --verbose
```

## 配置参考

有关所有配置选项的完整列表,请运行:

```bash
python ralph_orchestrator.py --help
```

## 后续步骤

- 了解 [AI Agents](agents.md) 及其功能
- 理解 [提示词工程](prompts.md) 以获得更好的结果
- 探索 [成本管理](cost-management.md) 策略
- 设置 [检查点](checkpointing.md) 以进行恢复
