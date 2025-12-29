# 安全考虑

## 概述

Ralph Orchestrator 在具有显著系统访问权限的环境中执行 AI 代理。本文档概述了安全操作的注意事项和最佳实践。

## 威胁模型

### 潜在风险

1. **意外代码执行**
   - AI 代理可能生成并执行有害代码
   - 超出项目范围的文件系统修改
   - 系统命令执行

2. **数据泄露**
   - 提示词或代码中的 API 密钥
   - Git 历史记录中的敏感数据
   - 状态文件中的凭据

3. **资源耗尽**
   - 生成代码中的无限循环
   - 过度的 API 调用
   - 磁盘空间消耗

4. **供应链**
   - 被入侵的 AI CLI 工具
   - 恶意提示词注入
   - 依赖项漏洞

## 安全控制

Ralph 实现了多层安全防护来抵御威胁：

```
 🔒 安全防御层级

   ╭───────────────────╮
   │    用户输入       │
   ╰───────────────────╯
     │
     │
     ∨
   ┌───────────────────┐
   │   输入验证        │
   └───────────────────┘
     │
     │
     ∨
   ┌───────────────────┐
   │   进程隔离        │
   └───────────────────┘
     │
     │
     ∨
   ┌───────────────────┐
   │   文件边界        │
   └───────────────────┘
     │
     │
     ∨
   ┌───────────────────┐
   │   Git 安全        │
   └───────────────────┘
     │
     │
     ∨
   ┌───────────────────┐
   │ 环境变量清理      │
   └───────────────────┘
     │
     │
     ∨
   ╭───────────────────╮
   │    AI 代理        │
   ╰───────────────────╯
```

<details>
<summary>graph-easy 源代码</summary>

```
graph { label: "🔒 Security Defense Layers"; flow: south; }
[ User Input ] { shape: rounded; } -> [ Input Validation ]
[ Input Validation ] -> [ Process Isolation ]
[ Process Isolation ] -> [ File Boundaries ]
[ File Boundaries ] -> [ Git Safety ]
[ Git Safety ] -> [ Env Sanitization ]
[ Env Sanitization ] -> [ AI Agent ] { shape: rounded; }
```

</details>

### 进程隔离

Ralph 在子进程中运行 AI 代理，具有以下特性：

- 超时保护（默认 5 分钟）
- 输出大小限制
- 错误边界

```python
result = subprocess.run(
    cmd,
    capture_output=True,
    text=True,
    timeout=300,  # 5分钟超时
    env=filtered_env  # 清理后的环境
)
```

### 文件系统边界

#### 受限路径

- 在项目目录内工作
- 无权访问系统文件
- 保护 .git 完整性

#### 安全默认值

```python
# 验证路径保持在项目内
def validate_path(path):
    abs_path = os.path.abspath(path)
    project_path = os.path.abspath('.')
    return abs_path.startswith(project_path)
```

### Git 安全

#### 受保护的操作

- 不允许强制推送
- 不允许分支删除
- 不允许历史重写

#### 仅检查点提交

```bash
# Ralph 仅创建检查点提交
git add .
git commit -m "Ralph checkpoint: 迭代 N"
```

### 环境变量清理

#### 过滤变量

```python
SAFE_ENV_VARS = [
    'PATH', 'HOME', 'USER',
    'LANG', 'LC_ALL', 'TERM'
]

def get_safe_env():
    return {k: v for k, v in os.environ.items()
            if k in SAFE_ENV_VARS}
```

#### 无凭据泄露

- 绝不通过环境传递 API 密钥
- 代理应使用自己的凭据存储
- 提示词或日志中不包含机密信息

## 最佳实践

### 1. 提示词安全

#### 应该做

- 执行前审查提示词
- 使用具体的、有界的指令
- 包含安全约束

#### 不应该做

- 在提示词中包含凭据
- 请求系统级更改
- 使用无界迭代

### 2. 代理配置

#### Claude

```bash
# 如果可用，使用受限模式
claude --safe-mode PROMPT.md
```

#### Gemini

```bash
# 限制上下文和功能
gemini --no-web --no-exec PROMPT.md
```

### 3. 仓库设置

#### .gitignore

```gitignore
# 安全敏感文件
*.key
*.pem
.env
.env.*
secrets/
credentials/

# Ralph 工作区
.agent/metrics/
.agent/logs/
```

#### Pre-commit 钩子

```yaml
# .pre-commit-config.yaml
repos:
  - repo: https://github.com/Yelp/detect-secrets
    hooks:
      - id: detect-secrets
        args: ["--baseline", ".secrets.baseline"]
```

### 4. 运行时监控

#### 资源限制

```python
# 设置资源限制
import resource

# 限制内存使用为 1GB
resource.setrlimit(
    resource.RLIMIT_AS,
    (1024 * 1024 * 1024, -1)
)

# 限制 CPU 时间为 1 小时
resource.setrlimit(
    resource.RLIMIT_CPU,
    (3600, -1)
)
```

#### 审计日志

```python
import logging
import json

# 记录所有代理执行
logging.info(json.dumps({
    'event': 'agent_execution',
    'agent': agent_name,
    'timestamp': time.time(),
    'user': os.getenv('USER'),
    'prompt_hash': hashlib.sha256(prompt.encode()).hexdigest()
}))
```

## 安全检查清单

### 运行 Ralph 之前

- [ ] 审查 PROMPT.md 中的不安全指令
- [ ] 检查提示词中是否包含凭据
- [ ] 验证工作目录是否正确
- [ ] 确保 Git 仓库已备份
- [ ] 确认代理工具是最新的

### 执行期间

- [ ] 监控资源使用情况
- [ ] 观察意外的文件更改
- [ ] 检查代理输出是否存在异常
- [ ] 验证检查点是否创建
- [ ] 确保日志中不包含敏感数据

### 完成之后

- [ ] 审查生成的代码是否存在安全问题
- [ ] 检查 Git 历史是否泄露了机密
- [ ] 验证系统文件未被修改
- [ ] 清理临时文件
- [ ] 轮换任何可能暴露的凭据

## 事件响应

### 如果怀疑被入侵

1. **立即行动**

   ```bash
   # 停止 Ralph
   pkill -f ralph_orchestrator

   # 保留证据
   cp -r .agent /tmp/ralph-incident-$(date +%s)

   # 检查修改
   git status
   git diff
   ```

2. **调查**
   - 审查 .agent/metrics/state\_\*.json
   - 检查系统日志
   - 检查 Git 历史记录
   - 分析代理输出

3. **恢复**

   ```bash
   # 重置到最后已知的好状态
   git reset --hard <last-good-commit>

   # 清理工作区
   rm -rf .agent

   # 如需要则轮换凭据
   # 更新受影响服务的 API 密钥
   ```

## 沙箱选项

### Docker 容器

```dockerfile
FROM python:3.11-slim
RUN useradd -m -s /bin/bash ralph
USER ralph
WORKDIR /home/ralph/project
COPY --chown=ralph:ralph . .
CMD ["./ralph", "run"]
```

### 虚拟机

```bash
# 在带有快照的 VM 中运行
vagrant up
vagrant ssh -c "cd /project && ./ralph run"
vagrant snapshot restore clean
```

### 受限用户

```bash
# 创建受限用户
sudo useradd -m -s /bin/bash ralph-runner
sudo usermod -L ralph-runner  # 禁用密码登录

# 以受限用户运行
sudo -u ralph-runner ./ralph run
```

## API 密钥管理

### 安全存储

#### 绝不在以下位置存储密钥

- PROMPT.md 文件
- Git 仓库
- 脚本中的环境变量
- 日志文件

#### 推荐方法

1. 代理特定的凭据存储
2. 系统密钥链/密钥环
3. 加密保险库（如 HashiCorp Vault）
4. 云密钥管理器

### 密钥轮换

```bash
# 定期轮换计划
# 1. 生成新密钥
# 2. 更新代理配置
# 3. 使用新密钥测试
# 4. 撤销旧密钥
```

## 合规考虑

### 数据隐私

- 不在提示词中处理个人身份信息（PII）
- 共享前清理输出
- 遵守数据驻留要求

### 审计跟踪

- 维护执行日志
- 跟踪提示词修改
- 记录代理交互

### 访问控制

- 限制谁可以运行 Ralph
- 限制代理权限
- 控制仓库访问

## 安全更新

保持最新状态：

- AI CLI 工具更新
- Python 安全补丁
- Git 安全公告
- 依赖项漏洞

```bash
# 检查更新
npm update -g @anthropic-ai/claude-code
pip install --upgrade subprocess
git --version
```

## 报告安全问题

如果您发现安全漏洞：

1. **不要**公开问题
2. 将安全报告发送至：<security@ralph-orchestrator.org>
3. 包含：
   - 漏洞描述
   - 复现步骤
   - 潜在影响
   - 建议修复（如有）

我们致力于在 48 小时内响应并迅速提供修复。
